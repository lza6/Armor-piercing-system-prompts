## ADDED Requirements

### Requirement: Tenant-Scoped Notifications
The system SHALL deliver notifications within tenant boundaries. Notifications triggered by tenant-specific events (quota warnings, billing, security alerts) MUST be routed only to administrators of the originating tenant.

#### Scenario: Quota warning routed to correct tenant
- **GIVEN** tenant "acme-corp" reaches 90% of its API quota
- **WHEN** the quota warning notification is generated
- **THEN** only administrators of "acme-corp" receive the notification
- **AND** administrators of other tenants are not notified

#### Scenario: Cross-tenant notification isolation
- **GIVEN** a security alert triggered for tenant "globex"
- **WHEN** the alert notification is dispatched
- **THEN** the notification appears only in the "globex" notification feed
- **AND** the notification is not visible to any other tenant
