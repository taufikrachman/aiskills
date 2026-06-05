# Payment Integration

Integrate payment gateways: Midtrans, Stripe, Xendit. Production patterns.

## Rules

### 1. Architecture
```
Frontend → Backend (create transaction) → Payment Gateway → Webhook → Backend (verify) → Fulfill
```
- NEVER process payment on client side. Always through backend.
- Frontend only handles UI (Snap popup, redirect, QR display).
- Backend creates transaction, handles callback, verifies signature.

### 2. Idempotency
```ts
// Prevent double-charge on retry
const existing = await findByOrderId(orderId);
if (existing) return existing; // Return existing transaction, don't create new one

const tx = await gateway.createTransaction({ order_id: orderId, ... });
await saveTransaction({ orderId, gatewayTxId: tx.id, status: 'pending' });
```
- Use unique `order_id` (ULID/UUID). Gateway ensures one-time processing.
- Store gateway transaction ID for reconciliation.

### 3. Webhook Signature Verification
```ts
function verifySignature(payload: any, signature: string): boolean {
  const computed = crypto.createHash('sha512')
    .update(orderId + statusCode + grossAmount + serverKey)
    .digest('hex');
  return computed === signature;
}
// Midtrans: SHA512(orderId + statusCode + grossAmount + serverKey)
// Xendit: HMAC-SHA256 with webhook verification token
```
NEVER skip this. Anyone can POST to your callback URL.

### 4. Payment States
```
pending → settlement/capture → success
pending → deny/cancel → failed
pending → expire → expired
success → refund → refunded
```
- Map gateway-specific statuses to your app's status enum.
- Handle all possible states, not just success/fail.

### 5. Error Handling
- Timeout (30s): retry creating transaction with same order_id.
- Callback timeout: poll gateway API every 10s for 2 minutes.
- Reconciliation: daily cron job to find mismatched orders.
- Never assume payment succeeded without callback.

## Anti-Patterns
- ❌ No signature verification on callback
- ❌ No idempotency (double charge risk)
- ❌ Client-side amount calculation (always server-side)
- ❌ Polling gateway every second (rate limit)
