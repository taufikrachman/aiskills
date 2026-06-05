# Redis Caching Strategy

Use Redis for caching, sessions, rate limiting, and real-time features.

## Rules

### 1. Cache Patterns
```
Cache-Aside: app checks cache → miss → fetch DB → write to cache → return
Write-Through: app writes to cache + DB simultaneously
Write-Behind: app writes to cache → async write to DB
```
- Use Cache-Aside for read-heavy data.
- TTL: products 1h, user sessions 7d, rate limit counters 1m.

### 2. Key Naming Convention
```
{namespace}:{entity}:{id}
user:session:abc123
product:list:category-5
rate-limit:login:192.168.1.1
```
Use namespaces for: easy invalidation (SCAN/DELETE by prefix).

### 3. Session Store
```ts
// Store session with TTL
await redis.set(`session:${sessionId}`, JSON.stringify(session), 'EX', 604800);
// Invalidate on logout
await redis.del(`session:${sessionId}`);
```

### 4. Rate Limiter
```ts
const key = `rate-limit:${userId}:${Date.now() / 60000 | 0}`; // Per-minute
const count = await redis.incr(key);
await redis.expire(key, 60);
if (count > 60) throw new TooManyRequestsException();
```

### 5. Caching Anti-Patterns
- ❌ No TTL (memory leak)
- ❌ Caching user-specific data globally
- ❌ Redis as primary data store (it's a cache, not a DB)
- ❌ `KEYS *` in production (use `SCAN`)
