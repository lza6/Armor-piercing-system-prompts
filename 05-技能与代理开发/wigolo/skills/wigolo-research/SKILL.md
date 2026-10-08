---
name: wigolo-research
description: |
  Local-first multi-step research with question decomposition, parallel search, structured briefs, cross-references, and gap analysis. Use when the user needs comprehensive analysis, comparison reports, literature reviews, or says "research", "compare X vs Y", "deep dive", "thorough analysis", "find everything about". Returns a structured `brief` with `topics`, `highlights`, `key_findings`, `sections.overview.cross_references`, `sections.comparison`, `sections.gaps`.
license: AGPL-3.0-only
metadata:
  author: KnockOutEZ
  version: 0.1.43-beta.2
  homepage: https://github.com/KnockOutEZ/wigolo
  repository: https://github.com/KnockOutEZ/wigolo
---

# wigolo research

Comprehensive multi-source research with structured output. Beats chaining `search` + `fetch` manually for multi-source synthesis.

## Quick Reference

```json
// Standard research
{ "question": "How does Deno 2 compare to Node.js for production?", "depth": "standard" }

// Comprehensive (more sources, deeper analysis)
{ "question": "SQLite vs PostgreSQL vs DuckDB for analytics", "depth": "comprehensive" }

// Quick factual check
{ "question": "What are the breaking changes in React 19?", "depth": "quick" }

// Domain-scoped research
{ "question": "Next.js App Router patterns", "depth": "standard", "include_domains": ["nextjs.org", "vercel.com"] }

// With structured output schema
{ "question": "Compare Prisma vs Drizzle vs TypeORM", "depth": "standard", "schema": { "type": "object", "properties": { "orm": { "type": "string" }, "bundle_size": { "type": "string" }, "type_safety": { "type": "string" } } } }
```

## Depth Levels

| Depth | Sub-queries | Sources | Time | Use case |
|-------|-------------|---------|------|----------|
| `quick` | 2-3 | 5-8 | ~15s | Quick factual check |
| `standard` | 4-5 | 10-15 | ~40s | Normal research (default) |
| `comprehensive` | 6-7 | 20-25 | ~80s | Deep comparison, full review |

Override the source count for the chosen depth with `max_sources` (cap 50) — raise it to widen coverage, lower it to keep a run fast.

## Output: Structured Brief

When MCP sampling is unavailable (common case), the output carries a `brief`:

```json
{
  "brief": {
    "key_findings": [...],       // top passages across all sources — start report here
    "topics": [...],             // sources grouped by sub-query
    "sections": {
      "overview": { "cross_references": [...] },  // findings corroborated by 2+ sources — most reliable
      "comparison": {...},                         // entity-specific points for X vs Y queries
      "gaps": [...]                                // sub-queries / named entities with limited coverage
    }
  },
  "sub_queries": [...],
  "sources": [...],
  "query_type": "..."          // decomposition strategy used
}
```

**Gaps surface named sub-entities that decomposition or search could not corroborate** — never silently dropped.

## Writing the Report

See [wigolo/rules/synthesis.md](../wigolo/rules/synthesis.md). Quick version:

1. Start with `key_findings` for the executive summary.
2. Use `sections.overview.cross_references` for the most reliable claims.
3. Write sections from `topics`.
4. Build comparison table from `sections.comparison` (if present).
5. Note `sections.gaps` as limitations.
6. Cite with [N] format from `citations`.

## Anti-Patterns

- DON'T use for single-URL lookups — use `fetch` instead.
- DON'T use for data gathering — use `agent` with a schema instead.
- DON'T pre-probe cache before calling research — it checks internally.

## When NOT to use wigolo-research

- **You want a single-shot search result, not a synthesized report** — use `search` with `format: "answer"`.

## See Also

- [wigolo-search](../wigolo-search/SKILL.md) — for single-query search
- [wigolo-agent](../wigolo-agent/SKILL.md) — for structured data gathering (not reports)
- [wigolo/rules/synthesis.md](../wigolo/rules/synthesis.md) — how to write from briefs
