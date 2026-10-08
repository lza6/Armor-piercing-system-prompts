# Self-hosting

wigolo is built to run where your agents run. If that's a laptop with a coding agent, [getting started](./getting-started.md) already covered you. This page is for the other deployments: a VPS running n8n or a custom agent loop, a homelab box serving several tools, a container platform. One daemon process gives every agent on that machine (or network) the same ten tools and one shared knowledge cache.

## Run it as a daemon

Any way you keep a process alive works. The daemon is a single Node process; no database server, no queue, no sidecar required.

**systemd** (a VPS-typical unit):

```ini
[Unit]
Description=wigolo web-intelligence daemon
After=network-online.target

[Service]
ExecStart=/usr/bin/npx wigolo serve --port 3333
Environment=WIGOLO_DATA_DIR=/var/lib/wigolo
# Token only needed if you bind beyond loopback (see below):
# Environment=WIGOLO_API_TOKEN_FILE=/etc/wigolo/token
Restart=on-failure
User=wigolo

[Install]
WantedBy=multi-user.target
```

**tmux/screen** (quick and dirty): `tmux new -d -s wigolo 'npx wigolo serve'`.

**Docker Compose**: use [`packaging/compose.serve.yml`](../packaging/compose.serve.yml) — it ships with a healthcheck, a persistent volume, and the token wiring commented inline. See [installation](./installation.md#docker).

## Binding beyond loopback

By default the daemon binds `127.0.0.1` and is open to local callers only. To serve other machines:

```bash
WIGOLO_API_TOKEN=$(openssl rand -hex 32) wigolo serve --host 0.0.0.0
```

The bind gate is **fail-closed**: a non-loopback bind with no token refuses to start. Your options, explicitly:

1. Set `WIGOLO_API_TOKEN` (or `WIGOLO_API_TOKEN_FILE` for the secret-mount pattern) — every REST/MCP request then needs `Authorization: Bearer <token>`. `/health` stays open for probes.
2. Pass `--allow-unauthenticated` — only sensible when your own auth layer sits in front.

Full auth semantics: [REST API](./rest-api.md#auth-model--fail-closed).

## Wiring self-hosted agents

Anything that can speak MCP-over-HTTP or plain HTTP can use the daemon:

- **MCP-capable agents** — point them at `http://<host>:3333/mcp` (StreamableHTTP; `/sse` for legacy clients) with the bearer token in their auth header config.
- **n8n and other HTTP-node automations** — plain REST: `POST http://<host>:3333/v1/search` (or any [tool route](./rest-api.md#endpoints)) with a JSON body and the `Authorization` header. The `/openapi.json` contract imports directly into OpenAPI-aware nodes.
- **Your own services** — use the [SDKs](./sdks.md) with `WIGOLO_BASE_URL` and `WIGOLO_API_TOKEN` pointed at the daemon.

One daemon, many callers: requests share the knowledge cache, so your research agent benefits from what your monitoring agent fetched an hour ago.

## Persistence

Everything lives under the data dir (`WIGOLO_DATA_DIR`, default `~/.wigolo`): the cache database, on-device models, keys, plugins. Persist that path — a Docker named volume, a mounted disk — and the daemon is otherwise stateless; you can kill and restart it freely. Watch jobs are persisted there too and resume with the daemon.

## Resource sizing

- **Disk**: roughly 250 MB for the on-device ranking + embedding models, plus the browser engine binary (~0.5–1 GB depending on how many browser families you install; one is the default). The cache grows with use and is bounded by your TTLs and `wigolo config --cleanup cache`.
- **Memory**: idle footprint is small (tens of MB); browser-rendered fetches are the spike — each pooled browser costs real memory, so on a small VPS set `MAX_BROWSERS=1`.
- **CPU**: reranking and embedding are short in-process bursts; a single shared vCPU handles interactive agent use.

## The datacenter-IP reality

Be aware of an honest ceiling before you deploy scraping-adjacent workflows to a VPS: anti-bot systems score **IP reputation**, and datacenter ranges start with low scores. Some challenge-protected sites will not clear from a datacenter IP no matter what the client does — the same fetch works fine from a residential connection. wigolo's tiered fetching and per-domain learning ([`wigolo tune`](./cli.md#tune)) get you the reliability that's achievable, and when a page can't be cleared you get a labeled `blocked_by_challenge` failure instead of junk parading as content.

The opt-in workaround for legitimate research that keeps hitting this wall is routing through a proxy whose IP reputation matches your use:

```bash
wigolo config --set useProxy=true
PROXY_URL=https://user:pass@proxy.example.com:8443 wigolo serve
```

Proxy credentials never persist to disk — the userinfo is moved to the OS keychain and only the credential-free URL is stored. Politeness still applies through a proxy: robots.txt on by default, per-domain rate limits, research-grade volumes.

## Network posture

- **SSRF guards by default.** Fetch/crawl/watch-webhook targets resolving to private or loopback addresses are refused. `WIGOLO_FETCH_ALLOW_PRIVATE=true` re-enables private targets for local-dev use.
- **Remote binds tighten further.** Under a non-loopback bind, loopback-literal targets (`localhost`, `127.x.x.x`, `::1`) are refused even when private fetching is allowed — a remote caller must not be able to probe services on the daemon's own box through wigolo. Opt out only deliberately: `WIGOLO_SERVE_ALLOW_LOCAL_TARGETS=1`.
- **DNS-rebinding + browser-origin guards** protect the MCP transport and admin routes; see [privacy & security](./privacy-security.md).

## Reverse proxy / tunnel

The daemon serves plain HTTP and does not terminate TLS. The recommended remote topology:

1. Bind wigolo to loopback (`127.0.0.1:3333`, the default).
2. Terminate TLS in front — a reverse proxy on the same box, or an outbound tunnel.
3. **Keep the bearer token anyway.** A tunnel delivers remote requests from 127.0.0.1, so loopback-source is deliberately not trusted as authentication; the token is what actually gates callers.

With a configured bearer token, a proxy may forward its normal public `Host` header. The MCP transport still rejects browser-originated requests; open-mode loopback deployments retain the Host allowlist as a DNS-rebinding guard.

[← Docs index](./README.md) · [Next: Skills](./skills.md)
