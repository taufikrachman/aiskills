# MySQL Query Master

Production MySQL/MariaDB patterns for high-performance queries.

## Rules

### 1. Storage Engine
- **InnoDB** (default): ACID transactions, row-level locking, foreign keys. Use for everything.
- **MyISAM**: no transactions, table-level locking. Avoid unless legacy.
- Always set: `ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_unicode_ci`.

### 2. Index Strategy
```sql
-- Single column index
CREATE INDEX idx_status ON orders(status);

-- Composite index (order matters: leftmost prefix)
CREATE INDEX idx_user_status_date ON orders(user_id, status, created_at DESC);

-- Covering index (no table lookup needed)
CREATE INDEX idx_cover ON orders(user_id, status, created_at, total_amount);
```
- Index every foreign key and frequently filtered column.
- Use `EXPLAIN SELECT ...` to verify index usage. Look for `type: ref` or `type: range`.
- Avoid `type: ALL` (full table scan on large tables).
- `SHOW INDEX FROM table` to see existing indexes.

### 3. Query Optimization
```sql
-- Use specific columns, not SELECT *
SELECT id, name, email FROM users WHERE status = 'active';

-- Use LIMIT with ORDER BY for pagination
SELECT * FROM orders ORDER BY created_at DESC LIMIT 50 OFFSET 0;

-- Avoid OFFSET for large offsets — use keyset pagination
SELECT * FROM orders WHERE created_at < '2026-06-01' ORDER BY created_at DESC LIMIT 50;

-- Use JOINs correctly
SELECT u.name, o.total     -- INNER JOIN: matching rows only
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

SELECT u.name, COUNT(o.id) -- LEFT JOIN: all users, even without orders
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id;
```

### 4. Common Pitfalls
- `WHERE DATE(created_at) = '2026-06-05'` — function prevents index use. Use range: `WHERE created_at >= '2026-06-05' AND created_at < '2026-06-06'`.
- `WHERE status != 'deleted'` — negative condition often skips index. Use `WHERE status IN ('active', 'pending')`.
- `WHERE name LIKE '%keyword%'` — leading wildcard prevents index. Use FULLTEXT index instead.
- No index on foreign key → slow JOINs.

### 5. Character Set & Collation
```sql
CREATE DATABASE myapp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
```
- `utf8mb4`: full Unicode (includes emoji). NOT `utf8` (3-byte only).
- `utf8mb4_unicode_ci`: case-insensitive, accent-insensitive. For most apps.
- `utf8mb4_bin`: case-sensitive. For unique usernames/emails.

### 6. Migration Safety
- Use transactions for DDL where supported (InnoDB: `ALTER TABLE` is transactional in MySQL 8.0+).
- Test on staging with production-size data.
- For large tables: use `pt-online-schema-change` (Percona) for zero-downtime migrations.
- Never `ALTER TABLE` directly on a 10M+ row table during peak hours.

## Anti-Patterns
- ❌ MyISAM for transactional data
- ❌ `SELECT *` in production queries
- ❌ No index on JOIN columns
- ❌ `utf8` charset (use `utf8mb4`)
- ❌ Direct `ALTER TABLE` on large tables without testing
