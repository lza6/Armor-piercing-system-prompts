# Installation

wigolo runs anywhere Node.js 20+ runs. npm is the primary channel; Docker images are published for container setups. A few more channels (single-file binary, Homebrew, hosted install script) are packaged in the repo and publish with an upcoming release — they're listed at the bottom so you don't chase an artifact that isn't live yet.

## npm / npx (primary)

No install needed — `npx` resolves the published package:

```bash
npx wigolo init
```

Or install globally:

```bash
npm install -g wigolo
wigolo init
```

`init` performs a complete setup by default (browser engine + on-device models + verification, with a per-component report). Pass `--no-warmup` to defer downloads to first use. See [getting started](./getting-started.md) for the init walkthrough.

## Docker

The published image is `ghcr.io/knockoutez/wigolo`. It's the slim variant: the browser engine binary and on-device models download on first use into the `/data` volume, keeping the image small and the downloads persistent.

MCP over stdio (one local client):

```bash
docker run -i --rm -v wigolo-data:/data ghcr.io/knockoutez/wigolo
```

Wire it into an MCP host, e.g.:

```bash
claude mcp add wigolo -- docker run -i --rm -v wigolo-data:/data ghcr.io/knockoutez/wigolo
```

HTTP daemon (remote MCP + REST, `/health` endpoint, multi-client) — use the compose file at [`packaging/compose.serve.yml`](../packaging/compose.serve.yml):

```bash
docker compose -f packaging/compose.serve.yml up
```

The compose file binds `0.0.0.0` inside the container, which is a non-loopback bind: the daemon **fails closed** and refuses to start until you set `WIGOLO_API_TOKEN` (uncomment the line in the file) or explicitly opt into open access. Details in [self-hosting](./self-hosting.md).

The repo's `Dockerfile` also has a `full` build target with the browser engine preinstalled at build time — useful for `--rm`/no-volume runs where first-use downloads would repeat:

```bash
docker build --target full -t wigolo:full .
```

The named volume persists the local cache, on-device models, the browser engine binary, and encrypted keys across restarts.

## MCP bundle (.mcpb)

The repo ships an MCP bundle manifest under [`mcpb/`](../mcpb/) for desktop hosts that support one-click MCP bundle installs. Prebuilt `.mcpb` artifacts publish with an upcoming release; until then, npm is the recommended path for desktop hosts too.

## MCP registries

wigolo carries registry manifests at the repo root — `smithery.yaml`, `glama.json`, and `mcp.json` (a plain MCP server config you can copy) — and is listed on MCP directories including [Glama](https://glama.ai/mcp/servers/KnockOutEZ/wigolo) and [Smithery](https://smithery.ai/server/ktowhid20/wigolo).

## Agent auto-wire

`wigolo init --agents=<csv>` (or `wigolo setup mcp`) writes the MCP server entry — and, where the host supports them, instructions and skills — for these targets:

| Agent | id |
| --- | --- |
| Claude Code | `claude-code` |
| Cursor | `cursor` |
| VS Code | `vscode` |
| Gemini CLI | `gemini-cli` |
| Zed | `zed` |
| Windsurf | `windsurf` |
| Codex | `codex` |
| OpenCode | `opencode` |
| Antigravity | `antigravity` |

```bash
npx wigolo init --agents=claude-code,cursor
```

For OpenCode, wigolo writes the global `~/.config/opencode/opencode.json` entry in OpenCode's local MCP format:

```json
{
  "mcp": {
    "wigolo": {
      "type": "local",
      "command": ["npx", "-y", "wigolo"],
      "enabled": true
    }
  }
}
```

Cline isn't an `--agents` target — wire its MCP entry by hand (next section) and it still gets the full [skill packs](./skills.md) via `wigolo skills add`.

### Any other MCP host

Point it at the stdio server with this config block (this is exactly what ships in `mcp.json`):

```json
{
  "mcpServers": {
    "wigolo": {
      "command": "npx",
      "args": ["-y", "wigolo"]
    }
  }
}
```

For hosts that speak HTTP instead of stdio, run `wigolo serve` and use `http://127.0.0.1:3333/mcp` — see the [REST API](./rest-api.md).

## Uninstall

```bash
npx wigolo uninstall
```

Removes all agent integrations: the MCP server config, the global instructions block, installed skills, and the slash command. It deliberately does **not** remove `~/.wigolo` (cache, models, keys). Full cleanup:

```bash
rm -rf ~/.wigolo
```

## Coming with a future release

These channels exist as source in [`packaging/`](../packaging/) but their published artifacts are not live yet — the install commands below will not work until they are:

- **Install script** — `packaging/install.sh`, a POSIX bootstrap that installs under `~/.wigolo` with a pinned, checksum-verified runtime (no root, no system packages).
- **Homebrew** — a formula at `packaging/homebrew/wigolo.rb`; the public tap is not yet published.
- **Single-file binary** — bundling scripts at `packaging/binary/`; no binary assets are attached to releases yet.

[← Docs index](./README.md) · [Next: Configuration](./configuration.md)
