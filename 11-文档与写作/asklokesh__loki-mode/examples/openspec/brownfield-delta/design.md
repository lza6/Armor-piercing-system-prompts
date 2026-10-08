## Context

The REST API (v2) has 80+ endpoints across 12 resource types. Mobile clients over-fetch by an average of 40% due to fixed response shapes. Three v1 endpoints remain deployed but have zero traffic. The client SDK is used by 15 internal services and 3 external partners.

## Goals / Non-Goals

**Goals:**
- Reduce client over-fetching through field-level selection
- Consolidate 80+ REST endpoints into a single GraphQL schema
- Remove zero-traffic deprecated endpoints
- Maintain SDK public API compatibility during transition

**Non-Goals:**
- Real-time subscriptions (future phase)
- Federation across microservices (single schema for now)
- Removing REST API entirely (coexistence during transition)

## Decisions

### Decision: Apollo Server over express-graphql
Apollo Server provides built-in schema validation, playground, performance tracing, and plugin support. The additional dependency size is justified by the operational tooling it enables.

### Decision: Schema-First over Code-First
Using SDL (Schema Definition Language) files as the source of truth for the GraphQL schema. This enables non-developer stakeholders to review the API contract and generates TypeScript types from the schema.

## Risks / Trade-offs

- **Risk:** Query complexity attacks. Mitigation: implement query depth limiting (max depth 7) and cost analysis middleware.
- **Risk:** Partner SDK breakage. Mitigation: 90-day dual-availability window where both REST and GraphQL are live, with partner notification and migration support.
- **Trade-off:** Caching is more complex with GraphQL than REST. Accept higher initial cache miss rates and implement persisted queries in a follow-up.
