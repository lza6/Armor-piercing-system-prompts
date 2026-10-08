## MODIFIED Requirements

### Requirement: User Data Retrieval
The system SHALL expose user profile data through a GraphQL query that accepts a user ID and returns only the fields requested by the client. Previously this was served by GET /api/v2/users/:id which returned a fixed JSON payload.

#### Scenario: Partial field selection
- **GIVEN** a client that needs only name and email for a user
- **WHEN** the client sends a GraphQL query requesting only those fields
- **THEN** the response contains only name and email
- **AND** no additional fields are transferred

### Requirement: Resource Listing
The system SHALL expose paginated resource listings through a GraphQL query supporting cursor-based pagination with first/after arguments. Previously this was served by GET /api/v2/resources with offset/limit query parameters.

#### Scenario: Cursor-based pagination
- **GIVEN** a collection of 500 resources
- **WHEN** the client requests the first 20 resources
- **THEN** the response includes 20 resource nodes and a pageInfo object with endCursor and hasNextPage

### Requirement: Resource Mutation
The system SHALL expose create and update operations through GraphQL mutations with input validation. Previously these were served by POST and PUT /api/v2/resources endpoints.

#### Scenario: Create resource via mutation
- **GIVEN** a valid resource input object
- **WHEN** the client sends a createResource mutation
- **THEN** the resource is created and the response includes the new resource with its generated ID

## REMOVED Requirements

### Requirement: Legacy User Search Endpoint
GET /api/v1/users/search is removed. This endpoint was deprecated in v2 and has had zero requests in the past 90 days. Consumers should use the GraphQL users query with filter arguments.

### Requirement: Legacy Bulk Export Endpoint
GET /api/v1/export/bulk is removed. This endpoint was superseded by the async export system in v2 and has no active consumers. The async export GraphQL mutation provides the same functionality with progress tracking.
