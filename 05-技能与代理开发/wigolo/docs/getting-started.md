# Getting started

From zero to your agent searching the web in about five minutes. You need Node.js 20 or newer.

## 1. Initialize

```bash
npx wigolo init
```

`init` is unattended by default — no prompts, safe in scripts and CI. It performs a complete setup: downloads the browser engine and the on-device ranking and embedding models, verifies each component, and prints a per-component report so failures surface loudly instead of hiding until first use. A degraded component doesn't abort setup — it's named in the report, init still exits 0 (agents wired, config persisted), and the component lazy-retries on first use. A non-zero exit is reserved for hard failures like a failed requested agent registration.

To wire your coding agent at the same time, name it:

```bash
npx wigolo init --agents=claude-code
```

`--agents` takes a comma-separated list (see the [full matrix](./installation.md#agent-auto-wire)). Omit it to set up the engine only and point any MCP client at wigolo yourself.

Useful variants:

- `npx wigolo init --no-warmup` — skip all downloads; components lazy-load on first use.
- `npx wigolo init --interactive` — plain-text prompt flow (agent picker, onboarding questions).
- `npx wigolo init --wizard` — the rich guided setup TUI.
- `npx wigolo init --json` — machine-readable summary on stdout.

## 2. First search — through your agent

If you wired an agent, just ask it something that needs the web. The agent now has ten wigolo tools (`search`, `fetch`, `crawl`, `cache`, `extract`, `find_similar`, `research`, `agent`, `diff`, `watch`) and instructions on when to reach for each.

## 3. First search — from the terminal

Every tool also runs as a one-shot CLI command:

```bash
npx wigolo search "css container queries" --limit=2
```

```text
Search: "css container queries" (2 results, 1357ms, engines: bing, duckduckgo)

  [1] CSS container queries - CSS | MDN - MDN Web Docs - developer.mozilla.org (score: 1.00)
      CSS container queries Container queries enable you to apply styles to an
      element based on certain attributes of its container ...

  [2] Using container size and style queries - CSS | MDN - developer.mozilla.org (score: 0.85)
      Using container size and style queries Container queries enable you to
      apply styles to elements nested within a specific container ...
```

Fetch a page as clean markdown:

```bash
npx wigolo fetch https://example.com --max-content-chars=400
```

```text
Fetch: https://example.com/

  This domain is for use in documentation examples without needing permission.
  Avoid use in operations.

  [Learn more](https://iana.org/domains/example)

  [cached: false, 149 chars]
```

Add `--json` to any tool command for a machine-readable result on stdout.

## 4. Check the install

```bash
npx wigolo doctor
```

`doctor` reports the data directory, browser engine, on-device models, configured LLM providers, the search backend, and per-engine status — including which optional engines want an API key and exactly which env var enables them. `wigolo doctor --fix` repairs known failures.

For an end-to-end capability smoke test (real network, real extraction):

```bash
npx wigolo verify
```

Exit code 0 means every capability passed or was skipped; 1 means something failed.

## Where to next

- [Configuration](./configuration.md) — search backends, LLM providers (optional), cache TTLs, proxies.
- [Tools](./tools.md) — what each of the 10 tools does and returns.
- [Installation](./installation.md) — Docker, agent matrix, and other channels.
- [REST API](./rest-api.md) — run wigolo as a daemon for remote agents.

[← Docs index](./README.md) · [Next: Installation](./installation.md)
