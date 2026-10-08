# wigolo-langchain

LangChain integration for [wigolo](https://github.com/KnockOutEZ/wigolo) — a local-first web search MCP server for AI coding agents.

## Installation

```bash
pip install wigolo-langchain
```

Requires wigolo to be available via npx:
```bash
npm install -g wigolo
# or use npx (default, no install needed)
```

## Quick Start

### As a Retriever

```python
from wigolo_langchain import WigoloMcpClient, WigoloSearchRetriever

async def main():
    async with WigoloMcpClient() as client:
        retriever = WigoloSearchRetriever(
            client=client,
            max_results=5,
            include_domains=["docs.python.org"],
        )
        docs = await retriever.ainvoke("Python asyncio tutorial")
        for doc in docs:
            print(doc.metadata["title"], doc.metadata["url"])
```

### As LangChain Tools (for Agents)

```python
from wigolo_langchain import WigoloMcpClient, WigoloSearchTool, WigoloFetchTool

async def main():
    async with WigoloMcpClient() as client:
        search = WigoloSearchTool(client=client)
        fetch = WigoloFetchTool(client=client)

        # Use in a LangChain agent
        tools = [search, fetch]
```

### Custom MCP Server Command

```python
# Use a local development build instead of npx
client = WigoloMcpClient(command="node", args=["./dist/index.js"])
```

## API Reference

### WigoloMcpClient

Async MCP client that communicates with wigolo via subprocess.

- `command` — executable to run (default: `"npx"`)
- `args` — arguments (default: `["wigolo"]`)
- `timeout` — request timeout in seconds (default: `30.0`)
- `connect()` / `disconnect()` — lifecycle management
- `call_tool(name, arguments)` — invoke any wigolo MCP tool
- Supports `async with` context manager

### WigoloSearchRetriever

LangChain `BaseRetriever` that searches via wigolo.

- `client` — `WigoloMcpClient` instance
- `max_results` — max search results (default: 5)
- `include_domains` — domain whitelist
- `category` — "general", "code", "docs", "news", "papers"

### WigoloSearchTool / WigoloFetchTool

LangChain `BaseTool` wrappers for wigolo search and fetch.

- Accept the same parameters as the wigolo MCP tools
- Return JSON strings (for agent consumption)
- Handle errors gracefully (return error JSON instead of raising)

## License

[GNU AGPL-3.0-only](../../LICENSE) — same license as the parent wigolo project. Full terms in the root `LICENSE`.
