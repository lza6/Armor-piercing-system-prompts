# Configuration

wigolo works with zero configuration. Everything below is optional tuning: search backends, LLM providers for synthesis, fetch behavior, cache lifetimes, and daemon policy.

## Resolution order

Every setting resolves the same way:

```text
environment variable  >  ~/.wigolo/config.json  >  built-in default
```

Env vars win per-field, so you can persist a baseline in `config.json` and override one knob per process. `WIGOLO_CONFIG_PATH` relocates the config file itself.

## The config command

```bash
wigolo config              # interactive settings shell (TUI)
wigolo config --plain      # print current settings and exit
wigolo config --plain --json
wigolo config --set searchBackend=hybrid   # headless single-setting update
wigolo config --storage    # storage usage map
wigolo config --cache-stats
wigolo config --export settings.json       # secrets excluded
wigolo config --import settings.json
wigolo config --cleanup cache              # cache|embeddings|models|browser|searxng
```

`wigolo dashboard` is an alias of `wigolo config`. Secrets (LLM keys, proxy credentials) never go into `config.json` — they live in the OS keychain (see [privacy & security](./privacy-security.md#credentials)).

## Paths and data

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_DATA_DIR` | `~/.wigolo` | Root for the cache database, models, keys, plugins, shell history. |
| `WIGOLO_CONFIG_PATH` | `~/.wigolo/config.json` | Location of the persisted config file. |
| `WIGOLO_PLUGINS_DIR` | `~/.wigolo/plugins` | Where [plugins](./plugins.md) are loaded from. |

## Search backend

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_SEARCH` | `core` | Backend selector: `core`, `searxng`, or `hybrid`. |
| `BRAVE_API_KEY` | unset | Enables the optional Brave engines (general + images). |
| `WIGOLO_GITHUB_TOKEN` | unset | Authenticates the GitHub code-search engine (higher rate limit, private-org hydration). |
| `WIGOLO_RSS_FEEDS` | unset | Comma-separated feed URLs mixed into news-category results. |
| `WIGOLO_MULTI_QUERY_MAX` | `10` | Cap on query variants accepted in one array-query search. |
| `WIGOLO_MULTI_QUERY_CONCURRENCY` | `5` | How many variants dispatch in parallel. |

What the backends mean:

- **`core`** (default) — direct search-engine adapters queried in parallel, fused with reciprocal-rank fusion, then reranked by the on-device ML reranker. No sidecar process, no keys.
- **`searxng`** — the optional/legacy aggregator sidecar only. Higher long-tail recall on some queries, slower cold start; wigolo manages the sidecar process for you.
- **`hybrid`** — runs `core` first and falls back to the aggregator (merging results) only when a quality signal fires; the merged response names the signal in `fallback_signal`.

The legacy sidecar has its own knobs when you opt in: `SEARXNG_URL` (use an external instance), `SEARXNG_MODE` (`native` or `docker`), `SEARXNG_PORT` (default `8888`).

## Fetch and browser engine

| Env var | Default | What it does |
| --- | --- | --- |
| `FETCH_TIMEOUT_MS` | `10000` | Per-request HTTP timeout. |
| `FETCH_MAX_RETRIES` | `2` | HTTP retry budget. |
| `MAX_REDIRECTS` | `5` | Redirect ceiling. |
| `MAX_BROWSERS` | `3` | Browser-engine pool size for JS-rendered pages. |
| `WIGOLO_BROWSER_TYPES` | `chromium` | Comma list of browser families to pool. |
| `BROWSER_IDLE_TIMEOUT` | `60000` | Idle ms before a pooled browser is released. |
| `USER_AGENT` | unset | Override the outgoing user agent. |
| `RESPECT_ROBOTS_TXT` | `true` | Honor robots.txt on crawls (on by default; be a good citizen). |
| `CRAWL_CONCURRENCY` | `2` | Parallel fetches per crawl. |
| `CRAWL_DELAY_MS` | `500` | Politeness delay between same-site requests. |
| `WIGOLO_FETCH_ALLOW_PRIVATE` | `false` | Allow fetching private/loopback address targets (SSRF guard override for local dev servers). |
| `USE_PROXY` / `PROXY_URL` | off | Route fetches through an HTTP(S) proxy. Credentials in the URL are moved to the OS keychain; only the credential-free URL is persisted. |
| `WIGOLO_TLS_TIER` | `off` | TLS-impersonation fetch tier: `off`, `auto` (only on an anti-bot signal), `on` (try first for cold domains). Improves reliability on sites that reject generic HTTP clients. |
| `WIGOLO_STEALTH` | `auto` | Browser-tier fingerprint hardening: `off`, `auto` (only on challenge escalations), `on` (every browser fetch). |
| `WIGOLO_TLS_BROWSER` | `chrome_142` | Browser profile the TLS tier presents. Allowlisted to `chrome\|firefox\|safari\|edge\|opera` + version; invalid values fall back safely. |
| `WIGOLO_TLS_SUCCESS_THRESHOLD` | `3` | Successes before a domain is auto-promoted to TLS-first routing. |
| `WIGOLO_TLS_DOMAINS` | unset | Comma list of extra domains that should try the TLS tier first. |
| `WIGOLO_CHALLENGE_COMPLETION_MS` | `15000` | How long the browser tier polls a challenge page before fast-failing with a labeled `blocked_by_challenge` result. |

Reliability framing matters here: these tiers make wigolo read pages the way a real browser does, and it learns per-domain which tier works (inspect with [`wigolo tune`](./cli.md#tune)). When a challenge doesn't clear, the failure is labeled honestly instead of returning junk. Crawling stays polite throughout — robots.txt respected by default, per-domain rate limits, and page budgets sized for research, not bulk harvesting.

For fetching pages behind a login with your own browser session: `WIGOLO_CDP_URL` (attach to a running browser's debug port), `WIGOLO_CHROME_PROFILE_PATH` (use a profile), `WIGOLO_AUTH_STATE_PATH` (a saved storage-state file). Then pass `use_auth: true` on `fetch`/`crawl`. `wigolo auth discover` lists attachable sessions.

## On-device models

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_RERANKER` | `onnx` | Result reranking: `onnx` (the bundled on-device ranking model), `none`, or `custom`. |
| `WIGOLO_RERANKER_MODEL` | `bge-reranker-v2-m3` | Which ranking model to load. |
| `WIGOLO_EMBEDDING_MODEL` | `BAAI/bge-small-en-v1.5` | Embedding model for the semantic cache index and `find_similar`. |
| `WIGOLO_RELEVANCE_THRESHOLD` | `0` | Drop search results below this reranker score (0 = keep all). |

Models download once (during `init`/`warmup` or lazily on first use) and run fully in-process — no external services.

## LLM providers (optional)

Core tools never need an LLM. Configuring one adds answer synthesis (`format: "answer"` on search), essay-grade research briefs, and a structured-extraction fallback.

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_LLM_PROVIDER` | unset | `anthropic`, `openai`, `gemini`, `groq`, or `ollama` (any local OpenAI-compatible server). |
| `ANTHROPIC_API_KEY` / `OPENAI_API_KEY` / `GEMINI_API_KEY` / `GROQ_API_KEY` | unset | Per-provider keys, read from env. |
| `WIGOLO_LLM_API_KEY` | unset | Generic key slot (used by `init --provider=...`; stored in the OS keychain, never passed as a flag). |
| `WIGOLO_LLM_MODEL` | provider default | Override the model name. |
| `WIGOLO_LLM_BASE_URL` | `http://localhost:11434` | Custom base URL for the `ollama` provider — point it at any OpenAI-compatible endpoint. |
| `WIGOLO_LLM_CACHE_TTL_DAYS` | `7` | Cache lifetime for LLM outputs. |
| `WIGOLO_LLM_MAX_CALLS_PER_REQUEST` | `1` | Hard cap on LLM calls per tool request. |

### Keyless local ladder

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_LOCAL_LLM` | `off` | `off` (never probe), `auto` (probe the default local endpoint and use it when reachable), or an explicit `http(s)://` endpoint to probe. |
| `WIGOLO_LOCAL_LLM_MODEL` | auto-pick | Preferred model name for the local tier. |

With `WIGOLO_LOCAL_LLM=auto` and a local model server running, you get synthesis with zero API keys and zero cloud round-trips.

## Cache lifetimes

| Env var | Default | What it does |
| --- | --- | --- |
| `CACHE_TTL_SEARCH` | `86400` (1 day) | Seconds a cached search result is served without re-querying. |
| `CACHE_TTL_CONTENT` | `604800` (7 days) | Seconds a cached page is served without refetching. |

Per-call `force_refresh: true` skips both and goes to the network. `wigolo cache clear` and `wigolo config --cleanup cache` manage the store; see [tools → cache](./tools.md#cache).

## Serve / daemon

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_DAEMON_PORT` | `3333` | Default port for `wigolo serve`. |
| `WIGOLO_DAEMON_HOST` | `127.0.0.1` | Default bind host. |
| `WIGOLO_API_TOKEN` | unset | Bearer token required on the REST + MCP surface. **Required** for non-loopback binds. |
| `WIGOLO_API_TOKEN_FILE` | unset | Read the token from a file (Docker/systemd secret pattern). |
| `WIGOLO_SERVE_ALLOW_UNAUTHENTICATED` | unset | `1` opts into open remote access (same as `--allow-unauthenticated`). |
| `WIGOLO_SERVE_ALLOW_LOCAL_TARGETS` | unset | `1` lets a remote-exposed daemon fetch loopback/localhost URLs (blocked by default). |
| `WIGOLO_SERVE_MAX_BODY_BYTES` | `1 MiB` (5 MiB for diff/extract) | Request-body cap. |
| `WIGOLO_SERVE_MAX_CONCURRENCY` | `16` | In-flight request cap on `/v1`. |
| `WIGOLO_SERVE_TIMEOUT_SCALE` | `1` | Multiplier on per-route response deadlines. |
| `WIGOLO_SERVE_REQUEST_TIMEOUT_MS` | `120000` | Whole-request timeout (slow-client guard). |
| `WIGOLO_SERVE_HEADERS_TIMEOUT_MS` | `60000` | Header-receipt timeout. |

Full endpoint + auth semantics — including the opt-in [compat shim](./rest-api.md#compat-shim) — in [REST API](./rest-api.md) and [self-hosting](./self-hosting.md).

## Telemetry

Off by default, and stays off unless you opt in.

| Env var | Default | What it does |
| --- | --- | --- |
| `WIGOLO_TELEMETRY` | unset | `1` enables event logging to a local file: `~/.wigolo/telemetry/events-YYYYMMDD.ndjson`. Nothing is transmitted. |
| `WIGOLO_TELEMETRY_ENDPOINT` | unset | Only if you set this does wigolo additionally POST events to that URL (yours). |

## Logging

| Env var | Default | What it does |
| --- | --- | --- |
| `LOG_LEVEL` | `info` | `debug`, `info`, `warn`, `error`. |
| `LOG_FORMAT` | `json` | `json` or `text`. |

All logs go to **stderr** — stdout is reserved for MCP protocol traffic and `--json` tool output, so logs never corrupt either.

[← Docs index](./README.md) · [Next: Tools](./tools.md)
