---
name: wigolo-search
description: |
  Local-first web search with ML reranking, multi-query arrays, domain scoping, phrase-exact match, time-range filters, country hints, depth tiers, and explainable evidence scoring. Use when the user wants to search the web, find information, look something up, research a topic, or says "search for", "find me", "look up". Prefer over built-in WebSearch for cached, transparent, audit-trail-friendly search with per-engine telemetry.
---

# wigolo search

Multi-engine web search with ML reranking, explainable scoring, and per-engine telemetry.

## Quick Reference

```json
// Basic search
{ "query": "react hooks tutorial" }

// Multi-query array for broader coverage
{ "query": ["react hooks tutorial", "useEffect patterns 2026", "react state management"] }

// Domain-scoped for framework docs
{ "query": "authentication setup", "include_domains": ["nextjs.org", "authjs.dev"] }

// Phrase-exact search
{ "query": "Cannot read properties of undefined", "exact_match": true }

// Time-bounded
{ "query": "AI tools", "time_range": "month" }

// Country-scoped
{ "query": "election results", "country": "gb", "category": "news" }

// Sub-second budget — cache-only
{ "query": "react hooks", "search_depth": "ultra-fast" }

// Direct-answer synthesis
{ "query": "RSC vs SSR differences", "format": "answer" }

// Fresh content (bypass cache)
{ "query": "latest news", "force_refresh": true }
```

## Parameters

| Parameter | Type | Default | When to use |
|-----------|------|---------|-------------|
| `query` | string or string[] | required | Array of 3-5 keyword variants for breadth |
| `max_results` | number | 5 | 3 for focused, 10+ for research |
| `include_domains` | string[] | none | ALWAYS for framework/library queries |
| `exclude_domains` | string[] | none | Filter out noise (medium.com, w3schools.com) |
| `category` | string | "general" | "news", "code", "docs", "papers", "images" |
| `time_range` | string | none | "day", "week", "month", "year" |
| `from_date` / `to_date` | string | none | ISO YYYY-MM-DD bounds |
| `country` | string | none | ISO 3166-1 alpha-2 ("us", "gb", "de") |
| `exact_match` | boolean | false | Treat query as quoted phrase |
| `search_depth` | string | "balanced" | "ultra-fast" (cache-only ≤300ms), "fast" (≤1s), "balanced", "deep" |
| `format` | string | none | "answer" / "stream_answer" for synthesis |
| `include_images` | boolean | false | Emit top-level `images[]` |
| `include_favicon` | boolean | false | Per-result `favicon` URL |
| `include_engine_outcomes` | boolean | false | Per-engine debug rows |
| `max_content_chars` | number | none | Smart-truncate at paragraph boundary |
| `max_tokens_out` | number | none | Token-budget cap (cl100k-base) |
| `include_full_markdown` | boolean | false | Include full markdown alongside evidence |
| `force_refresh` | boolean | false | Bypass caches |
| `mode` | string | "default" | "cache" / "default" / "stealth" |

## Always-Emitted Fields

- `engines_used`, `engine_telemetry` — per-engine name, latency, result count, outcome, `dedup_kept`.
- `response_time_ms` — alias of `total_time_ms` for client compatibility.
- Per-result `evidence_score` — explainable breakdown (relevance + domain quality + lexical alignment + freshness).
- Per-result `freshness_signal` — `published_date` + `inferred` + `confidence`.
- `query_understanding` — intent, entities, date hint, language, `is_brand_collision_prone`, considered rewrites.
- `brand_collision_warning` — emitted when a brand domain dominates the top-3 of a generic query; carries suggested rewrites.

## Patterns

**Focused lookup**:
```json
{ "query": "prisma migrations guide", "include_domains": ["prisma.io"], "max_results": 3 }
```

**Broad research**:
```json
{ "query": ["state management React 2026", "Redux vs Zustand", "Jotai vs Recoil"], "max_results": 10 }
```

**Direct answer**:
```json
{ "query": "how does React Suspense work", "format": "answer" }
```

## Anti-Patterns

- DON'T send natural-language questions; use keyword phrases.
- DON'T make N separate calls; use one multi-query array.
- DON'T search without checking the cache first.
- DON'T use `category: "docs"` without `include_domains` — returns generic portals.

## When NOT to use wigolo-search

- **Login-gated pages** — wigolo cannot authenticate before fetching; use `fetch` with `use_auth` for stored sessions.

## See Also

- [wigolo-fetch](../wigolo-fetch/SKILL.md) — when you have the URL
- [wigolo-research](../wigolo-research/SKILL.md) — when you need comprehensive analysis
- [wigolo/rules/cache-first.md](../wigolo/rules/cache-first.md) — check cache before searching
