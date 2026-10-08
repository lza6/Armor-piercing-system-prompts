---
name: wigolo
description: Local-first web intelligence MCP server for AI coding agents. Ten tools for search, fetch, crawl, cache, extract, find similar, research, agent-driven data gathering, page diffing, and change watching. No API keys. Results cached in a local knowledge store.
author: KnockOutEZ
license: AGPL-3.0-only
repository: https://github.com/KnockOutEZ/wigolo
transport: stdio
install: npx wigolo
runtime: node
min_runtime_version: "20"
tools:
  - name: fetch
    description: Fetch one URL, return clean markdown. Auto-routes between HTTP and browser engine. Supports sections, auth, screenshots, browser actions.
  - name: search
    description: Search the web, return extracted markdown per result. Single query or array of query variants. Domain, category, date filters. Formats include ML-scored highlights with citations for host-LLM synthesis.
  - name: crawl
    description: Crawl a site from a seed URL. BFS, DFS, sitemap, or map (URL-only) strategies with regex include/exclude filters.
  - name: cache
    description: Full-text search over previously fetched content. URL glob, date filters, stats, clear, and change detection via re-fetch.
  - name: extract
    description: Structured extraction from URL or raw HTML. Modes: selector (CSS), tables, metadata (meta + JSON-LD), schema (heuristic field matching), structured (tables + dl + JSON-LD + chart hints + key-value pairs in one call).
  - name: find_similar
    description: Find pages similar to a URL or concept. Hybrid cache (keyword search + embeddings) + optional web supplement.
  - name: research
    description: Multi-step research pipeline. Question decomposition, parallel sub-search, source synthesis with citations. Quick, standard, or comprehensive depth.
  - name: agent
    description: Natural-language data gathering. Plans searches/URLs, fetches in parallel within page and time budgets, optionally applies a JSON Schema to each page.
  - name: diff
    description: Compare two page versions — a live URL vs its cached copy, two URLs, or two markdown blobs. Unified patch, per-section hunks, or a counts summary at line, word, or section granularity.
  - name: watch
    description: Monitor a page for changes over time. Create lazy watch jobs on one or many URLs, list them, and check on demand with optional SSRF-guarded webhook delivery.
---

# wigolo

Local-first web intelligence MCP server for AI coding agents. Ships ten tools over stdio. All network results land in a local knowledge cache.

## Host-LLM synthesis (read me first)

Wigolo has no internal LLM. It returns *structured evidence* so the calling model (you) writes the final answer. Fold structure into your reply rather than collapsing it away:

- `search` with `format: "highlights"` — ML-scored passages + `citations`. Quote and cite [N].
- `research` — when MCP sampling is unavailable (common), the output carries a `brief` with `topics`, `highlights`, `key_findings`. Use it as the scaffold for the report you write.
- `find_similar` — may return a `cold_start` string. Pass it to the user; it explains why results came from the web and how to warm the cache.
- `extract` with `mode: "structured"` — one call for tables + `<dl>` definitions + JSON-LD + chart hints + key-value pairs.
- `fetch` metadata — surfaces `og_type`, `canonical_url`, and `og_image`; use `canonical_url` to dedupe tracked/canonical URLs.

## Quick Setup

**Claude Code:**
```bash
claude mcp add wigolo -- npx wigolo
```

**Cursor / VS Code / any MCP client:**
```json
{
  "mcpServers": {
    "wigolo": {
      "command": "npx",
      "args": ["wigolo"]
    }
  }
}
```

**Warmup (recommended, one-time):**
```bash
npx wigolo warmup          # installs browser engine + bootstraps search engine
npx wigolo warmup --all    # also installs Firefox, WebKit, ML reranker, and embeddings
npx wigolo warmup --force  # wipe search engine state and rebuild
```

Warmup flags: `--force`, `--all`, `--reranker`, `--firefox`, `--webkit`, `--embeddings`, `--no-searxng`, `--verify`.

## Tools

### fetch

Fetch a single URL and return clean markdown. Use when you already have a specific URL.

