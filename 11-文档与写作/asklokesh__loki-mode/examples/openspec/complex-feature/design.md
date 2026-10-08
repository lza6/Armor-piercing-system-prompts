## Context

The platform currently operates as a single-tenant system. All users share one database schema, one set of API keys, and one billing account. The authentication system issues JWTs without tenant context. The admin dashboard displays aggregate metrics only.

## Goals / Non-Goals

**Goals:**
- Full data isolation between tenants at the database level
- Independent API key management per tenant
- Per-tenant rate limiting and quota enforcement
- Automated billing based on metered usage
- Zero cross-tenant data leakage

**Non-Goals:**
- Custom branding per tenant (future phase)
- Tenant self-service onboarding portal (admin-only for now)
- Multi-region tenant placement (single region initially)

## Decisions

### Decision: Row-Level Security over Separate Databases
Using PostgreSQL row-level security (RLS) with a tenant_id column on all tables. This approach avoids the operational complexity of managing separate databases per tenant while still providing strong isolation guarantees enforced at the database level.

### Decision: JWT Tenant Claims over Session Lookup
Encoding tenant_id directly in the JWT rather than performing a session lookup on each request. This reduces authentication latency and avoids an additional database round-trip. The trade-off is that tenant changes require token reissuance.

### Decision: API Gateway Tenant Resolution
Tenant context is resolved at the API gateway layer before requests reach application code. This centralizes tenant resolution logic and ensures all downstream services receive a validated tenant context.

## Risks / Trade-offs

- **Risk:** Row-level security misconfiguration could expose data across tenants. Mitigation: automated integration tests that verify cross-tenant isolation on every deployment.
- **Risk:** JWT tenant claims become stale if a user is moved between tenants. Mitigation: short token expiry (15 minutes) with refresh token rotation.
- **Trade-off:** Single-database multi-tenancy limits per-tenant database tuning. Acceptable for the initial launch targeting up to 100 tenants.
