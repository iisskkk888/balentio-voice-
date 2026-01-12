# Stripe Integration Guidelines

## Webhook Idempotency

### Problem

Stripe may send the same webhook event multiple times. Without idempotency, you could:
- Charge users twice
- Create duplicate subscription records
- Grant extra credits

### Solution: Track Processed Events

Create a dedicated table for webhook events:

```typescript
// schema.ts
export const webhookEvents = pgTable('webhook_events', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  stripeEventId: text('stripe_event_id').notNull().unique(), // Important: UNIQUE constraint
  eventType: text('event_type').notNull(),
  processedAt: timestamp('processed_at').defaultNow().notNull(),
});
```

### Idempotent Webhook Handler

```typescript
app.post('/webhooks/stripe', async (c) => {
  const sig = c.req.header('stripe-signature');
  const body = await c.req.text();

  // 1. Verify signature
  let event;
  try {
    event = stripe.webhooks.constructEvent(body, sig, env.STRIPE_WEBHOOK_SECRET);
  } catch (err) {
    return c.json({ error: 'Invalid signature' }, 400);
  }

  // 2. Check if already processed (idempotency)
  const existing = await db.query.webhookEvents.findFirst({
    where: eq(webhookEvents.stripeEventId, event.id),
  });

  if (existing) {
    console.log(`Event ${event.id} already processed, skipping`);
    return c.json({ received: true }); // Return 200 to acknowledge
  }

  // 3. Process event in transaction
  await db.transaction(async (tx) => {
    // Record event first (prevents duplicate processing)
    await tx.insert(webhookEvents).values({
      stripeEventId: event.id,
      eventType: event.type,
    });

    // Then handle the event
    switch (event.type) {
      case 'checkout.session.completed':
        await handleCheckoutCompleted(tx, event.data.object);
        break;
      case 'customer.subscription.updated':
        await handleSubscriptionUpdated(tx, event.data.object);
        break;
      case 'customer.subscription.deleted':
        await handleSubscriptionDeleted(tx, event.data.object);
        break;
    }
  });

  return c.json({ received: true });
});
```

## Database Constraints for Idempotency

Add unique constraints on Stripe resource IDs:

```typescript
export const subscriptions = pgTable('subscriptions', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  userId: integer('user_id').notNull().references(() => users.id),
  stripeSubscriptionId: text('stripe_subscription_id').notNull().unique(), // UNIQUE
  stripeCustomerId: text('stripe_customer_id').notNull(), // Not unique (one customer, many subscriptions)
  status: text().notNull(), // 'active', 'canceled', 'past_due'
  currentPeriodEnd: timestamp('current_period_end').notNull(),
});

export const users = pgTable('users', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  email: text().notNull().unique(),
  stripeCustomerId: text('stripe_customer_id').unique(), // UNIQUE - one customer per user
});
```

## Webhook Event Handlers

### checkout.session.completed

```typescript
async function handleCheckoutCompleted(tx, session) {
  const { customer, subscription } = session;

  // Update user with Stripe customer ID
  await tx.update(users)
    .set({ stripeCustomerId: customer })
    .where(eq(users.email, session.customer_email));

  // Create subscription record (unique constraint prevents duplicates)
  await tx.insert(subscriptions).values({
    userId: user.id,
    stripeSubscriptionId: subscription,
    stripeCustomerId: customer,
    status: 'active',
    currentPeriodEnd: new Date(session.subscription.current_period_end * 1000),
  });
}
```

### customer.subscription.updated

```typescript
async function handleSubscriptionUpdated(tx, subscription) {
  await tx.update(subscriptions)
    .set({
      status: subscription.status,
      currentPeriodEnd: new Date(subscription.current_period_end * 1000),
    })
    .where(eq(subscriptions.stripeSubscriptionId, subscription.id));
}
```

### customer.subscription.deleted

```typescript
async function handleSubscriptionDeleted(tx, subscription) {
  await tx.update(subscriptions)
    .set({ status: 'canceled' })
    .where(eq(subscriptions.stripeSubscriptionId, subscription.id));
}
```

## Outgoing Requests to Stripe

When making requests to Stripe API, use idempotency keys:

```typescript
import { randomUUID } from 'crypto';

const paymentIntent = await stripe.paymentIntents.create({
  amount: 1000,
  currency: 'usd',
}, {
  idempotencyKey: randomUUID(), // Prevents duplicate charges
});
```

## Testing Webhooks Locally

Use Stripe CLI to forward webhooks:

```bash
# Install Stripe CLI
brew install stripe/stripe-cli/stripe

# Forward webhooks to local server
stripe listen --forward-to localhost:3000/webhooks/stripe

# Use the webhook secret from CLI output in .env:
# STRIPE_WEBHOOK_SECRET=whsec_...
```

## Webhook Response Timing

- **Respond within 5 seconds** - Stripe will retry if timeout
- **Return 200 immediately** - don't wait for slow operations
- **Process async tasks** after responding

```typescript
// Good: Quick response
app.post('/webhooks/stripe', async (c) => {
  // Verify + record event (fast)

  // Respond immediately
  c.status(200);

  // Then process (don't await)
  processEventAsync(event).catch(console.error);

  return c.json({ received: true });
});
```

## Security Checklist

- ✅ **Always verify Stripe-Signature** - prevents fake webhooks
- ✅ **Use webhook_events table** - track processed events
- ✅ **Add UNIQUE constraints** - prevent duplicate records
- ✅ **Use transactions** - atomic operations only
- ✅ **Generate idempotency keys** - for outgoing Stripe API calls
- ✅ **Test with Stripe CLI** - don't wait for production bugs
