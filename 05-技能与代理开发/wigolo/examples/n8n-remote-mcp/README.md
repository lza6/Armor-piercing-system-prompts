# n8n / any remote client → wigolo

> **Config reference.** Nothing to run here — this page shows how to point a
> self-hosted [n8n](https://n8n.io) (or *any* remote MCP or REST client) at a
> `wigolo serve` instance. Every endpoint shown was verified against wigolo
> 0.2.0.

One `wigolo serve` process exposes three surfaces at once:

| Surface | URL | Speaks |
|---|---|---|
| MCP (streamable HTTP) | `http://HOST:3477/mcp` | MCP clients: n8n MCP Client node, agent frameworks |
| MCP (legacy SSE) | `http://HOST:3477/sse` (+ `/messages`) | older MCP clients |
| REST | `http://HOST:3477/v1/{tool}` | anything that can POST JSON |

Plus `GET /health` (open), `GET /v1/tools` and `GET /openapi.json` (token-gated
when a token is set).

## 1. Start the server

```bash
# reachable from other machines/containers -> a token is REQUIRED
WIGOLO_API_TOKEN=wigolo-demo-token npx wigolo serve --port 3477 --host 0.0.0.0
```

wigolo is fail-closed here: binding a non-loopback host **refuses to start
without a token** (set `WIGOLO_API_TOKEN`, or point `WIGOLO_API_TOKEN_FILE` at
a docker/systemd secret). There is an explicit `--allow-unauthenticated` opt-out
for isolated networks — prefer the token.

Running n8n in docker on the same machine? `--host 0.0.0.0` plus
`http://host.docker.internal:3477` from inside the container is the usual pair.

## 2. Point n8n at it

**MCP way (n8n >= 1.88, AI Agent + MCP Client Tool node):**

- Endpoint: `http://YOUR-WIGOLO-HOST:3477/mcp`
- Server Transport: *HTTP Streamable* (pick *SSE* and the `/sse` URL only for
  older builds)
- Authentication: *Bearer* → `wigolo-demo-token`

Your agent then sees all ten wigolo tools (`search`, `fetch`, `crawl`,
`cache`, `extract`, `find_similar`, `research`, `agent`, `diff`, `watch`) and
calls them like any other MCP tool.

> **Host-header rule (MCP endpoints only).** As a DNS-rebinding guard, `/mcp`
> and `/sse` accept only requests whose `Host` is loopback **or exactly the
> `--host` value the daemon was started with** — anything else is a
> `403 host_not_allowed`, even with a valid token. Two clean setups:
>
> 1. **Bind the address clients will use.** `--host 192.168.1.20` and point
>    n8n at `http://192.168.1.20:3477/mcp` — Host matches, done. (A `0.0.0.0`
>    bind does *not* whitelist your LAN IP.)
> 2. **Front it with a proxy/tunnel that rewrites `Host`** to `127.0.0.1`
>    while forwarding to the daemon — the usual pattern for DNS names and
>    public tunnels.
>
> The REST surface (`/v1/*`) is **not** Host-gated — the bearer token is its
> gate — so the HTTP Request node path below works from anywhere as-is.

**REST way (plain HTTP Request node):** import
[`workflow.json`](./workflow.json) — a manual trigger wired to an HTTP Request
node that POSTs `/v1/search` with the `Authorization: Bearer` header. Swap the
URL/token, replace the trigger with whatever starts your flow, and branch on
the JSON that comes back.

## 3. The same call, without n8n

Everything the workflow does is one curl — this is what "any remote client"
means in practice:

```bash
curl -s -X POST http://YOUR-WIGOLO-HOST:3477/v1/search \
  -H 'Authorization: Bearer wigolo-demo-token' \
  -H 'Content-Type: application/json' \
  -d '{"query": "typescript satisfies operator", "max_results": 3, "include_content": false}'
```

And the MCP handshake, if you're wiring a custom MCP client (verified
response: `"serverInfo": {"name": "wigolo", "version": "0.2.0"}`):

```bash
curl -s -X POST http://YOUR-WIGOLO-HOST:3477/mcp \
  -H 'Content-Type: application/json' \
  -H 'Accept: application/json, text/event-stream' \
  -H 'Authorization: Bearer wigolo-demo-token' \
  -d '{"jsonrpc":"2.0","id":1,"method":"initialize","params":{"protocolVersion":"2025-03-26","capabilities":{},"clientInfo":{"name":"my-client","version":"0.0.1"}}}'
```

## Notes for remote setups

- **Browser requests are rejected on purpose.** The MCP transport refuses any
  request carrying an `Origin` header (and non-allowlisted `Host` values), so
  a web page can't probe your daemon. Server-side clients only.
- **Tool timeouts:** `crawl`, `research`, and `agent` can legitimately run
  minutes. Raise your n8n HTTP node timeout accordingly (the per-route server
  deadlines are in `GET /openapi.json`).
- **TLS / public exposure:** put your usual reverse proxy or tunnel in front;
  wigolo itself serves plain HTTP. For the MCP endpoints, have the proxy
  rewrite `Host` to a loopback value (see the Host-header rule above); REST
  only needs the bearer token forwarded.
- Want the full endpoint walk-through with live output? See
  [rest-curl](../rest-curl/).
