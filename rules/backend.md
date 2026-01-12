# Backend Guidelines (Hono + Node.js)

## Project Structure

```
apps/api/
├── src/
│   ├── routes/          # Route handlers (thin layer)
│   ├── services/        # Business logic
│   ├── lib/            # Shared utilities
│   ├── middleware/     # Auth, logging, error handling
│   └── index.ts        # App initialization
```

## Hono Context Pattern

Always use single Context parameter instead of separate req/res:

```typescript
// Good: Context object pattern
app.get('/users/:id', async (c) => {
  const id = c.req.param('id');
  const user = await userService.getUser(id);
  return c.json(user);
});

// Bad: Don't destructure or use multiple params
app.get('/users/:id', async (req, res) => { ... });
```

## Type-Safe Validation

Use Zod for request validation (Hono has built-in validator):

```typescript
import { zValidator } from '@hono/zod-validator';
import { z } from 'zod';

const schema = z.object({
  email: z.string().email(),
  password: z.string().min(8),
});

app.post('/register', zValidator('json', schema), async (c) => {
  const data = c.req.valid('json'); // Typed automatically
  // ...
});
```

## Route Organization

Keep routes thin - delegate to services:

```typescript
// routes/users.ts
import { Hono } from 'hono';
import * as userService from '../services/user';

const users = new Hono();

users.get('/:id', async (c) => {
  const id = c.req.param('id');
  const user = await userService.getById(id);
  return c.json(user);
});

export default users;
```

## Error Handling

Use consistent error response format:

```typescript
// middleware/error-handler.ts
app.onError((err, c) => {
  console.error(err);

  return c.json({
    error: {
      message: err.message,
      code: err.code || 'INTERNAL_ERROR',
    },
  }, err.status || 500);
});
```

## Environment Variables

Use type-safe env vars pattern:

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_ACCESS_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  STRIPE_SECRET_KEY: z.string(),
  STRIPE_WEBHOOK_SECRET: z.string(),
});

export const env = envSchema.parse(process.env);
```

## CORS Configuration

```typescript
import { cors } from 'hono/cors';

app.use('/*', cors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
}));
```

## Best Practices

- **Keep routes under 20 lines** - move logic to services
- **One service per domain** - users.ts, subscriptions.ts, transcription.ts
- **Services return data, not responses** - let routes handle c.json()
- **Use async/await** - never mix callbacks
- **Validate all inputs** - use zValidator on every route
