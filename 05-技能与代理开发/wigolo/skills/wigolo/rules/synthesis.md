---
name: wigolo-synthesis
description: How to synthesize answers and reports from wigolo's structured output formats.
---

# Synthesis Patterns

Wigolo returns structured evidence — you (the host LLM) write the final answer. The synthesis fallback ladder is: host sampling → optional local language model (opt in with `WIGOLO_LOCAL_LLM=auto`, off by default) → deterministic evidence. When neither sampling nor a local language model is available, wigolo hands you the structured evidence and you synthesize.

## From search evidence (default shape)

`search` returns scored `evidence[]` (title/url/section_heading/excerpt/score/citation_id/source_span) plus `citations[]`.

1. Read the excerpts — already ranked by ML rerank.
2. Group overlapping themes across sources.
3. Write your answer citing `[N]` or `{citation_id}`.
4. The `citations` array maps indices to URLs.

```json
search({ "query": "react server components patterns" })
// Returns: { results: [...], evidence: [{title, url, excerpt, score, citation_id, source_span}], citations: [{index, url, title}] }
// → Write answer citing [1], [2], etc.
```

## From sampling synthesis (`format: "answer"` / `"stream_answer"`)

When the MCP client supports sampling, wigolo returns a pre-synthesized `answer` (or streams it). Falls back to evidence shape otherwise.

```json
search({ "query": "react server components patterns", "format": "answer" })
// Sampling-supported: { answer: "...", citations: [...] }
// No sampling:        { evidence: [...], citations: [...] }  // synthesize yourself
```

## From research briefs (`research` tool)

When MCP sampling is unavailable (common), the output carries a `brief`:

| Field | Use |
|-------|-----|
| `key_findings` | Top passages across all sources — start executive summary here |
| `topics` | Sources grouped by sub-query — write per-topic sections |
| `sections.overview.cross_references` | Findings corroborated by 2+ sources — most reliable, cite first |
| `sections.comparison` | Entity-specific points (for X vs Y queries) — build comparison table |
| `sections.gaps` | Sub-queries / named sub-entities with limited coverage — note as limitations |

Report structure:
1. Executive summary from `key_findings`
2. Cross-referenced findings (cite as "corroborated by N sources")
3. Per-topic sections from `topics`
4. Comparison table from `sections.comparison` (if present)
5. Limitations from `sections.gaps`
6. Sources with [N] citation format
