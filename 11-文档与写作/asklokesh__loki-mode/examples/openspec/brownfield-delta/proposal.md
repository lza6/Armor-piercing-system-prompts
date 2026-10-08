## Why

The current REST API has grown to over 80 endpoints with inconsistent naming, versioning challenges, and significant over-fetching on mobile clients. Migrating to GraphQL consolidates the API surface, enables clients to request exactly the data they need, and simplifies deprecation of legacy endpoints. Three deprecated REST endpoints have no active consumers and should be removed.

## What Changes

- Replace core REST endpoints with GraphQL resolvers
- Update client SDKs to use GraphQL queries instead of REST calls
- Remove deprecated REST endpoints that have zero active consumers
- Maintain backwards compatibility through a transition period with both APIs available

## Capabilities

### New Capabilities

### Modified Capabilities
- `api`: Core endpoints migrated from REST to GraphQL resolvers
- `clients`: Client SDK updated to use GraphQL transport

## Impact

- API gateway routing changes for /graphql endpoint
- Client SDK major version bump (breaking change)
- Monitoring and alerting must adapt to GraphQL query patterns
- API documentation must be regenerated for GraphQL schema
