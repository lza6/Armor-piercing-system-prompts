## MODIFIED Requirements

### Requirement: Client SDK Transport
The client SDK SHALL use GraphQL as its transport layer, constructing typed queries from method calls. Previously the SDK made direct REST calls with URL construction and JSON parsing. The SDK public API (method names and return types) remains unchanged to minimize consumer migration effort.

#### Scenario: SDK method uses GraphQL internally
- **GIVEN** a consumer calling sdk.getUser(userId)
- **WHEN** the SDK processes the call
- **THEN** a GraphQL query is sent to the /graphql endpoint
- **AND** the response is mapped to the same UserProfile type the consumer expects

#### Scenario: SDK handles GraphQL errors
- **GIVEN** a GraphQL response containing an errors array
- **WHEN** the SDK receives the response
- **THEN** the errors are mapped to the existing SDK exception types
- **AND** the consumer sees the same error interface as before