Parameters:
- `url` (string, required)
- `render_js`: `"auto"` (default) | `"always"` | `"never"`
- `use_auth`: boolean (default `false`) — reuses the user's browser session
- `max_chars`: number
- `section`: string — return only the content under a heading
- `section_index`: number (default `0`) — which heading match when multiple hit
- `screenshot`: boolean (default `false`)
- `headers`: object
- `force_refresh`: boolean — bypass cache
- `actions`: array of `{type, selector, text, ms, timeout, direction, amount}` — `click`, `type`, `wait`, `wait_for`, `scroll`, `screenshot`. Forces browser rendering when present.

Example:
```json
{ "url": "https://react.dev/reference/react/useState", "section": "Parameters" }
```

Tip: `section` is much cheaper than reading the full page. Repeat fetches of the same URL are free from cache unless `force_refresh: true`.

### search

Search the web and return extracted markdown per result. Use when you don't have a URL yet.

Parameters:
- `query` (string OR `string[]`, required) — array runs variants in parallel and dedupes
- `max_results`: number (default `5`, cap `20`)
- `include_content`: boolean (default `true`)
- `content_max_chars`: number (default `30000`)
- `max_total_chars`: number (default `50000`)
- `time_range`: `"day"` | `"week"` | `"month"` | `"year"`
- `include_domains` / `exclude_domains`: `string[]`
- `from_date` / `to_date`: ISO `YYYY-MM-DD`
- `category`: `"general"` | `"news"` | `"code"` | `"docs"` | `"papers"` | `"images"`
- `language`: string
- `search_engines`: `string[]` — override engine selection
- `format`: `"full"` (default) | `"context"` (token-budgeted string) | `"highlights"` (ML-scored passages + citations) | `"answer"` (synthesized via MCP sampling; falls back to `highlights` when unsupported) | `"stream_answer"` (answer + phase progress notifications)
- `max_highlights`: number (default `10`) — cap when `format: "highlights"`
- `force_refresh`: boolean

Example:
```json
{ "query": ["react server components patterns", "RSC data fetching", "react server components streaming"], "category": "docs", "include_domains": ["react.dev"], "max_results": 5 }
```

Tip: keyword queries beat natural-language questions. A 3–5 item `query` array usually finds more unique sources than one longer query.

### crawl

Crawl a site starting from a seed URL.

Parameters:
- `url` (string, required)
- `strategy`: `"bfs"` (default) | `"dfs"` | `"sitemap"` | `"map"` (URL-only discovery, no content)
- `max_depth`: number (default `2`)
- `max_pages`: number (default `20`)
- `include_patterns` / `exclude_patterns`: regex `string[]`
- `use_auth`: boolean (default `false`)
- `extract_links`: boolean (default `false`) — returns inter-page link graph
- `max_total_chars`: number (default `100000`)

Example:
```json
{ "url": "https://docs.python.org/3/library/", "strategy": "sitemap", "max_pages": 30, "include_patterns": ["^https://docs\\.python\\.org/3/library/asyncio"] }
```

Tip: `strategy: "sitemap"` is faster and more complete than BFS on doc sites. `strategy: "map"` returns URLs only — cheap way to scope before targeted fetches.

### cache

Search previously fetched content without hitting the network.

Parameters:
- `query`: full-text search — supports `AND`, `OR`, `NOT`, `"exact phrase"`
- `url_pattern`: glob (e.g. `"*react.dev*"`)
- `since`: ISO date
- `stats`: boolean — returns total URLs, size, date range
- `clear`: boolean — deletes matching entries (requires one of `query`, `url_pattern`, `since`)
- `check_changes`: boolean — re-fetches matching URLs, reports changed/unchanged with diff summaries

Example:
```json
{ "query": "useState OR useReducer", "url_pattern": "*react.dev*" }
```

Tip: cache hits are instant and cross-session. Run this before `search` or `fetch` when you suspect the content is already on disk.

### extract

Structured extraction from URL or raw HTML.

Parameters:
- `url` OR `html` (one required; `url` wins if both provided)
- `mode`: `"metadata"` (default) | `"selector"` | `"tables"` | `"schema"` | `"structured"` (tables + `<dl>` definitions + JSON-LD + chart hints + microdata/data-attr/grid key-value pairs in one call)
- `css_selector`: string — required for `mode: "selector"`
- `multiple`: boolean (default `false`) — return all matches, selector mode only
- `schema`: JSON Schema object with `properties` — required for `mode: "schema"`

