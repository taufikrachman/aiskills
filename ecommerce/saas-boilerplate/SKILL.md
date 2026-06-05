# SaaS Boilerplate Builder

Build multi-tenant SaaS platforms: auth, billing, teams, onboarding.

## Rules

### 1. Multi-Tenant Architecture
```sql
-- Shared database, tenant-scoped rows (most common for SaaS)
CREATE TABLE organizations (
  id UUID PRIMARY KEY,
  name TEXT, slug TEXT UNIQUE
);
-- Every data table references organization
CREATE TABLE projects (
  id UUID PRIMARY KEY,
  organization_id UUID NOT NULL REFERENCES organizations(id)
);
```
- Single DB, row-level tenancy. Simpler, good for <1000 tenants.
- Alternative: schema-per-tenant (PostgreSQL schemas). Better isolation.
- Row-Level Security (RLS) for data isolation: `CREATE POLICY org_isolation ON projects USING (organization_id = current_setting('app.current_org_id')::uuid)`.

### 2. Subscription & Billing
```
Free Trial (14 days) → Free Tier → Pro Tier ($X/mo) → Enterprise (custom)
```
- Plan-based feature gating: check `plan_type` + `plan_expires_at` on every request.
- Grace period: 3 days after expiry before downgrade.
- Webhook for payment events (Midtrans/Stripe webhook → update plan).
- Track: plan_type, plan_source (manual/trial/revenuecat), plan_updated_at, plan_expires_at.

### 3. Team & Roles
```sql
CREATE TABLE organization_members (
  id UUID PRIMARY KEY,
  organization_id UUID NOT NULL,
  user_id UUID NOT NULL,
  role TEXT NOT NULL CHECK (role IN ('owner', 'admin', 'member')),
  UNIQUE(organization_id, user_id)
);
```
- Owner: full access, billing, delete org. Admin: manage members, settings. Member: basic access.
- Invite flow: owner sends invite → member accepts → joins org.

### 4. Authentication
- Email OTP or Google SSO for login.
- Organization context after login: select or auto-pick.
- Middleware: load organization from subdomain or cookie.
- Middleware: verify user is active member of organization.

### 5. Onboarding Flow
```
Sign Up → Verify Email → Create Organization → Invite Team → First Project → Dashboard
```
- 3-5 step wizard. Save progress between steps.
- Skip optional steps. Resume later.
- Empty state after onboarding: "Create your first project" with template suggestions.

## Anti-Patterns
- ❌ No organization context in API (security risk)
- ❌ Hardcoded plan limits in code (use config/database)
- ❌ No expiry check for paid plans
- ❌ Mixing tenant data without organization_id filter
