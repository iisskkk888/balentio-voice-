# Security Guidelines

## JWT Configuration

### Token Types and Expiry

Use separate secrets for access and refresh tokens:

```typescript
// lib/env.ts
const envSchema = z.object({
  JWT_ACCESS_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
});

// lib/jwt.ts
export const JWT_CONFIG = {
  access: {
    secret: env.JWT_ACCESS_SECRET,
    expiresIn: '15m', // 15 minutes
  },
  refresh: {
    secret: env.JWT_REFRESH_SECRET,
    expiresIn: '30d', // 30 days
  },
};
```

### Token Generation

```typescript
import { sign } from 'hono/jwt';

export async function generateTokens(userId: number) {
  const payload = { userId, iat: Math.floor(Date.now() / 1000) };

  const accessToken = await sign(payload, JWT_CONFIG.access.secret, {
    exp: Math.floor(Date.now() / 1000) + 15 * 60, // 15 minutes
  });

  const refreshToken = await sign(payload, JWT_CONFIG.refresh.secret, {
    exp: Math.floor(Date.now() / 1000) + 30 * 24 * 60 * 60, // 30 days
  });

  return { accessToken, refreshToken };
}
```

### Token Validation

Always validate tokens on protected routes:

```typescript
import { verify } from 'hono/jwt';

export async function authMiddleware(c, next) {
  const token = c.req.header('Authorization')?.replace('Bearer ', '');

  if (!token) {
    return c.json({ error: 'Unauthorized' }, 401);
  }

  try {
    const payload = await verify(token, JWT_CONFIG.access.secret);
    c.set('userId', payload.userId);
    await next();
  } catch (err) {
    return c.json({ error: 'Invalid token' }, 401);
  }
}
```

## Token Storage (Desktop App)

Use secure native storage:

```typescript
// Desktop app - use Tauri secure storage
import { Store } from '@tauri-apps/plugin-store';

const store = new Store('.credentials.dat');

// Save tokens
await store.set('access_token', accessToken);
await store.set('refresh_token', refreshToken);

// Load tokens
const accessToken = await store.get('access_token');
```

**Never store tokens in:**
- localStorage
- sessionStorage
- Plain text files
- IndexedDB (unless encrypted)

## Environment Variables

### Secret Requirements

- **Minimum 32 characters** for JWT secrets
- **Generate with crypto**: `openssl rand -base64 32`
- **Never commit** .env files to git
- **Use different secrets** for development and production

### .env File Structure

```bash
# Database
DATABASE_URL="postgresql://user:pass@host:5432/db"

# JWT (separate secrets!)
JWT_ACCESS_SECRET="<32+ char random string>"
JWT_REFRESH_SECRET="<different 32+ char random string>"

# Stripe
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# ElevenLabs
ELEVENLABS_API_KEY="..."
```

## API Security Checklist

- ✅ **Validate all inputs** - use Zod schemas
- ✅ **Verify JWT on protected routes** - use authMiddleware
- ✅ **Use HTTPS only** - no HTTP in production
- ✅ **Rate limit endpoints** - prevent abuse
- ✅ **Sanitize error messages** - don't leak secrets
- ✅ **Enable CORS selectively** - only allow frontend domain
- ✅ **Verify Stripe webhook signatures** - see rules/stripe.md
- ✅ **Use parameterized queries** - Drizzle does this by default

## Password Handling (Supabase Auth)

Since Balentio Voice uses Supabase Auth, password hashing is handled automatically. However, if implementing custom auth:

```typescript
import { hash, compare } from 'bcrypt';

// Hash password (registration)
const hashedPassword = await hash(password, 10);

// Verify password (login)
const isValid = await compare(password, user.hashedPassword);
```

## Rate Limiting

```typescript
import { rateLimiter } from 'hono-rate-limiter';

app.use('/api/*', rateLimiter({
  windowMs: 15 * 60 * 1000, // 15 minutes
  limit: 100, // 100 requests per window
  standardHeaders: 'draft-6',
  keyGenerator: (c) => c.req.header('x-forwarded-for') || 'unknown',
}));
```

## Security Headers

```typescript
import { secureHeaders } from 'hono/secure-headers';

app.use('*', secureHeaders({
  contentSecurityPolicy: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'"],
  },
  xFrameOptions: 'DENY',
  xContentTypeOptions: 'nosniff',
}));
```
