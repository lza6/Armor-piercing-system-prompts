# REST + curl

`wigolo serve` exposes every tool as a plain HTTP endpoint: `POST /v1/{tool}`
with a JSON body, `GET /v1/tools` for discovery, `GET /openapi.json` for the
full machine-readable contract, `GET /health` for liveness. Anything that can
speak HTTP — curl, a cron job, another service — can use wigolo.

## Run it

```bash
./demo.sh                                      # open mode on 127.0.0.1
WIGOLO_API_TOKEN=wigolo-demo-token ./demo.sh   # bearer-token mode
WIGOLO="wigolo" PORT=4000 ./demo.sh            # global install, custom port
```

The script starts a daemon on `127.0.0.1:3477`, exercises the endpoints, and
tears the daemon down on exit. Requires node >= 20, curl, jq.

## What you'll see

```console
$ curl -s http://127.0.0.1:3477/health | jq .
{
  "status": "healthy",
  "searxng": "not_configured",
  "browsers": "ready",
  "cache": "active",
  "uptime_seconds": 0
}

$ curl -s http://127.0.0.1:3477/v1/tools | jq -r 'map(.name) | join("  ")'
search  fetch  crawl  cache  extract  find_similar  research  agent  diff  watch

$ curl -s -X POST http://127.0.0.1:3477/v1/search \
    -H 'Content-Type: application/json' \
    -d '{"query": "typescript satisfies operator", "max_results": 3, "include_content": false}' \
    | jq '{results: [.results[] | {title, url}], total_time_ms}'
{
  "results": [
    { "title": "Documentation - TypeScript 4.9",
      "url": "https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html" },
    { "title": "Mastering the `satisfies` Operator in TypeScript",
      "url": "https://www.xjavascript.com/blog/satisfies-typescript/" },
    { "title": "The satisfies Operator - Learning TypeScript",
      "url": "https://www.learningtypescript.com/articles/the-satisfies-operator" }
  ],
  "total_time_ms": 2
}

$ curl -s http://127.0.0.1:3477/openapi.json | jq '.info.version'
"0.2.0"
```

(`total_time_ms: 2` is a warm local-cache answer — the first query for a topic
takes a couple of seconds while the engines are consulted.)

## Bearer tokens

Set `WIGOLO_API_TOKEN` (or point `WIGOLO_API_TOKEN_FILE` at a secret file) and
the daemon requires `Authorization: Bearer <token>` on `/v1/*` and
`/openapi.json`. `/health` stays open for probes:

```console
$ curl -s -o /dev/null -w '%{http_code}\n' http://127.0.0.1:3477/v1/tools
401
$ curl -s -o /dev/null -w '%{http_code}\n' \
    -H 'Authorization: Bearer wigolo-demo-token' http://127.0.0.1:3477/v1/tools
200
```

The daemon binds `127.0.0.1` by default. Binding a non-loopback host
(`--host 0.0.0.0`) **refuses to start without a token** unless you explicitly
opt out — see [n8n-remote-mcp](../n8n-remote-mcp/) for the remote-access setup.

## Beyond curl

The same daemon also serves the MCP protocol at `/mcp` (and legacy SSE at
`/sse`) for agent frameworks, and the typed SDKs ride this exact REST surface:
[sdk-typescript-research](../sdk-typescript-research/),
[sdk-python-agent](../sdk-python-agent/).
