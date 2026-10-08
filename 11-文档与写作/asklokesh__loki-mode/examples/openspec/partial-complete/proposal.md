## Why

Users currently have no way to search across their content. As data grows, manual browsing becomes impractical. Full-text search with filtering and ranking is needed to help users find relevant items quickly. This has been the top-requested feature for two consecutive quarters.

## What Changes

- Add a search index backed by Elasticsearch
- Implement a search API endpoint with query, filter, and pagination support
- Build a search UI with auto-complete, result highlighting, and faceted filters

## Capabilities

### New Capabilities
- `search`: Full-text search indexing, query API, and search UI components

### Modified Capabilities

## Impact

- New Elasticsearch cluster dependency in infrastructure
- Indexing pipeline needs to process existing and new content
- Frontend gains a search bar component in the global navigation
- API adds search endpoint with query parameter validation