Example:
```json
{ "url": "https://example.com/product", "mode": "schema", "schema": { "type": "object", "properties": { "price": { "type": "string" }, "name": { "type": "string" }, "sku": { "type": "string" } } } }
```

Tip: `mode: "schema"` does heuristic matching over CSS classes, ARIA labels, microdata, and JSON-LD — no LLM call required. `mode: "structured"` returns every structured pattern on the page (`tables`, `definitions`, `jsonld`, `chart_hints`, `key_value_pairs`) in one response — prefer it over chaining multiple extract calls.

### find_similar

Find pages related to a URL or a free-text concept.

Parameters:
- `url` OR `concept` (one required)
- `max_results`: number (default `10`, cap `50`)
- `include_domains` / `exclude_domains`: `string[]`
- `include_cache`: boolean (default `true`)
- `include_web`: boolean (default `true`)

Example:
```json
{ "url": "https://react.dev/reference/react/useState", "max_results": 8, "include_domains": ["react.dev", "developer.mozilla.org"] }
```

Tip: uses hybrid 3-way RRF fusion — keyword search + semantic embeddings + live web search. Each result carries `match_signals` with `embedding_rank`, `fts5_rank`, and `fused_score`. If the cache is empty or embeddings aren't set up, the response includes a `cold_start` string — pass it to the user to explain why results came from the web.

### research

Multi-step research pipeline with decomposition, parallel search, and cited synthesis.

Parameters:
- `question` (string, required)
- `depth`: `"quick"` (~15s, 2 sub-queries, 5–8 sources) | `"standard"` (~40s, default) | `"comprehensive"` (~80s, 7 sub-queries, 20–25 sources)
- `max_sources`: number (cap `50`) — overrides depth default
- `include_domains` / `exclude_domains`: `string[]`
- `schema`: JSON Schema — if present, report is structured to fill these fields
- `stream`: boolean — emit progress notifications per phase

Example:
```json
{ "question": "How do modern JS bundlers tree-shake ESM vs CJS?", "depth": "standard", "include_domains": ["webpack.js.org", "rollupjs.org", "esbuild.github.io", "vitejs.dev"] }
```

Tip: `research` checks cache internally — no need to pre-probe. With MCP sampling, the tool synthesizes the report directly. Without sampling (the common case), the output ships a `brief` with `topics`, `highlights`, and `key_findings`, plus the raw sources — the host LLM writes the final report from the brief.

### agent

Natural-language data gathering. Plans queries and URLs from a prompt, runs them in parallel within budget, optionally applies a schema.

Parameters:
- `prompt` (string, required)
- `urls`: `string[]` — seed URLs to include
- `schema`: JSON Schema — extract structured fields per page and merge
- `max_pages`: number (default `10`, cap `100`)
- `max_time_ms`: number (default `60000`, cap `600000`)
- `stream`: boolean

Example:
```json
{ "prompt": "Compare pricing tiers for Supabase, Firebase, and Clerk", "schema": { "type": "object", "properties": { "provider": { "type": "string" }, "free_tier": { "type": "string" }, "paid_start": { "type": "string" } } }, "max_pages": 12 }
```

Tip: output includes a `steps` array showing every action (plan, search, fetch, extract, synthesize) with timings. Use this to debug why an agent run produced a weak result.

### diff

Compare two versions of a page. Point it at a live URL and its cached copy, two URLs, or two markdown blobs.

Parameters:
- `old` (object, required): one of `{ url, markdown, content_hash }`
- `new` (object, required): one of `{ url, markdown }`
- `output`: `"unified"` (default), `"hunks"`, or `"summary"`
- `granularity`: `"line"` (default), `"word"`, or `"section"`

Example:
```json
{ "old": { "url": "https://docs.example.com/api" }, "new": { "url": "https://docs.example.com/api" }, "output": "hunks", "granularity": "section" }
```

Tip: same URL on both sides diffs the cached copy against a fresh fetch. Use `output: "summary"` for counts only.

### watch

