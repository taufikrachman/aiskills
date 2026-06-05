# Monitoring & Logging

Production observability: structured logging, error tracking, health checks.

## Rules

### 1. Structured Logging
```ts
// Always structured, never string interpolation
logger.info('Order created', { orderId, userId, amount, duration: Date.now() - start });
// ❌ logger.info(`Order ${orderId} created for user ${userId}`);
```
Log format: ISO timestamp, level, message, context object.
Levels: `error` (needs action), `warn` (watch), `info` (normal), `debug` (dev only).

### 2. Health Check Endpoint
```ts
@Get('health')
async health() {
  const db = await this.db.query('SELECT 1');  // DB check
  const redis = await this.redis.ping();        // Redis check
  return { status: 'ok', timestamp: new Date(), checks: { db, redis } };
}
```
Responds within 2 seconds. Returns JSON. Used by load balancer and Docker healthcheck.

### 3. Error Tracking
- Capture: unhandled rejections, uncaught exceptions, API errors > 500.
- Context: userId, requestId, route, error stack.
- Alert on: error rate spike (> 5% of requests), DB connection failure, payment failure.
- Tools: Sentry, GlitchTip (self-hosted), or custom error_logs table.

### 4. Performance Monitoring
- Track: request duration, DB query time, external API call time.
- Alert on: p95 response time > 2s, DB pool exhaustion.
- Dashboard: request rate, error rate, response time percentiles.

### 5. Alert Thresholds
```
🟢 p95 latency < 500ms
🟡 p95 latency 500ms-2s (watch)
🔴 p95 latency > 2s (alert)
🔴 Error rate > 5% (alert immediately)
🔴 Health check fails (alert immediately)
```
Alert via: Telegram, Discord, email. Don't alert on transient spikes — use 5-minute average.

## Anti-Patterns
- ❌ `console.log` in production
- ❌ No health check endpoint
- ❌ Alerting on every single error (alert fatigue)
- ❌ Logging PII (emails, phone numbers) in plain text
