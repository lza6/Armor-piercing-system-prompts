---
name: wigolo-diff
description: |
  Compare two versions of a page and see exactly what changed — a live URL against its cached copy, two URLs, or two markdown blobs. Section-level hunks, word- or line-level granularity, or a summary of counts. Use when the user says "what changed", "diff these", "compare this page to last time", "show me the changes", "did this doc update", or wants a changelog between two versions of the same content.
license: AGPL-3.0-only
metadata:
  author: KnockOutEZ
  version: 0.1.43-beta.2
  homepage: https://github.com/KnockOutEZ/wigolo
  repository: https://github.com/KnockOutEZ/wigolo
---

# wigolo diff

Structural comparison of two page versions. Point it at a live URL and its cached copy, two URLs, or two raw markdown blobs, and it returns a git-style patch, structured per-section hunks, or a counts-only summary.

## Quick Reference

```json
// Live page vs its cached version (re-fetches `new` side)
{ "old": { "url": "https://docs.example.com/api" }, "new": { "url": "https://docs.example.com/api" } }

// Two different URLs
{ "old": { "url": "https://v1.example.com/spec" }, "new": { "url": "https://v2.example.com/spec" } }

// Two markdown blobs directly
{ "old": { "markdown": "# Title\nold body" }, "new": { "markdown": "# Title\nnew body" } }

// Structured per-section hunks at word granularity
{ "old": { "url": "https://example.com" }, "new": { "url": "https://example.com" }, "output": "hunks", "granularity": "word" }

// Counts only
{ "old": { "url": "https://example.com" }, "new": { "url": "https://example.com" }, "output": "summary" }
```

## Parameters

| Parameter | Type | Default | When to use |
|-----------|------|---------|-------------|
| `old` | object | required | Left-hand side. One of `{ url, markdown, content_hash }` |
| `new` | object | required | Right-hand side. One of `{ url, markdown }` |
| `output` | string | "unified" | "unified" (git-style patch), "hunks" (per-section), "summary" (counts only) |
| `granularity` | string | "line" | "line", "word" (changed tokens only — tighter), "section" (walks H1/H2/H3) |

## Output Shapes

- **`unified`** — a git-style unified patch, ready to read or store.
- **`hunks`** — structured per-section change blocks. Combine with `granularity: "section"` to align hunks to headings, or `granularity: "word"` for tight intra-line edits.
- **`summary`** — counts only: `added_lines`, `removed_lines`, `modified_lines`, and `total_changed_chars` (sum of added + removed line chars across the edit script). Cheapest way to answer "did anything change, and how much".

## Comparing Against the Cache

The most common use is drift detection: fetch a page now and diff it against the copy wigolo cached earlier. Pass the same URL for both sides — wigolo reads the cached body for `old` and re-fetches for `new`. For a scoped, multi-URL sweep, prefer `cache` with `check_changes: true`.

## Anti-Patterns

- DON'T pass an `old`/`new` object without one of the accepted side keys — the handler rejects it.
- DON'T use `output: "unified"` when you only need counts — `summary` is cheaper.
- DON'T diff two unrelated pages expecting a meaningful changelog — diff versions of the same content.

## When NOT to use wigolo-diff

- **Bulk change detection across many cached URLs** — use `cache` with `check_changes: true`.
- **Ongoing scheduled monitoring** — use `watch`, which diffs on each check.

## See Also

- [wigolo-cache](../wigolo-cache/SKILL.md) — `check_changes` for bulk drift detection
- [wigolo-watch](../wigolo-watch/SKILL.md) — scheduled change monitoring
- [wigolo-fetch](../wigolo-fetch/SKILL.md) — pull a fresh version to compare
