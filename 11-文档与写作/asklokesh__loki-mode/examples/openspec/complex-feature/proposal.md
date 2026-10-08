## Why

The platform currently serves a single organization. Enterprise customers require multi-tenant isolation to onboard their own teams, manage API access independently, and receive separate billing. Without multi-tenancy, each enterprise customer requires a dedicated deployment, increasing operational cost and limiting scalability.

## What Changes

- Introduce tenant-scoped authentication and API key management
- Add data isolation at the database and storage layers
- Implement per-tenant rate limiting and quota management
- Enable per-tenant billing and usage tracking
- Update the admin dashboard to surface tenant management controls
- Add tenant-scoped notification delivery

## Capabilities

### New Capabilities
- `auth`: Tenant authentication, API key lifecycle, and rate limiting per tenant
- `data`: Tenant data isolation and cross-tenant migration tooling
- `api`: Tenant-aware request routing and quota enforcement
- `billing`: Per-tenant usage metering and invoice generation
- `notifications`: Tenant-scoped notification channels and delivery

### Modified Capabilities
- `admin`: Admin dashboard updated to display tenant list, usage, and management actions

## Impact

- Database schema gains tenant_id foreign keys on all user-facing tables
- API gateway needs tenant resolution middleware
- Authentication service needs tenant-scoped token issuance
- Billing system needs metering pipeline and invoice generation
- Admin dashboard needs new tenant management views
- Notification service needs tenant routing
