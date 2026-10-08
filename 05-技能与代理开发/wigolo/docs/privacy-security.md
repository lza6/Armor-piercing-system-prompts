# Privacy & security

wigolo's privacy model is structural, not a policy promise: the software runs on your machine, stores on your disk, and has no vendor backend to send anything to.

## Everything stays local

The whole state of a wigolo install lives in the data dir (`~/.wigolo` by default):

| Path | Contents |
| --- | --- |
| `wigolo.db` | The knowledge cache: pages, search results, full-text + vector indexes. |
| `jobs.db` | Watch jobs. |
| model caches | The on-device embedding and ranking models (two directories, created at `init`/`warmup`). |
| `config.json` | Non-secret settings (secrets are excluded by design — see below). |
| `keys/` | Encrypted credential files, only when the OS keychain isn't available. |
| `plugins/` | Installed [plugins](./plugins.md). |
| `skills/receipts.json` | The [skills](./skills.md) install ledger. |
| `shell-history` | Interactive shell history. |
| `telemetry/` | Opt-in local event files — absent unless you enable telemetry. |
| `searxng/` | The optional legacy aggregator sidecar, only if you opted into that backend. |
| `daemon-admin.token` | Per-process admin-route token (owner-only file permissions, rotated each daemon start). |

`rm -rf ~/.wigolo` erases all of it. `wigolo config --storage` shows what's using space.

## Network egress

wigolo makes outbound connections only to:

1. The **search engines and websites your queries target** — that's the product working.
2. The **LLM provider you configure**, if you configure one. Keyless local setups (including `WIGOLO_LOCAL_LLM=auto` against a local model server) never leave the machine for synthesis.
3. A **telemetry endpoint you set yourself** — see below; there is no default.

Component downloads (browser engine, models) fetch from their public distribution sources during `init`/`warmup` or first use. There is no license check, no update phone-home, no account.

## Telemetry: off, local, yours

Off by default. If you opt in with `WIGOLO_TELEMETRY=1`, events are appended to a local NDJSON file (`~/.wigolo/telemetry/events-YYYYMMDD.ndjson`) — nothing is transmitted. A network POST happens only if you additionally set `WIGOLO_TELEMETRY_ENDPOINT` to a URL of your own; wigolo ships no vendor endpoint to send to. `wigolo doctor` always shows the current telemetry state.

## Credentials

Secrets never sit in plaintext on disk:

- **LLM API keys** go to the OS keychain. Where no keychain is available, they're written as AES-256-GCM-encrypted files under `~/.wigolo/keys/`. `init` reads a key only from the `WIGOLO_LLM_API_KEY` env var — never from a flag, so it can't leak into shell history or process listings.
- **Proxy credentials** (and the URLs for opt-in solver/reader services): userinfo is split off and stored in the keychain; only the credential-free URL is persisted to `config.json`.
- **`wigolo config --export`** excludes secrets, so a shared settings file can't leak keys.
- The **REST bearer token** supports `WIGOLO_API_TOKEN_FILE` so deployments can mount it as a secret instead of an env var visible in process inspection.

## Serve-mode hardening

The HTTP daemon assumes the network is hostile:

- **Fail-closed bind gate** — a non-loopback bind with no token refuses to start; open remote access requires a named override flag. ([Details](./rest-api.md#auth-model--fail-closed).)
- **Bearer auth** on the REST + MCP surface when a token is configured; `/health` alone stays open for probes.
- **DNS-rebinding guard** — requests whose `Host` header isn't loopback (or the configured bind host) are rejected, so a malicious page resolving its domain to `127.0.0.1` gets a 403, not your daemon.
- **Browser-origin guard** — any request carrying an `Origin` header is rejected on the MCP and admin routes before token checking, so web pages can't probe token validity.
- **Loopback source is never trusted as auth** — tunnels deliver remote traffic from 127.0.0.1, so authentication is the token, not the source address.
- **Slow-client timeouts, body caps, concurrency caps** bound resource use ([limits](./rest-api.md#resource-limits)).

## SSRF guards

URL-taking surfaces refuse targets that resolve to private or loopback address space:

- `fetch` / `crawl` / every URL-bearing REST route — private targets blocked unless you opt in for local dev (`WIGOLO_FETCH_ALLOW_PRIVATE=true`).
- `watch` **webhook destinations** — a watch notification can't be pointed at your internal network.
- Remote-exposed daemons additionally refuse loopback-literal targets outright, so a remote caller can't use wigolo to probe services on its own host ([posture](./self-hosting.md#network-posture)).

## Responsible disclosure

Please don't open public issues for vulnerabilities. Report privately via GitHub's "Report a vulnerability" on the repository's Security tab — the process, scope, and response expectations are in [SECURITY.md](../SECURITY.md).

[← Docs index](./README.md) · [Back to start: Getting started](./getting-started.md)
