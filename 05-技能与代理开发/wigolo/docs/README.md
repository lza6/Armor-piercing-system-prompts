# wigolo documentation

wigolo is a local-first web-intelligence server for AI agents: search, fetch, crawl, extract, research, and a persistent knowledge cache, all running on your machine with no API keys required for core work. It speaks MCP over stdio for local coding agents (Claude Code, Cursor, and friends), exposes a remote MCP endpoint and a REST API for self-hosted agents (n8n, VPS automations, your own services), ships SDKs for agent frameworks, and doubles as a plain CLI for scripts.

Everything the server returns is transparent: per-result evidence scores, per-engine telemetry, honest failure labels. Everything it stores stays in `~/.wigolo` on your disk.

## Pages

| Page | What's in it |
| --- | --- |
| [Getting started](./getting-started.md) | The 5-minute path: `npx wigolo init`, wire an agent, run your first search. |
| [Installation](./installation.md) | Every install channel — npm, Docker, MCP bundle, registries — plus the agent auto-wire matrix and uninstall. |
| [Configuration](./configuration.md) | Resolution order, the settings TUI, and grouped env-var tables for search, fetch, models, LLM providers, cache, and serve. |
| [Tools](./tools.md) | The 10 tools with parameters, response fields, and worked examples. |
| [CLI](./cli.md) | Full command reference: management commands, one-shot tools, the interactive shell, and the `--json` contract. |
| [REST API](./rest-api.md) | `wigolo serve`, endpoints, the fail-closed auth model, resource limits, and a live curl quickstart. |
| [SDKs](./sdks.md) | TypeScript and Python clients, plus LangChain, CrewAI, LlamaIndex, and Vercel AI SDK integrations. |
| [Self-hosting](./self-hosting.md) | Running wigolo where your agents run: VPS, Docker, tokens, reverse proxies, and honest notes on datacenter IPs. |
| [Skills](./skills.md) | Agent skill packs: the 11-pack catalog, install scopes, and the receipts model. |
| [Plugins](./plugins.md) | Extending wigolo with your own search engines and content extractors. |
| [Troubleshooting](./troubleshooting.md) | Symptom-to-fix table, platform notes, and the FAQ. |
| [Privacy & security](./privacy-security.md) | What lives on disk, what leaves your machine, credential handling, and responsible disclosure. |

## Getting help

- Bugs and feature requests: [GitHub issues](https://github.com/KnockOutEZ/wigolo/issues)
- Questions and ideas: [GitHub discussions](https://github.com/KnockOutEZ/wigolo/discussions)
- Security reports: see [privacy & security](./privacy-security.md#responsible-disclosure) — please do not open public issues for vulnerabilities.

[Next: Getting started](./getting-started.md)
