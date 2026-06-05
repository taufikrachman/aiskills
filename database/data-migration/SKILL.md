# Data Migration & Seeding

Safe database migrations and seed data strategies.

## Rules

### 1. Migration Rules
- Every migration MUST be reversible (write `down` method).
- Never modify a deployed migration. Create a new one.
- Test `up` AND `down` on staging before production.
- Use transactions for multi-statement migrations (PostgreSQL, not MySQL).
- Add comments explaining WHY, not just WHAT.

### 2. Naming Convention
```
20260605-1430-add-user-email-verification.ts
YYYYMMDD-HHmm-description.ts
```
Sortable by timestamp. Descriptive name.

### 3. Seed Data
- Seeds should be idempotent: `ON CONFLICT DO NOTHING` or check before insert.
- Separate `seed-dev` (fake data) from `seed-prod` (required reference data).
- Production seeds: roles, categories, plans, country/currency lists.
- Never seed user data in production.

### 4. Large Data Migrations
- Batch updates: process 1000 rows at a time, not all at once.
- Run during low-traffic window. Notify team before running.
- Lock tables only when absolutely necessary.
- For zero-downtime: add column → backfill in batches → make NOT NULL → drop old.

### 5. Rollback Strategy
```ts
// Always test rollback first
await migration.down(); // reverse
await migration.up();   // re-apply
// If up fails, down should restore original state
```
- Keep database backups before major migrations.
- Use `--dry-run` on TypeORM/Knex migrations to preview SQL.
