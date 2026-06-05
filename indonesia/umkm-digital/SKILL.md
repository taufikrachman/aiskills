# UMKM Digital Toolkit 🇮🇩

Build digital tools for Indonesian MSMEs (UMKM): warung, toko kelontong, bengkel, salon.

## Core Rules

### 1. Local Context
- Names: use Indonesian examples (Warung Bu Siti, Toko Makmur). Not "John's Store".
- Currency: always IDR (Rp). Use dot as thousand separator: Rp 50.000.
- Date: DD/MM/YYYY. Time: 24-hour or WIB/WITA/WIT timezone.
- Phone: Indonesian format (+62, 08xx). Validate Indonesian numbers: 10-13 digits.

### 2. Offline-First Architecture
UMKM often have unreliable internet. ALL apps must work offline.
- Local SQLite database. Sync to server when online.
- Queue mutations offline → push on reconnect.
- Conflict resolution: last-write-wins for simple cases.
- Show sync status indicator (online/offline/syncing).
- Initial sync: download all data once. Delta sync: only changes.

### 3. Common UMKM Features
- **Pencatatan Penjualan**: simple POS, barcode scan (optional), quick add products.
- **Stok Barang**: stock in/out, low stock alert, stock adjustment.
- **Laporan Sederhana**: daily sales, monthly profit, best-selling products.
- **Hutang/Piutang**: customer credit book, payment tracking.
- **Multi-Toko**: multiple outlets/carts, owner sees all.
- **Kasir/Tim**: add cashiers, role-based access (owner vs staff).

### 4. Indonesian Payment Methods
- Cash (tunai) — default, always supported.
- QRIS — QR code payment, standard in Indonesia. Show QR, wait for confirmation.
- Transfer bank — manual confirmation by staff.
- GoPay/OVO/Dana — e-wallet integration (via Midtrans/Xendit).

### 5. Indonesian Tax (Pajak UMKM)
- PPh Final UMKM: 0.5% of gross revenue (if revenue < Rp 4.8B/year).
- Include tax in transaction data: `taxAmount`, `taxRate`.
- Tax report: export CSV for tax filing.
- PPn (VAT) 11%: optional, for PKP-registered businesses.

### 6. WhatsApp Integration
- Share receipt via WhatsApp (most common in Indonesia).
- WhatsApp Business API for order notifications.
- Click-to-chat: `https://wa.me/628xxxxxxxxxx?text=...`.
- Auto-reply template for orders.

## Anti-Patterns
- ❌ Assuming always-on internet
- ❌ English-only UI (use Bahasa Indonesia)
- ❌ Complex accounting (this is for UMKM, not enterprise)
- ❌ Credit card as default payment (QRIS/cash is primary in Indonesia)
