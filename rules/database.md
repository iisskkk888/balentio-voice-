# Database Guidelines (Drizzle ORM + PostgreSQL)

## Schema Conventions

### Primary Keys

Always use `generatedAlwaysAsIdentity()` (modern PostgreSQL pattern):

```typescript
import { pgTable, text, timestamp, integer } from 'drizzle-orm/pg-core';

export const users = pgTable('users', {
  id: integer().primaryKey().generatedAlwaysAsIdentity(),
  email: text().notNull().unique(),
  createdAt: timestamp('created_at').defaultNow().notNull(),
  updatedAt: timestamp('updated_at').defaultNow().notNull(),
});
```

### Naming Conventions

- **Tables**: plural snake_case (`users`, `usage_logs`, `webhook_events`)
- **Columns**: snake_case (`created_at`, `user_id`, `word_count`)
- **Timestamps**: Always include `created_at` and `updated_at`
- **Foreign keys**: `{table}_id` (e.g., `user_id`, `subscription_id`)

### Column Types

```typescript
// Use text for flexible strings (not varchar)
email: text().notNull(),

// Use integer for IDs and counts
wordCount: integer('word_count').notNull(),

// Use timestamp for dates
createdAt: timestamp('created_at').defaultNow().notNull(),

// Use boolean with default
isActive: boolean('is_active').default(true).notNull(),
```

## Migration Workflow

### Creating Migrations

```bash
# Generate migration from schema changes
bun drizzle-kit generate

# Migration file: drizzle/0001_create_users.sql
```

### Migration Naming

Format: `{number}_{descriptive_name}.sql`
- `0001_create_users.sql`
- `0002_add_subscriptions.sql`
- `0003_add_usage_logs.sql`

### Running Migrations

```typescript
// apps/api/src/lib/db.ts
import { drizzle } from 'drizzle-orm/node-postgres';
import { migrate } from 'drizzle-orm/node-postgres/migrator';
import { Pool } from 'pg';

const pool = new Pool({ connectionString: env.DATABASE_URL });
export const db = drizzle(pool);

// Run on startup
await migrate(db, { migrationsFolder: './drizzle' });
```

## Row Level Security (RLS)

Define RLS policies in migrations, reference in schema:

```sql
-- Migration: 0004_enable_rls.sql
ALTER TABLE users ENABLE ROW LEVEL SECURITY;

CREATE POLICY "Users can view their own data"
  ON users FOR SELECT
  USING (auth.uid() = id);

CREATE POLICY "Users can update their own data"
  ON users FOR UPDATE
  USING (auth.uid() = id);
```

## Query Patterns

### Select with Relations

```typescript
import { eq } from 'drizzle-orm';

// Single record
const user = await db.query.users.findFirst({
  where: eq(users.id, userId),
  with: {
    subscription: true,
  },
});

// Multiple records
const logs = await db.query.usageLogs.findMany({
  where: eq(usageLogs.userId, userId),
  orderBy: (logs, { desc }) => [desc(logs.createdAt)],
  limit: 10,
});
```

### Insert

```typescript
const [user] = await db.insert(users)
  .values({
    email: 'user@example.com',
    supabaseId: authUser.id,
  })
  .returning();
```

### Update

```typescript
await db.update(users)
  .set({ updatedAt: new Date() })
  .where(eq(users.id, userId));
```

## Best Practices

- **Never use raw SQL** - always use Drizzle query builder
- **Use transactions** for multi-table operations
- **Index foreign keys** - Drizzle doesn't auto-create indexes
- **Add unique constraints** for business keys (email, stripe_customer_id)
- **Use prepared statements** for repeated queries
- **Avoid SELECT *** - specify columns explicitly

## Transaction Example

```typescript
await db.transaction(async (tx) => {
  await tx.insert(usageLogs).values({ userId, wordCount });
  await tx.update(users).set({ totalWords: user.totalWords + wordCount });
});
```
