# wigolo — Python client

> ```bash
> pip install wigolo
> ```

A thin, dependency-free Python client for the [wigolo](https://github.com/KnockOutEZ/wigolo)
local-first web intelligence server. It speaks the wigolo REST API and nothing
more — no retries, no re-ranking, no interpretation, no caching. The server
does all of that; this client just gets requests there and responses back with
typed method signatures.

- **Zero runtime dependencies** — stdlib only.
- **Sync and async** clients with an identical surface.
- **Embedded local mode** — `local_client()` probes or spawns a local server
  for you, no manual setup.
- **Fully typed** (`py.typed`, PEP 561).

## Tools

One method per tool: `search`, `fetch`, `crawl`, `cache`, `extract`,
`find_similar`, `research`, `agent`, `diff`, `watch`, plus `health()`,
`list_tools()`, and `openapi()`.

## Sync quickstart

```python
from wigolo import Client

with Client(base_url="http://127.0.0.1:3333") as client:
    res = client.search(query="local first web search", max_results=5)
    for r in res.get("results", []):
        print(r.get("title"), r.get("url"))

    page = client.fetch(url="https://example.com")
    print(page.get("markdown", "")[:500])

    tables = client.extract(
        html="<table><tr><th>a</th></tr><tr><td>1</td></tr></table>",
        mode="tables",
    )
    print(tables["data"])
```

## Async quickstart

```python
import asyncio
from wigolo import AsyncClient

async def main():
    async with AsyncClient(base_url="http://127.0.0.1:3333") as client:
        res = await client.search(query="local first web search", max_results=5)
        print([r.get("url") for r in res.get("results", [])])

asyncio.run(main())
```

The async client runs each request on a bounded thread pool
(`max_workers`, default 16). Cancelling an awaited call returns promptly but
**abandons** the in-flight request — the worker thread runs to completion in
the background (a blocking socket read cannot be portably aborted).

## Embedded local mode (zero setup)

```python
from wigolo import local_client

# Reuses a healthy local daemon if one is already listening; otherwise spawns
# one for you and waits for it to become healthy.
with local_client() as client:
    print(client.health())
    print(client.search(query="wigolo"))
```

`local_client()` (equivalently `Client(local=True)`, or setting
`WIGOLO_LOCAL=1`) resolves a port (`WIGOLO_LOCAL_PORT`, default `3333`), probes
`/health`, and either reuses an existing REST-capable daemon or spawns a new
one via the `wigolo` CLI on your `PATH` (override with `WIGOLO_CLI`). A daemon
this client spawns is stopped on `close()`; a daemon it merely reused is left
running. In local mode `base_url` / `WIGOLO_BASE_URL` are ignored.

`AsyncClient(local=True)` accepts the same `port` / `command` overrides, but
note the daemon probe-or-spawn runs **synchronously in the constructor** (it
may block the calling thread up to ~20s while the daemon becomes healthy).

### Security notes for embedded mode

- **`WIGOLO_CLI` is an exec-from-env vector.** In embedded mode the SDK spawns
  the process named by `WIGOLO_CLI` (a JSON argv list, or a single executable
  path). Anything that can set this env var chooses what binary runs. When you
  hand the SDK untrusted environments, strip `WIGOLO_CLI` (and
  `WIGOLO_LOCAL_PORT`) before construction and pass the trusted argv through the
  explicit `command=` argument — the argument always overrides the env.
- **Point `command` at the server binary itself, not a wrapper.** On POSIX,
  forced-kill escalation signals only the direct child. A wrapper such as
  `["npx", "wigolo"]` makes the launcher the direct child, so a hung `close()`
  can kill the launcher while the real daemon it spawned is orphaned and keeps
  holding the port. Resolve to the actual `wigolo` executable so `close()`
  reaches the process that owns the socket.

## Configuration

Resolution order for each option is **explicit argument > environment
variable > default**. When an argument is passed explicitly, the corresponding
env var is not consulted.

| Option     | Argument   | Env var             | Default                 |
|------------|------------|---------------------|-------------------------|
| Base URL   | `base_url` | `WIGOLO_BASE_URL`   | `http://127.0.0.1:3333` |
| Bearer token | `token`  | `WIGOLO_API_TOKEN`  | none                    |
| Local mode | `local`   | `WIGOLO_LOCAL=1`    | off                     |
| Local port | `port`    | `WIGOLO_LOCAL_PORT` | `3333`                  |
| Spawn command | `command` | `WIGOLO_CLI`     | `wigolo` on `PATH`      |

The bearer token is only sent when set — the server requires it only when it
runs with a token configured.

## Timeouts

The `timeout` option (per client, or per call) is a **per-socket-operation**
timeout — the maximum idle time on connect or read inactivity — **not** a total
wall-clock deadline. When unset, each method uses its per-tool default, which
mirrors the server's **unscaled** per-route deadline. If your server runs with
`WIGOLO_SERVE_TIMEOUT_SCALE` set, its effective deadline scales but the client
defaults do not — pass an explicit `timeout` to match.

## Notes

- `stream` on `research` / `agent` is accepted by the schema but has **no
  effect over this transport** — responses are returned whole.
- A degraded call stays HTTP 200 with in-body `warning` / `error` fields;
  those are returned verbatim and never raise.
- `crawl(strategy="map")` returns `urls` (no `pages`); the other strategies
  return `pages`.
- The `searxng` field in `health()` is the search-aggregator sidecar status.

## Errors

- `WigoloError` — base class.
- `WigoloAPIError` — a non-2xx HTTP response, carrying `status`, `error`,
  `error_reason`, `stage`, and `retry_after` (parsed from `Retry-After` on 429).
- `WigoloConnectionError` — a transport-level failure (e.g. connection
  refused) with no HTTP response. Its message points at `local_client()` as
  the zero-setup path.

## License

AGPL-3.0-only. See `LICENSE`.
