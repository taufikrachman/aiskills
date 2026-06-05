# POS System Architecture

Build Point of Sale systems: offline-first, real-time sync, multi-store.

## Rules

### 1. Core Architecture
```
Mobile App (Flutter/RN) ←→ Backend API (NestJS) ←→ Database (PostgreSQL)
        ↕                          ↕
   Local SQLite DB            Redis (caching, queue)
```
- Local DB for offline: product catalog, transactions, settings.
- Background sync: push pending → pull changes.
- Conflict resolution: server wins for stock, last-write-wins for products.

### 2. Sync Protocol
```
1. Push: send local changes to server
2. Pull: download server changes since last sync
3. Delta: only changed data (use `updated_at >= lastSyncTime`)
```
- Full sync on: fresh install, re-login, data corruption.
- Delta sync on: periodic timer, connectivity restored, after mutation.
- Batch API for efficiency: `POST /sync-logs/batch` (max 50).

### 3. Transaction Lifecycle
```
Create Cart → Checkout → Save Local → Queue Sync → Push to Server → Confirm → Pull Back
```
- Save to local DB IMMEDIATELY (offline-first).
- Queue sync event. Process in background.
- On push success: update local with server ID (local_xxx → UUID).
- On push fail: retry with backoff (2s → 4s → 8s). Max 3 retries.

### 4. Multi-Store Support
- Store-scoped queries: `WHERE store_id = ?` on every query.
- Store selector in UI. Switching stores triggers re-sync.
- Plan-based limits: Free = 1 store, Pro = unlimited.
- Store roles: Owner (full access), Cashier (POS only).

### 5. Receipt Printing
- 80mm thermal paper format.
- Generate PDF locally → share via WhatsApp/printer.
- QR codes for digital receipts.
- Receipt data: store info, items, subtotal, discounts (item + transaction), tax, total, payment.

## Anti-Patterns
- ❌ Assuming internet for transaction creation
- ❌ No local backup before sync
- ❌ Full sync on every transaction (use delta)
- ❌ Hardcoded store ID (always from secure storage)
