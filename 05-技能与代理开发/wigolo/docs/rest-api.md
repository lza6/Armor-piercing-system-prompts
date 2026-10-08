# REST API

`wigolo serve` turns wigolo into an HTTP daemon: a REST endpoint per tool, a machine-readable OpenAPI contract, and remote MCP transports — one process serving CLI scripts, SDKs, and multiple agents at once.

```bash
wigolo serve [--port N] [--host H] [--allow-unauthenticated]
```

Defaults: `127.0.0.1:3333` (override with `WIGOLO_DAEMON_PORT` / `WIGOLO_DAEMON_HOST`). Startup names every surface:

```text
[wigolo serve] Daemon running at http://127.0.0.1:3333
[wigolo serve] Health check: curl http://127.0.0.1:3333/health
[wigolo serve] REST API: http://127.0.0.1:3333/v1  (OpenAPI: http://127.0.0.1:3333/openapi.json)
[wigolo serve] Auth: open on loopback only
[wigolo serve] MCP endpoint: http://127.0.0.1:3333/mcp (StreamableHTTP)
[wigolo serve] SSE endpoint: http://127.0.0.1:3333/sse
```

## Endpoints

| Endpoint | Method | What it is |
| --- | --- | --- |
| `/v1/{tool}` | POST | One route per tool: `search`, `fetch`, `crawl`, `cache`, `extract`, `find_similar`, `research`, `agent`, `diff`, `watch`. Body = the tool's JSON params ([tools reference](./tools.md)); response = the tool's JSON result. |
| `/v1/tools` | GET | Lists all tools with descriptions and endpoints. |
| `/health` | GET | Liveness + component status. **Always open** — no token required, safe for load balancers and container healthchecks. |
| `/openapi.json` | GET | The OpenAPI 3.1 contract (also at `/v1/openapi.json`). |
| `/mcp` | POST/GET/DELETE | Remote MCP over StreamableHTTP — point HTTP-capable MCP clients here. |
| `/sse` + `/messages` | GET / POST | Legacy MCP-over-SSE transport. |
| `/admin/reset-breakers` | POST | Operator control route (resets search-engine circuit breakers). Guarded by a per-process token written owner-only to disk; used by `wigolo doctor --fix`. |

## Auth model — fail closed

- **Loopback bind (default): open.** No token needed for local callers.
- **Non-loopback bind: token required.** Binding to anything that isn't loopback with no token configured **refuses to start** unless you pass `--allow-unauthenticated` (or `WIGOLO_SERVE_ALLOW_UNAUTHENTICATED=1`) — an explicit, logged operator override.

Set the token with `WIGOLO_API_TOKEN`, or `WIGOLO_API_TOKEN_FILE` to read it from a file (the Docker/systemd secret pattern — keeps it out of process listings). With a token configured, every `/v1`, `/openapi.json`, compat-shim, and MCP request needs `Authorization: Bearer <token>`; `/health` stays open.

The MCP transport additionally rejects any request carrying a browser `Origin` header (CLI/MCP clients never send one). In open mode it also requires an allowlisted loopback `Host`, blocking DNS-rebinding attacks; with a configured bearer token, remote MCP clients may use their normal Host header.

One more remote-exposure guard: under a non-loopback bind, tool calls targeting loopback/localhost URLs are refused (a remote caller could otherwise probe services on the daemon's own box). `WIGOLO_SERVE_ALLOW_LOCAL_TARGETS=1` opts back in. See [self-hosting](./self-hosting.md#network-posture).

## Resource limits

All transport policy, all env-tunable — the tool layer's own defaults are separate:

| Limit | Default | Override |
| --- | --- | --- |
| Request body | 1 MiB (5 MiB for `diff`/`extract`) | `WIGOLO_SERVE_MAX_BODY_BYTES` |
| Response deadline | 60s search/cache/diff/find_similar · 120s fetch/extract/watch · 300s crawl/research/agent | `WIGOLO_SERVE_TIMEOUT_SCALE` (multiplier) |
| In-flight requests | 16 | `WIGOLO_SERVE_MAX_CONCURRENCY` |
| Whole-request / headers timeout | 120s / 60s | `WIGOLO_SERVE_REQUEST_TIMEOUT_MS` / `WIGOLO_SERVE_HEADERS_TIMEOUT_MS` |

Server-side param clamps (also injected into the served OpenAPI bounds, so contract and enforcement can't drift): `crawl.max_pages` ≤ 200, `crawl.max_depth` ≤ 5, `agent.max_time_ms` ≤ 240000, `search.query` array ≤ 10 variants.

## Quickstart (recorded)

Start a daemon and probe it — these are real responses from `wigolo serve --port 3477`:

```bash
curl -s http://127.0.0.1:3477/health
```

```json
{"status":"healthy","searxng":"not_configured","browsers":"ready","cache":"active","uptime_seconds":6}
```

(The `searxng` field reports the optional legacy aggregator sidecar; `not_configured` is the normal state on the default backend.)

```bash
curl -s -X POST http://127.0.0.1:3477/v1/search \
  -H 'Content-Type: application/json' \
  -d '{"query":"wigolo local-first web intelligence","max_results":2,"search_depth":"fast"}'
```

```json
{"results":[{"title":"...","url":"https://github.com/KnockOutEZ/wigolo",
  "snippet":"wigolo runs on your machine as an MCP server ...",
  "relevance_score":0.968,
  "evidence_score":{"final":0.224,"components":{"base_rrf":0.023,"domain_quality":1,
    "lexical_alignment":0.4,"engine_consensus":2,"rare_terms":1.6}, "explanation":"..."}}],
 "engines_used":["bing","duckduckgo"], "...":"..."}
```

With a token:

```bash
WIGOLO_API_TOKEN=$(openssl rand -hex 32) wigolo serve --host 0.0.0.0 --port 3333
curl -s -H "Authorization: Bearer $WIGOLO_API_TOKEN" http://<host>:3333/v1/tools
```

## OpenAPI as the machine contract

`GET /openapi.json` returns OpenAPI **3.1.0** (`info.title: "wigolo REST API"`, versioned with the release — currently `0.2.0`) covering every route, parameter schema, and the enforced clamps. Generate clients from it, validate against it, or hand it to an agent as the tool contract. The [SDKs](./sdks.md) are drift-tested against this document.

## Compat shim

`WIGOLO_FIRECRAWL_COMPAT=1` enables an opt-in, experimental compatibility shim at `/compat/firecrawl` that accepts hosted-scraper-style requests — useful for pointing existing integrations at your own wigolo instead. It sits behind the same auth and target guards as everything else.

## Error shape

Non-2xx responses carry a consistent JSON body: `error` (message), `error_reason` (stable machine code, e.g. `unauthorized`, `host_not_allowed`), and where relevant a `hint` naming the exact env var or flag to fix it. Degraded-but-successful tool calls stay 2xx with in-body `warning`/`error` fields — inspect those rather than relying on status codes alone.

[← Docs index](./README.md) · [Next: SDKs](./sdks.md)
