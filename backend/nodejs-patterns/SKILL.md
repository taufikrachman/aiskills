# Node.js Backend Patterns

Production patterns for Node.js backends beyond frameworks.

## Rules

### 1. Error Handling
- Create custom error classes: `AppError`, `NotFoundError`, `ValidationError`, `AuthError`.
- Central error handler: catch → log → return appropriate HTTP status.
- NEVER `console.error` in production. Use structured logger (pino, winston).
- Log: timestamp, level, message, context (userId, requestId), stack trace.

### 2. Async Patterns
- Use `async/await` exclusively. No raw Promises, no callbacks.
- `Promise.allSettled()` when you want all results (partial failures OK).
- `Promise.all()` only when all must succeed — with timeout wrapper.
- Never fire-and-forget. Always handle promise rejections.

### 3. Rate Limiting & Throttling
- Global: 60-100 req/min per IP. Use token bucket or sliding window.
- Endpoint-specific: auth endpoints 5/min, heavy queries 10/min.
- Response header: `X-RateLimit-Remaining`, `Retry-After`.

### 4. Caching Strategy
- Cache-Control headers for GET endpoints.
- Redis for hot data: sessions, rate limiter counters, frequent queries.
- Cache invalidation: write-through (update cache on write) or TTL-based.
- NEVER cache user-specific data in shared cache.

### 5. Background Jobs (BullMQ)
```ts
@Processor('notifications')
class NotificationProcessor {
  @Process('welcome-email')
  async handleWelcome(job: Job<{ userId: string }>) {
    await emailService.sendWelcome(job.data.userId);
  }
}
```
- Use for: emails, push notifications, report generation, data exports.
- Configure retry with backoff: `attempts: 3, backoff: { type: 'exponential', delay: 5000 }`.
- Use separate queues for different priorities.

### 6. Graceful Shutdown
```ts
process.on('SIGTERM', async () => {
  await app.close();        // NestJS HTTP server
  await redis.disconnect(); // Close connections
  await db.destroy();       // Close DB pool
  process.exit(0);
});
```
