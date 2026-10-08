---
name: wigolo-cache
description: |
  Local-first knowledge cache — full-text and hybrid semantic search over every page wigolo has already fetched, crawled, or searched. Use before any web request: cached hits return instantly and free. Triggers when the user says "check the cache", "have we seen this", "what's on disk", "search what I've already fetched", "cache stats", or "what changed since I last looked". Also clears matching entries and re-checks cached URLs for changes.
license: AGPL-3.0-only
metadata:
  author: KnockOutEZ
  version: 0.1.43-beta.2
  homepage: https://github.com/KnockOutEZ/wigolo
  repository: https://github.com/KnockOutEZ/wigolo
---

# wigolo cache

The local knowledge store. Every page wigolo fetches, crawls, or searches lands here with clean markdown and embeddings, so the second read is instant and free. **Check the cache before any `search` or `fetch`.**

## Quick Reference

```json
// Keyword search over cached content
{ "query": "oauth2 pkce" }

// Scope to a URL glob
{ "query": "pkce", "url_pattern": "*auth0.com*" }

// Higher-recall hybrid (keyword + semantic) lookup
{ "query": "token refresh rotation", "mode": "hybrid" }

// Only entries cached after a date
{ "query": "release notes", "since": "2026-06-01" }

// Cache statistics (no query needed)
{ "stats": true }

// Re-fetch matching cached URLs and report what changed
{ "url_pattern": "*docs.example.com*", "check_changes": true }

// Clear matching entries (requires at least one filter)
{ "url_pattern": "*staging.example.com*", "clear": true }
```

## Parameters

| Parameter | Type | Default | When to use |
|-----------|------|---------|-------------|
| `query` | string | none | Full-text keyword search over cached bodies |
| `url_pattern` | string | none | URL glob filter, e.g. `"*example.com*"` |
| `since` | string | none | ISO date — only entries cached after this date |
| `clear` | boolean | false | Delete matching entries (needs query, url_pattern, or since) |
| `stats` | boolean | false | Return total URLs, size, and date range |
| `check_changes` | boolean | false | Re-fetch matching URLs, report changed/unchanged + diff summary |
| `mode` | string | "fts" | "fts" (keyword BM25) or "hybrid" (keyword + semantic, fused) |
| `limit` | number | 20 | Max results to return |
| `max_tokens_out` | number | none | Token-budget cap on returned bodies (cl100k-base) |

## The Cache-First Workflow

Cached content is instant (0ms network), free (no engine query), and already extracted (clean markdown). A miss costs nothing; a redundant fetch wastes 5-15 seconds. So the first move is always:

```json
// Step 1 — probe the cache
cache({ "query": "server islands hydration", "url_pattern": "*docs.astro.build*" })

// Step 2 — miss? fall through to search or fetch
search({ "query": "astro server islands", "include_domains": ["docs.astro.build"] })
```

## Modes

- **`fts`** (default) — keyword BM25 over the local full-text index. Fast, exact-term.
- **`hybrid`** — additionally runs semantic vector search and fuses both rankings with reciprocal rank fusion for higher recall. Falls back to keyword search when the embedding index is empty or unavailable.

## Stats and Change Detection

`stats: true` reports total URLs, on-disk size, and the cached date range — useful to gauge whether `find_similar` has enough local signal yet.

`check_changes: true` re-fetches the matching cached URLs and reports which ones changed, with a per-URL diff summary. Scope it with `query` or `url_pattern` so you only re-check what matters.

## Anti-Patterns

- DON'T search or fetch before probing the cache — you may already have it.
- DON'T pass `clear: true` without a filter — the handler refuses an unscoped wipe.
- DON'T expect `mode: "hybrid"` to help on a cold cache — warm it with `crawl` first.

## When NOT to use wigolo-cache

- **Content you have never fetched** — the cache only holds what wigolo has already read; use `search` or `fetch` to bring it in first.

## See Also

- [wigolo-search](../wigolo-search/SKILL.md) — when the cache misses
- [wigolo-crawl](../wigolo-crawl/SKILL.md) — warm the cache in bulk
- [wigolo-diff](../wigolo-diff/SKILL.md) — compare a live page against its cached copy
- [wigolo-find-similar](../wigolo-find-similar/SKILL.md) — discover related cached pages
- [wigolo/rules/cache-first.md](../wigolo/rules/cache-first.md) — the cache-first rule
