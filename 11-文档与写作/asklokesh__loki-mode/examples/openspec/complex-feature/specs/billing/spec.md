## ADDED Requirements

### Requirement: Per-Tenant Billing
The system SHALL meter API usage per tenant and generate monthly invoices based on the tenant's pricing plan. Invoices MUST itemize usage by endpoint category (read, write, admin) and include applicable taxes.

#### Scenario: Monthly invoice generation
- **GIVEN** tenant "acme-corp" on the Pro plan with usage during the billing period
- **WHEN** the billing period closes on the first of the month
- **THEN** an invoice is generated with itemized usage by endpoint category
- **AND** the invoice is delivered to the tenant administrator via email

#### Scenario: Zero-usage month
- **GIVEN** tenant "initech" with no API usage during the billing period
- **WHEN** the billing period closes
- **THEN** an invoice is generated showing zero usage
- **AND** only the base plan fee (if applicable) is charged