Monitor a page for changes over time. Lazy execution — checks run when you call `check` or when an overdue job is picked up during another tool run.

Parameters:
- `action` (string, required): `"create"`, `"list"`, `"check"`, `"pause"`, `"resume"`, `"delete"`
- `url` or `urls`: single-URL or batch create (mutually exclusive)
- `interval_seconds`: required for create (min 60)
- `selector`: create-only CSS selector to scope the diff
- `notification`: `"inline"` (default) or an SSRF-guarded webhook URL
- `job_id`: required for check/pause/resume/delete

Example:
```json
{ "action": "create", "url": "https://nodejs.org/en/blog", "interval_seconds": 21600 }
```

Tip: webhook destinations are SSRF-guarded — a job cannot be pointed at internal or loopback addresses.

## Workflow Patterns

Quick routing:
- Use when `search` — you need information but don't have a URL.
- Use when `fetch` — you already have the URL.
- Use when `crawl` — you need multiple pages from one site.
- Use when `cache` — you want to check whether something is already on disk.
- Use when `extract` — you need specific fields, tables, or metadata, not the whole page.
- Use when `find_similar` — you have a good page/concept and want related content.
- Use when `research` — a question needs decomposition and multi-source synthesis.
- Use when `agent` — a natural-language task needs multi-step data gathering.
- Use when `diff` — you need to see what changed between two versions of a page.
- Use when `watch` — you want to monitor a page for changes over time.

**Cache-first lookup.** Before any `fetch` or `search`, probe the cache.
```json
cache({ "query": "oauth2 pkce", "url_pattern": "*auth0.com*" })
// empty? fall through to search
search({ "query": "oauth2 pkce flow", "include_domains": ["auth0.com"] })
```

**Fresh content (news, dashboards, changelogs).** Bypass cache explicitly.
```json
search({ "query": "node.js 22 release notes", "force_refresh": true, "time_range": "week" })
fetch({ "url": "https://nodejs.org/en/blog", "force_refresh": true })
```

**Scoped documentation research.** Crawl the relevant slice, then query cache.
```json
crawl({ "url": "https://docs.astro.build", "strategy": "sitemap", "max_pages": 40 })
cache({ "query": "server islands hydration", "url_pattern": "*docs.astro.build*" })
```

**Broad exploration.** Pass a query array; dedup is automatic.
```json
search({ "query": ["rust async runtimes comparison", "tokio vs async-std vs smol", "rust executor benchmarks"], "max_results": 8 })
```

**More like this.** Start with a known-good URL, widen via `find_similar`.
```json
find_similar({ "url": "https://react.dev/reference/react/useMemo", "max_results": 6, "include_domains": ["react.dev"] })
```

**Complex synthesis.** One `research` call replaces 5+ manual search/fetch cycles.
```json
research({ "question": "Tradeoffs of vector DBs for RAG at 100M+ embeddings", "depth": "comprehensive" })
```

**Structured data from multiple sources.** Use `agent` with a schema.
```json
agent({ "prompt": "Find latency and pricing for top 5 edge compute providers", "schema": { "type": "object", "properties": { "provider": {"type":"string"}, "cold_start_ms": {"type":"string"}, "price_per_million": {"type":"string"} } } })
```

**Table extraction.** Skip markdown entirely.
```json
extract({ "url": "https://en.wikipedia.org/wiki/List_of_programming_languages", "mode": "tables" })
```

**One-shot structured brief.** Tables + definition lists + JSON-LD + chart hints + key-value pairs in one call.
```json
extract({ "url": "https://example.com/product-page", "mode": "structured" })
```

**Direct quotes with citations.** ML-scored passages are ideal for host-LLM synthesis.
```json
search({ "query": "react server components data fetching", "format": "highlights", "max_highlights": 6, "include_domains": ["react.dev", "nextjs.org"] })
```

## Parameter Cheat Sheet

