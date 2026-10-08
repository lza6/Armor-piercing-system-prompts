# wigolo-llamaindex

LlamaIndex reader for [wigolo](https://github.com/KnockOutEZ/wigolo) — a local-first web search MCP server for AI coding agents.

## Installation

```bash
pip install wigolo-llamaindex
```

Requires wigolo to be available via npx:
```bash
npm install -g wigolo
# or use npx (default, no install needed)
```

## Quick Start

### Fetch URLs into Documents

```python
from wigolo_llamaindex import WigoloMcpClient, WigoloWebReader

async def main():
    async with WigoloMcpClient() as client:
        reader = WigoloWebReader(client=client)
        docs = await reader.aload_data(urls=[
            "https://docs.python.org/3/library/asyncio.html",
            "https://docs.python.org/3/library/typing.html",
        ])
        for doc in docs:
            print(f"{doc.metadata['title']}: {len(doc.text)} chars")
```

### Search and Load

```python
from wigolo_llamaindex import WigoloMcpClient, WigoloSearchReader

async def main():
    async with WigoloMcpClient() as client:
        reader = WigoloSearchReader(
            client=client,
            max_results=5,
            include_domains=["docs.python.org"],
        )
        docs = await reader.aload_data(query="Python asyncio best practices")
```

### Use in a RAG Pipeline

```python
from llama_index.core import VectorStoreIndex
from wigolo_llamaindex import WigoloMcpClient, WigoloWebReader

async def build_index():
    async with WigoloMcpClient() as client:
        reader = WigoloWebReader(client=client)
        docs = await reader.aload_data(urls=[
            "https://react.dev/learn",
            "https://react.dev/reference/react",
        ])
        index = VectorStoreIndex.from_documents(docs)
        return index
```

### Section Extraction

```python
reader = WigoloWebReader(client=client, section="API Reference")
docs = await reader.aload_data(urls=["https://docs.example.com/api"])
# Only loads the "API Reference" section from each page
```

## API Reference

### WigoloMcpClient

Async MCP client that communicates with wigolo via subprocess.

- `command` — executable (default: `"npx"`)
- `args` — arguments (default: `["wigolo"]`)
- `timeout` — seconds (default: `30.0`)
- Supports `async with` context manager

### WigoloWebReader

Fetches URLs and converts to Documents.

- `client` — `WigoloMcpClient` instance
- `render_js` — "auto", "always", "never" (default: "auto")
- `section` — extract only a specific heading section
- `max_chars` — cap content length
- `use_auth` — use stored browser session

Methods:
- `load_data(urls: list[str]) -> list[Document]`
- `aload_data(urls: list[str]) -> list[Document]`
- `lazy_load_data(urls: list[str]) -> Iterator[Document]`

### WigoloSearchReader

Searches the web and converts results to Documents.

- `client` — `WigoloMcpClient` instance
- `max_results` — default 5
- `include_domains` / `exclude_domains` — domain filters
- `category` — "general", "code", "docs", "news", "papers"

Methods:
- `load_data(query: str) -> list[Document]`
- `aload_data(query: str) -> list[Document]`
- `lazy_load_data(query: str) -> Iterator[Document]`

## License

[GNU AGPL-3.0-only](../../LICENSE) — same license as the parent wigolo project. Full terms in the root `LICENSE`.
