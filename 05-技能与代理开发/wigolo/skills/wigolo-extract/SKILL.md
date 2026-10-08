---
name: wigolo-extract
description: |
  Local-first structured extraction from any webpage — tables, definition lists, key-value pairs, JSON-LD, microdata, chart hints (SVG titles / aria-labels / figcaptions), brand assets, and metadata. Use when the user wants structured data, pricing tables, feature comparisons, or says "extract the table", "get structured data", "pull the pricing", "extract as JSON". For autonomous navigation across many pages, use wigolo's `agent` tool instead.
license: AGPL-3.0-only
metadata:
  author: KnockOutEZ
  version: 0.1.43-beta.2
  homepage: https://github.com/KnockOutEZ/wigolo
  repository: https://github.com/KnockOutEZ/wigolo
---

# wigolo extract

Structured data extraction beyond simple markdown.

## Quick Reference

```json
// Full structured extraction (ALWAYS prefer this)
{ "url": "https://bun.sh", "mode": "structured" }

// JSON Schema extraction — heuristic field matching
{ "url": "https://example.com/pricing", "mode": "schema", "schema": { "type": "object", "properties": { "name": { "type": "string" }, "price": { "type": "string" }, "sku": { "type": "string" } } } }

// CSS selector extraction
{ "url": "https://example.com", "mode": "selector", "css_selector": ".product-card", "multiple": true }

// Metadata only (matches fetch metadata shape)
{ "url": "https://example.com", "mode": "metadata" }

// From raw HTML
{ "html": "<table>...</table>", "mode": "tables" }
```

## Modes

| Mode | What it extracts | When to use |
|------|-----------------|-------------|
| `structured` | Tables + definition lists + JSON-LD + chart hints + key-value pairs | **Default choice — use this** |
| `tables` | HTML tables only | When you specifically need only tables |
| `schema` | Fields matching a JSON Schema (LLM-sourced fields verified against source; hallucinated values returned as null) | When you know the exact fields you want |
| `brand` | Logo, favicon, colors, fonts, social_links (with provenance; favicons never promote to logo_url) | Brand kit / identity extraction |
| `metadata` | OpenGraph, meta tags, JSON-LD, canonical_url, og_image | For page metadata only |
| `selector` | CSS selector matches | When you know the exact CSS selector |

**Always use `mode: "structured"` instead of `mode: "tables"`.** Structured captures everything tables does, plus definitions, key-value pairs, JSON-LD, and chart descriptions.

## Chart Hints

When a page has visual charts (SVG, Canvas), `chart_hints` contains text descriptions extracted from aria-labels, SVG `<title>`, and figcaptions. Use these to describe visual data even when the underlying data is JavaScript-rendered.

## Schema Mode

`mode: "schema"` matches your JSON Schema field names against page content via CSS classes, ARIA labels, microdata, and JSON-LD. When a language model is available it sources fields and **verifies each value against the source — hallucinated values are returned as `null`** rather than guessed. Pass `{ properties: { field: { type: "string" } } }`.

## Named Schemas

Instead of hand-writing a schema, pass `named_schema` for a strict, heuristic-only extraction (no LLM required) into a known shape:

```json
{ "url": "https://blog.example.com/post", "named_schema": "Article" }
```

Available: `Article`, `Recipe`, `Product`, `CodeSnippet`, `Paper`, `EventListing`. Mutually exclusive with `schema`.

## Brand Mode

`mode: "brand"` pulls a page's identity assets — `logo`, `favicon`, `colors`, `fonts`, and `social_links` — each with provenance. Favicons never get promoted to `logo_url`.

## Token Budget

`max_tokens_out` (cl100k-base) caps extracted output; trailing table rows or heavy keys are dropped first to fit.

## Anti-Patterns

- DON'T use `mode: "tables"` — use `mode: "structured"` instead.
- DON'T pass a schema without `properties` key — handler rejects it.
- DON'T extract for a whole page when you need markdown — use `fetch` instead.

## When NOT to use wigolo-extract

- **Multi-page autonomous structured extraction** — use wigolo's `agent` tool instead.
- **Page requires login / click / form-fill before the data appears** — handle authentication with `use_auth` or interact with the page before extracting.

## See Also

- [wigolo-fetch](../wigolo-fetch/SKILL.md) — for markdown content
- [wigolo-agent](../wigolo-agent/SKILL.md) — for multi-page schema-driven gathering