| Situation | Tool + parameters |
|---|---|
| Focused lookup, known site | `search` + `max_results: 3` + `include_domains` |
| Broad topic survey | `search` + `query: [...3-5 variants]` + `max_results: 8` |
| Fresh content required | any tool + `force_refresh: true` |
| Doc site indexing | `crawl` + `strategy: "sitemap"` |
| Site URL inventory only | `crawl` + `strategy: "map"` |
| Single heading from long page | `fetch` + `section: "..."` |
| Behind login | `fetch` / `crawl` + `use_auth: true` |
| Direct answer (sampling client) | `search` + `format: "answer"` |
| ML-scored passages + citations | `search` + `format: "highlights"` |
| LLM-ready context blob | `search` + `format: "context"` |
| Complex question, multi-source | `research` + `depth: "standard"` |
| Structured multi-page extraction | `agent` + `schema` |
| One-page structured data | `extract` + `mode: "structured"` (everything) or `"schema"` / `"tables"` (targeted) |
| Change tracking | `cache` + `check_changes: true` |

## Anti-Patterns

**Do not skip the cache.** Running `search` or `fetch` without probing `cache` wastes time on content already on disk. `research` and `agent` check cache internally; manual `search`/`fetch` do not.

**Do not send natural-language questions to `search`.** Use keywords. `"how do I debounce in React hooks"` loses to `"react useDebounce hook custom"`.

**Do not retry an identical failing query.** Reformulate keywords, swap `category`, or add `include_domains`. Same query → same empty result.

**Do not use `agent` or `research` for one-URL lookups.** Use `fetch`. `agent` is for multi-source gathering; `research` is for decomposable questions.

**Do not crawl `max_pages: 100` without filters.** Always add `include_patterns` to stay in-scope. Unfiltered crawls fetch nav, footer, and sitemap garbage.

**Do not fetch whole pages when you need one section.** `fetch` + `section` reads under one heading only.

**Do not set `force_refresh: true` by default.** It defeats the cache. Use it for news, status, changelogs — content that actually churns.

**Do not pass a JSON Schema to `extract` without `properties`.** The handler rejects schemas that lack a `properties` key.

## CLI Commands

```bash
wigolo                  # default: start MCP server on stdio
wigolo mcp              # explicit: start MCP server
wigolo warmup [flags]   # install browser engine, bootstrap search engine, optional extras
wigolo serve            # start HTTP daemon on WIGOLO_DAEMON_PORT (default 3333)
wigolo health           # health probe, exits 0 if ok
wigolo doctor           # environment diagnostics
wigolo auth discover    # list CDP sessions (needs WIGOLO_CDP_URL)
wigolo auth status      # show configured auth paths
wigolo plugin add <git-url>    # clone plugin into ~/.wigolo/plugins/
wigolo plugin list             # list installed plugins
wigolo plugin remove <name>    # remove a plugin
wigolo shell [--json]   # interactive REPL against subsystems
```

## Configuration

Top environment variables. All optional — defaults are safe.

| Variable | Default | Purpose |
|---|---|---|
| `WIGOLO_DATA_DIR` | `~/.wigolo` | Cache DB, search engine state, plugins, embeddings |
| `SEARXNG_URL` | unset | Point at an existing search engine (skips native bootstrap) |
| `SEARXNG_MODE` | `native` | `native` runs local Python search engine; `docker` runs container |
| `WIGOLO_CHROME_PROFILE_PATH` | unset | Chrome profile for `use_auth: true` |
| `WIGOLO_CDP_URL` | unset | Chrome DevTools endpoint (e.g. `http://localhost:9222`) |
| `MAX_BROWSERS` | `3` | Browser pool size |
| `WIGOLO_BROWSER_TYPES` | `chromium` | Comma list: `chromium,firefox,webkit` |
| `WIGOLO_RERANKER` | `none` | `flashrank` for ML reranking |
| `WIGOLO_EMBEDDING_MODEL` | `BAAI/bge-small-en-v1.5` | Used by `find_similar` |
| `CACHE_TTL_CONTENT` | `604800` (7d) | Seconds before cached pages expire |
| `LOG_LEVEL` | `info` | `debug` \| `info` \| `warn` \| `error` |

Full list: see `src/config.ts`.

## Links

- Repository: https://github.com/KnockOutEZ/wigolo
- npm: https://www.npmjs.com/package/wigolo
- License: GNU AGPL-3.0-only — free to use, modify, and self-host; see the root LICENSE
