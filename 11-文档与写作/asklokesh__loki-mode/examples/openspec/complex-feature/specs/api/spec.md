## ADDED Requirements

### Requirement: Tenant Request Routing
The system SHALL resolve the target tenant from each incoming API request using either the X-Tenant-ID header or a subdomain prefix, and inject the tenant context into the request pipeline before any business logic executes.

#### Scenario: Routing via header
- **GIVEN** an API request with header X-Tenant-ID set to "acme-corp"
- **WHEN** the request reaches the API gateway
- **THEN** the tenant context is set to "acme-corp"
- **AND** the request proceeds to the appropriate handler

#### Scenario: Missing tenant identifier
- **GIVEN** an API request with no X-Tenant-ID header and no subdomain prefix
- **WHEN** the request reaches the API gateway
- **THEN** the request is rejected with 400 Bad Request
- **AND** the error message indicates a missing tenant identifier

### Requirement: Tenant Quota Management
The system SHALL track cumulative API usage per tenant per billing period and enforce quota limits. When a tenant reaches 90% of its quota, the system SHOULD send a warning notification.

#### Scenario: Quota warning at 90%
- **GIVEN** tenant "acme-corp" with a quota of 100,000 requests per month
- **WHEN** the tenant reaches 90,000 requests
- **THEN** a quota warning notification is sent to the tenant administrator

#### Scenario: Quota exceeded
- **GIVEN** tenant "acme-corp" has exhausted its monthly quota
- **WHEN** the tenant sends an additional API request
- **THEN** the request is rejected with 429 Too Many Requests
- **AND** the response body includes quota reset time and upgrade options
