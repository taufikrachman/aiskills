# MongoDB Schema Design

Design MongoDB collections for performance and maintainability.

## Rules

### 1. Embedding vs Referencing
```
Embed (one-to-few):  store child docs inside parent document
  { user: { ..., addresses: [{ street, city }] } }

Reference (one-to-many): store child IDs, populate on read
  { order: { ..., userId: ObjectId(...) } }

Reference (many-to-many): junction collection
  { user_roles: { userId, roleId } }
```
- Embed when: child data is always accessed with parent, <100 items, rarely changes independently.
- Reference when: shared across documents, grows unbounded, independently updatable.

### 2. Index Strategy
```js
// Compound index for common query patterns
db.orders.createIndex({ userId: 1, status: 1, createdAt: -1 });

// Text index for search
db.products.createIndex({ name: 'text', description: 'text' });

// TTL index for expiring data
db.sessions.createIndex({ createdAt: 1 }, { expireAfterSeconds: 604800 });
```
- Every query in WHERE must have index support.
- Use `explain()` to verify indexes are used.
- Avoid too many indexes (write performance hit).

### 3. Schema Patterns
- **Bucket**: group time-series data into hourly/daily docs (reduces index size).
- **Computed**: pre-aggregate counts/ averages, update on write (not read).
- **Polymorphic**: single collection for similar but different types (`type: 'book' | 'movie'`).

### 4. Mongoose Best Practices
```ts
const OrderSchema = new Schema({
  userId: { type: Schema.Types.ObjectId, ref: 'User', required: true, index: true },
  items: [{
    productId: { type: Schema.Types.ObjectId, required: true },
    quantity: { type: Number, required: true, min: 1 },
    price: { type: Number, required: true },
  }],
  totalAmount: { type: Number, required: true },
  status: { type: String, enum: ['pending', 'confirmed', 'cancelled'], default: 'pending' },
}, { timestamps: true });
```
- Use `timestamps: true` for automatic `createdAt`/`updatedAt`.
- Validate with enum or custom validators.
- Enable `{ strict: true }` to reject unknown fields.

## Anti-Patterns
- ❌ Embedding unbounded arrays (can exceed 16MB doc limit)
- ❌ No index on frequently queried fields
- ❌ `populate()` without index on foreign field
- ❌ Mongoose `find()` without `.lean()` for read-only queries
