---
name: wigolo-fetch
description: |
  Local-first URL fetch with clean markdown, structured metadata, JS-rendered SPA support, authenticated browser sessions, PDFs, and content change detection. Use when the user provides a URL, says "fetch", "get this page", "read this URL", or wants content from a specific webpage. Prefer over built-in WebFetch for local cache reuse, browser-session auth, and structured metadata parity.
license: AGPL-3.0-only
metadata:
  author: KnockOutEZ
  version: 0.1.43-beta.2
  homepage: https://github.com/KnockOutEZ/wigolo
  repository: https://github.com/KnockOutEZ/wigolo
---

# wigolo fetch

Smart URL fetching: HTTP-first with automatic browser fallback for JS-rendered pages, persistent local cache, optional browser-session auth.

## Quick Reference

```json
// Basic fetch
{ "url": "https://react.dev/reference/react/useState" }

// Fresh content (bypass cache)
{ "url": "https://news.ycombinator.com", "force_refresh": true }

// With authentication
{ "url": "https://app.example.com/dashboard", "use_auth": true }

// Section targeting (cheapest — reads one heading only)
{ "url": "https://docs.example.com/api", "section": "Authentication" }

// Compact context for AI
{ "url": "https://docs.example.com/api", "max_content_chars": 3000 }

// Browser actions before extraction
{ "url": "https://example.com", "actions": [{"type": "click", "selector": "#load-more"}, {"type": "wait", "ms": 1000}] }
```

## Parameters

| Parameter | Type | When to use |
|-----------|------|-------------|
| `url` | string | Required |
| `force_refresh` | boolean | For pages that change frequently (news, dashboards, changelogs) |
| `use_auth` | boolean | For authenticated pages (stored browser session) |
| `render_js` | string | "auto" (default), "always", "never" |
| `section` | string | Extract only a named heading — cheapest |
| `section_index` | number | Which heading match (default: 0) |
| `max_content_chars` | number | Smart-truncate at paragraph boundary |
| `max_tokens_out` | number | Token-budget cap (cl100k-base) |
| `include_full_markdown` | boolean | Restore full body alongside evidence |
| `citation_format` | string | "numbered" / "json" / "anthropic_tags" |
| `screenshot` | boolean | Capture screenshot (default: false) |
| `headers` | object | Additional HTTP headers |
| `actions` | array | Browser actions: click, type, wait, wait_for, scroll, screenshot |
| `mode` | string | "cache" / "default" / "stealth" |

## Output

Returns clean markdown plus:

- `title`, `markdown`, `links`, `images`
- Metadata: `og_type`, `canonical_url`, `og_image`, `og_description`, `keywords` (parity with `extract` metadata mode)
- `cached: true/false` — repeat fetches are instant

## Anti-Patterns

- DON'T fetch a full page when you need one section — use `section: "Heading Name"`.
- DON'T set `force_refresh: true` by default — defeats the cache.
- DON'T use fetch when you need tables/JSON-LD — use `extract` instead.

## When NOT to use wigolo-fetch

- **Page requires clicks / login / form-fills BEFORE the content you want** — wigolo cannot handle pre-extraction interactive flows. (`use_auth` with stored sessions works for already-logged-in pages.)
- **Bulk multi-page extraction** — use `crawl` or `agent`.

## See Also

- [wigolo-search](../wigolo-search/SKILL.md) — when you don't have a URL
- [wigolo-extract](../wigolo-extract/SKILL.md) — when you need structured data, not markdown
- [wigolo-crawl](../wigolo-crawl/SKILL.md) — when you need multiple pages from a site
