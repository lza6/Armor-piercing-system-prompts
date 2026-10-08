# wigolo-crewai

CrewAI tools for [wigolo](https://github.com/KnockOutEZ/wigolo) — local-first web
intelligence for your crews. Give agents web search, page fetch, multi-step
research, site crawl, and structured extraction, with results returned as JSON
your agents can reason over.

Thin wrapper over the wigolo Python SDK: no API keys required for the core
tools, and a zero-setup embedded daemon starts automatically.

## Install

```bash
pip install 'wigolo-crewai[crewai]'
```

This pulls in the `wigolo` SDK. `crewai` is an optional extra so the core
helpers can be imported without it.

## Quickstart

```python
from crewai import Agent
from wigolo_crewai import wigolo_tools

# Spawns a local wigolo daemon by default (local=True). Point at a running
# server instead with wigolo_tools(base_url="http://127.0.0.1:8787", token="...").
tools = wigolo_tools()

researcher = Agent(
    role="Web Researcher",
    goal="Find and summarize current information from the web",
    backstory="You dig through the web and return well-sourced findings.",
    tools=tools,
)
```

You can also add individual tools:

```python
from wigolo_crewai import WigoloSearchTool, WigoloResearchTool, build_client

client = build_client(local=True)
agent_tools = [WigoloSearchTool(client=client), WigoloResearchTool(client=client)]
```

## Tools

| Tool | Class | What it does |
|------|-------|--------------|
| `wigolo_search`   | `WigoloSearchTool`   | Search the web; ML-reranked results with extracted content |
| `wigolo_fetch`    | `WigoloFetchTool`    | Fetch a page as clean markdown (JS rendering optional) |
| `wigolo_research` | `WigoloResearchTool` | Multi-step research brief: topics, findings, cross-references, gaps |
| `wigolo_crawl`    | `WigoloCrawlTool`    | Crawl a site (sitemap / breadth-first / depth-first / map) |
| `wigolo_extract`  | `WigoloExtractTool`  | Structured extraction: tables, metadata, key-values, schema fields |

Every tool returns a JSON string; wigolo errors are returned as a clean
`{"error": "..."}` string rather than raised into the agent loop.

## Configuration

`wigolo_tools(base_url=None, token=None, local=True)`:

- `local=True` (default) — spawn an embedded local daemon, zero setup.
- `base_url` / `token` — target a running wigolo server (set `local=False`).

Environment variables `WIGOLO_BASE_URL`, `WIGOLO_API_TOKEN`, and `WIGOLO_LOCAL`
are honored by the underlying SDK when the corresponding argument is left unset.

## License

GNU AGPL-3.0-only — see the root LICENSE.
