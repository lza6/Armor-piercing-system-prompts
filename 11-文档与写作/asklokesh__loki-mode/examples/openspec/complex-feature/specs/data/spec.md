## ADDED Requirements

### Requirement: Tenant Data Isolation
The system SHALL isolate tenant data at the database level using row-level security. Queries MUST automatically filter by the authenticated tenant_id, preventing data leakage between tenants.

#### Scenario: Query returns only tenant data
- **GIVEN** tenant "acme-corp" has 50 records and tenant "globex" has 30 records
- **WHEN** a user authenticated as "acme-corp" queries all records
- **THEN** exactly 50 records are returned
- **AND** no records belonging to "globex" are included

#### Scenario: Insert scoped to tenant
- **GIVEN** a user authenticated under tenant "acme-corp"
- **WHEN** the user creates a new record
- **THEN** the record is stored with tenant_id "acme-corp"

### Requirement: Tenant Data Migration
The system SHALL provide tooling to migrate data when a tenant is onboarded or offboarded, including bulk import from CSV and full data export in JSON format.

#### Scenario: Bulk import on onboarding
- **GIVEN** a new tenant "initech" being onboarded
- **WHEN** an administrator uploads a CSV file with 10,000 records
- **THEN** all records are imported under tenant_id "initech"
- **AND** a migration report is generated with success and error counts
