# wigolo-sdk

> ```bash
> npm i wigolo-sdk
> ```

A thin TypeScript client for a local-first wigolo web-intelligence daemon. It
speaks the daemon's REST API and nothing more: **no** retries, result
re-ranking, interpretation, or client-side caching. A `2xx` body is returned to
you verbatim; a non-`2xx` becomes a typed error. All the intelligence
(ML-reranked search, browser-engine fetches, structured extraction, the local
knowledge cache) lives in the daemon.

- **Zero runtime dependencies.** Uses the platform `fetch`.
- **Edge-safe core.** The main entry point imports no Node built-ins, so it runs
  on browsers, edge runtimes, Deno, and Node. The optional `wigolo-sdk/local`
  subpath is Node-only (it can spawn a daemon).
- **Node `>=18`.** `require(esm)` works on Node `>=22.12`.

## Quickstart

Point the client at a running daemon (start one with `wigolo serve`):

```ts
import { WigoloClient } from 'wigolo-sdk';

const client = new WigoloClient({
  // baseUrl defaults to http://127.0.0.1:3333 (or WIGOLO_BASE_URL)
  // token defaults to WIGOLO_API_TOKEN — only needed when the server sets one
});

const res = await client.search({ query: 'local-first web search', max_results: 3 });
console.error(`${res.results?.length ?? 0} results`);
```

### Embedded local mode (Node only)

No daemon running? `createLocalClient` reuses one on the port or spawns
`wigolo serve` for you, and hands back a lifecycle you own:

```ts
import { createLocalClient } from 'wigolo-sdk/local';

const { client, owned, close } = await createLocalClient();
try {
  const page = await client.fetch({ url: 'https://example.com' });
  console.error(page.title);
} finally {
  await close(); // stops the daemon only if this call spawned it (owned === true)
}
```

Resolution order for the spawn command is `command` option > `WIGOLO_CLI` env
> `wigolo` on `PATH`. Port is `port` option > `WIGOLO_LOCAL_PORT` env > 3333.

#### Security notes for embedded mode

- **`WIGOLO_CLI` is an exec-from-env vector.** In embedded mode the SDK spawns
  the process named by `WIGOLO_CLI` (a JSON argv array, or a single executable
  path). Anything that can set this env var chooses what binary runs. If you
  pass the SDK untrusted environments, strip `WIGOLO_CLI` (and
  `WIGOLO_LOCAL_PORT`) before construction and pass the trusted argv through the
  explicit `command` option — the option always overrides the env.
- **Point `command` at the server binary itself, not a wrapper.** On POSIX,
  forced-kill escalation signals only the direct child. A wrapper like
  `["npx", "wigolo"]` makes the launcher the direct child, so a hung
  `close()` can kill the launcher while the real daemon it spawned is orphaned
  and keeps holding the port. Resolve to the actual `wigolo` executable so
  `close()` reaches the process that owns the socket.

## Methods

One method per tool (camelCase), each POSTing its params object verbatim:

`search`, `fetch`, `crawl`, `cache`, `extract`, `findSimilar`, `research`,
`agent`, `diff`, `watch`, plus `health()`, `listTools()`, `openapi()`.

All methods are bound arrow fields, so destructuring is safe:

```ts
const { search, fetch } = new WigoloClient();
```

### camelCase methods, snake_case wire fields

The method names are camelCase (`findSimilar`), but request and response field
names are the daemon's snake_case wire names (`max_results`, `total_time_ms`,
`response_time_ms`). Types reflect the wire shape.

## Options

```ts
new WigoloClient({
  baseUrl,   // > WIGOLO_BASE_URL > http://127.0.0.1:3333
  token,     // > WIGOLO_API_TOKEN (sent as `Authorization: Bearer <token>`)
  timeoutMs, // default per-request deadline; overrides the per-tool default
  fetch,     // injectable fetch (tests / custom transports)
});
```

Per-call overrides:

```ts
await client.research({ question: 'q' }, { timeoutMs: 120_000, signal: myAbortSignal });
```

Explicit options win over env; env is read only when the option is absent (and
every env read is guarded, so a runtime that throws on env access — e.g. Deno
without `--allow-env` — does not crash construction).

## Timeouts

Each tool has a default deadline that **mirrors the server's unscaled per-route
deadline**:

| Tool | Default (ms) |
| --- | --- |
| search, cache, find_similar | 75000 |
| fetch, extract, watch | 135000 |
| crawl, research, agent | 315000 |

> If the server runs with `WIGOLO_SERVE_TIMEOUT_SCALE > 1`, its real deadlines
> are larger than these client defaults. Raise `timeoutMs` to match, or the
> client may abort a request the server would still complete.

## Errors

- `WigoloApiError` — a non-`2xx` response. Fields: `status`, `error`,
  `error_reason`, `stage`, and `retryAfter` (parsed from `Retry-After`, present
  on `429`). An unparseable error body falls back to a raw snippet in `error`.
- `WigoloConnectionError` — a transport failure (connection refused, timeout,
  abort). A connection-refused message points at `createLocalClient`.
- Both extend `WigoloError`.

A degraded-but-successful call stays `2xx` and is returned verbatim — inspect
in-body `warning` / `error` fields; the client never throws on a `2xx`.

## Notes

- `stream` (on `research` / `agent`) is accepted by the schema but **inert over
  this transport** — there is no notification channel over REST.
- `crawl` with `strategy: 'map'` returns `urls` (and no `pages`); the other
  strategies return `pages`.
- The `/health` response's `searxng` field is the search-aggregator sidecar
  status.

## License

AGPL-3.0. See `LICENSE`.
