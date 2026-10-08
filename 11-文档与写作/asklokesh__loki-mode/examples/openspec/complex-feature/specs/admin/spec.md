## MODIFIED Requirements

### Requirement: Admin Dashboard Tenant View
The admin dashboard SHALL display a tenant management section listing all active tenants with their usage metrics, plan tier, and status. Previously the admin dashboard showed only aggregate platform metrics without tenant segmentation.

#### Scenario: View tenant list
- **GIVEN** a platform administrator accessing the admin dashboard
- **WHEN** the administrator navigates to the Tenants section
- **THEN** a table lists all active tenants with name, plan, monthly usage, and status

#### Scenario: Drill into tenant detail
- **GIVEN** a platform administrator viewing the tenant list
- **WHEN** the administrator selects tenant "acme-corp"
- **THEN** a detail view shows API usage charts, active API keys count, and billing history
