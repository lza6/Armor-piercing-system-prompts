## ADDED Requirements

### Requirement: Tenant Authentication
The system SHALL authenticate users within the context of their assigned tenant, issuing JWT tokens that include a tenant_id claim. Users MUST NOT access resources outside their tenant boundary.

#### Scenario: Login with tenant context
- **GIVEN** a user assigned to tenant "acme-corp"
- **WHEN** the user authenticates with valid credentials
- **THEN** the issued JWT contains tenant_id "acme-corp"
- **AND** subsequent API requests are scoped to that tenant

#### Scenario: Cross-tenant access denied
- **GIVEN** a user authenticated under tenant "acme-corp"
- **WHEN** the user attempts to access a resource belonging to tenant "globex"
- **THEN** the request is rejected with 403 Forbidden

### Requirement: Tenant API Keys
The system SHALL allow tenant administrators to create, rotate, and revoke API keys scoped to their tenant. Each key MUST be cryptographically unique and associated with a single tenant.

#### Scenario: Create API key
- **GIVEN** a tenant administrator for "acme-corp"
- **WHEN** the administrator creates a new API key
- **THEN** a unique key is generated and associated with "acme-corp"
- **AND** the key is displayed once and cannot be retrieved again

#### Scenario: Revoke API key
- **GIVEN** an active API key for tenant "acme-corp"
- **WHEN** the tenant administrator revokes the key
- **THEN** all subsequent requests using that key are rejected with 401 Unauthorized

### Requirement: Tenant Rate Limiting
The system SHALL enforce per-tenant rate limits on API requests. When a tenant exceeds its rate limit, the system MUST return 429 Too Many Requests with a Retry-After header.

#### Scenario: Rate limit exceeded
- **GIVEN** tenant "acme-corp" with a rate limit of 1000 requests per minute
- **WHEN** the tenant sends the 1001st request within a minute
- **THEN** the response is 429 Too Many Requests
- **AND** a Retry-After header indicates when the limit resets
