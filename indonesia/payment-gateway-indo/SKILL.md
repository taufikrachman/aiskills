# Payment Gateway Indonesia 🇮🇩

Integrate Indonesian payment gateways: Midtrans, Xendit, Duitku, QRIS.

## Rules

### 1. Provider Selection
| Gateway | Best For | Settlement | Support |
|---------|----------|-----------|---------|
| **Midtrans** | General e-commerce, most popular | T+1 to T+3 | Good docs |
| **Xendit** | Subscription, marketplace | T+1 | Developer-friendly |
| **Duitku** | UMKM, simpler setup | T+1 | Low fee |
| **QRIS** | Offline retail, warung | Instant | Universal |

### 2. Payment Flow Pattern
```ts
// 1. Create transaction on your backend
const orderId = `INV-${Date.now()}`;
const tx = await midtrans.createTransaction({
  order_id: orderId,
  gross_amount: 50000,
  customer_details: { first_name: 'Budi', email: 'budi@email.com', phone: '08123456789' },
  item_details: [{ id: 'ITEM1', price: 50000, quantity: 1, name: 'Product A' }],
});

// 2. Get payment token → use in frontend
const token = tx.token; // Midtrans Snap token

// 3. Handle callback
// Midtrans sends HTTP POST to your /api/payment/callback
// Verify signature, update order status, trigger fulfillment
```

### 3. Webhook Handling
```ts
@Post('payment/callback')
async handleMidtransCallback(@Body() payload: any, @Headers() headers: Record<string, string>) {
  // 1. Verify signature (prevents fraudulent callbacks)
  const isValid = midtrans.verifySignature(payload, headers['x-midtrans-signature']);
  if (!isValid) throw new BadRequestException('Invalid signature');

  // 2. Idempotency check — don't process same callback twice
  const existing = await this.findByOrderId(payload.order_id);
  if (existing?.status === payload.transaction_status) return { success: true };

  // 3. Update order + fulfill
  await this.updateOrder(payload.order_id, {
    status: mapMidtransStatus(payload.transaction_status),
    paymentMethod: payload.payment_type,
    paidAt: new Date(payload.settlement_time || payload.transaction_time),
  });
}
```
NEVER trust callback payload without signature verification.

### 4. QRIS Specifics
- NMID: obtained from payment gateway (Midtrans/Xendit).
- QR generation: use gateway API, not generate yourself.
- Status polling: QRIS is async. Poll every 3s for 2 minutes max.
- Static vs Dynamic QR: dynamic preferred (amount is embedded).
- Offline QR: store QR image locally, show even when offline.

### 5. Error & Edge Cases
- Pending payment: show "Menunggu Pembayaran" with expiration timer.
- Expired: cancel order, release inventory.
- Failed: show reason, offer retry with different method.
- Refund: full or partial. Track refund status.
- Chargeback: handle dispute, provide evidence.

### 6. Reconciliation
- Daily settlement report: match gateway report with internal orders.
- Track: order ID → payment status → settlement date → settlement amount.
- Discrepancies: admin fee, rounding, failed settlements.
- Export CSV for accounting.

## Anti-Patterns
- ❌ No signature verification on callbacks
- ❌ Processing same callback twice (no idempotency)
- ❌ Storing payment credentials (PAN, CVV)
- ❌ Client-side amount validation (always verify on server)
