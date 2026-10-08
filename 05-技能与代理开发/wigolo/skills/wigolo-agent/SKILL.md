---
name: wigolo-agent
description: |
  Autonomous data gathering across sources — plans search queries and URLs from a natural-language prompt, executes in parallel within a time budget, optionally extracts structured fields via JSON Schema, and synthesizes results with full step transparency. Use when the user needs data collected from the web with a specific shape, says "gather data", "find pricing for", "collect information about", "extract from multiple sites", or provides a JSON schema for web data.
license: AGPL-3.0-only
metadata:
  author: KnockOutEZ
  version: 0.1.43-beta.2
  homepage: https://github.com/KnockOutEZ/wigolo
  repository: https://github.com/KnockOutEZ/wigolo
---

# wigolo agent

Natural-language data gathering with optional JSON Schema output. Local-first: every fetched page lands in the cache for later reuse.

## Quick Reference

```json
// Natural language data gathering
{ "prompt": "Find pricing tiers for the top 5 headless CMS platforms" }

// With structured output schema
{
  "prompt": "Find pricing for Contentful, Sanity, and Strapi",
  "schema": { "type": "object", "properties": { "name": { "type": "string" }, "free_tier": { "type": "string" }, "pro_price": { "type": "string" }, "enterprise": { "type": "string" } } }
}

// With starting URLs
{
  "prompt": "Compare features across these CMS platforms",
  "urls": ["https://contentful.com/pricing", "https://sanity.io/pricing"],
  "max_pages": 6
}
```

## Parameters

| Parameter | Type | Default | When to use |
|-----------|------|---------|-------------|
| `prompt` | string | required | Natural-language task description |
| `urls` | string[] | none | Seed URLs to include |
| `schema` | object | none | JSON Schema for structured extraction per page |
| `max_pages` | number | 10 | Hard cap on pages fetched (max 100) |
| `max_time_ms` | number | 60000 | Time budget in ms (max 600000) |
| `stream` | boolean | false | Emit progress notifications per step |
| `max_tokens_out` | number | none | Token-budget cap (cl100k-base) |
| `include_full_markdown` | boolean | false | Pages return evidence excerpts by default |
| `citation_format` | string | "numbered" | "numbered" / "json" / "anthropic_tags" |

## How It Works

1. **Plans** — interprets prompt, generates search queries and URLs.
2. **Executes** — searches and fetches in parallel within budget.
3. **Extracts** — if schema provided, extracts fields from each page and merges.
4. **Synthesizes** — produces natural-language result or structured data.
5. **Reports** — `steps` array shows every action with timings.

Synthesis follows a fallback ladder: host sampling → optional local language model → deterministic extraction.

## Output Transparency

Every response includes a `steps` array:

```json
[
  { "action": "plan", "detail": "Generated 3 search queries", "time_ms": 200 },
  { "action": "search", "detail": "Found 8 results", "time_ms": 5000 },
  { "action": "fetch", "detail": "Fetched 5 pages", "time_ms": 8000 },
  { "action": "extract", "detail": "Extracted schema from 5 sources", "time_ms": 3000 }
]
```

Use `steps` to debug weak results — if extraction is poor, check which pages were fetched.

## Anti-Patterns

- DON'T use for reports/analysis — use `research` instead.
- DON'T use for single-page extraction — use `extract` instead.
- DON'T set `max_pages` high without time budget — set `max_time_ms` too.

## When NOT to use wigolo-agent

- **Per-page interactive flow needed (login, multi-step wizard, click-through pagination)** — handle authentication or interaction externally, then chain with wigolo's `extract`.

## See Also

- [wigolo-extract](../wigolo-extract/SKILL.md) — for single-page extraction
- [wigolo-research](../wigolo-research/SKILL.md) — for reports and analysis (not data gathering)
