---
name: wigolo-cache-first
description: Always check wigolo's local cache before making web requests.
---

# Cache-First Rule

Before ANY web search or fetch, check the cache:

```json
{ "query": "relevant keywords" }
```

Call the `cache` tool with the relevant keywords. If it has content, use it. If not, proceed to search/fetch.

Why: cached content is instant (0ms network), free (no network search query), and already extracted (clean markdown). A cache miss costs nothing — a redundant fetch wastes 5-15 seconds.

After fetching or searching, content is automatically cached with embeddings for future `find_similar` queries.

## Parameters

| Parameter | What it does |
|-----------|--------------|
| `query` | Full-text keyword search over cached bodies |
| `url_pattern` | URL glob filter, e.g. `"*example.com*"` |
| `since` | ISO date — only entries cached after this date |
| `clear` | Delete matching entries (requires a filter) |
| `stats` | Return total URLs, size, and date range |
| `check_changes` | Re-fetch matching URLs and report what changed |
| `mode` | `"fts"` (keyword, default) or `"hybrid"` (keyword + semantic) |
| `limit` | Max results to return (default 20) |

## Example

```json
// Step 1: check cache
cache({ "query": "oauth2 pkce", "url_pattern": "*auth0.com*" })

// Step 2: if empty, search
search({ "query": "oauth2 pkce flow", "include_domains": ["auth0.com"] })
```

Exceptions: `research` and `agent` check the cache internally — no pre-probe needed.
