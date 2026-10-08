# SDKs & integrations

Thin clients over the [REST API](./rest-api.md) for TypeScript and Python, plus ready-made tool wrappers for four agent frameworks. Thin is the design: no client-side retries, reranking, or interpretation — the server does the intelligence, the SDK gets requests there and typed responses back. All packages are published at version 0.2.0 and track the server's OpenAPI contract.

| Package | Registry | What it is |
| --- | --- | --- |
| `wigolo-sdk` | npm | TypeScript REST client + embedded local mode |
| `wigolo` | PyPI | Python REST client (sync + async) + embedded local mode |
| `wigolo-langchain` | PyPI | LangChain retriever + tools |
| `wigolo-crewai` | PyPI | CrewAI tool set |
| `wigolo-llamaindex` | PyPI | LlamaIndex readers |
| `wigolo-vercel-ai-sdk` | npm | Vercel AI SDK tool factories |

## TypeScript — `wigolo-sdk`

```bash
npm i wigolo-sdk
```

```ts
import { WigoloClient } from 'wigolo-sdk';

// baseUrl defaults to http://127.0.0.1:3333 (or WIGOLO_BASE_URL);
// token defaults to WIGOLO_API_TOKEN — only needed when the server sets one.
const client = new WigoloClient();

const res = await client.search({ query: 'local-first web search', max_results: 3 });
```

One method per tool — `search`, `fetch`, `crawl`, `cache`, `extract`, `findSimilar`, `research`, `agent`, `diff`, `watch` — plus `health()`, `listTools()`, `openapi()`. Method names are camelCase; request/response **fields are the daemon's snake_case wire names** (`max_results`, `response_time_ms`). The core entry point imports no Node built-ins, so it runs on browsers, edge runtimes, Deno, and Node (≥18).

No daemon running? The Node-only `wigolo-sdk/local` subpath reuses a healthy local daemon or spawns one:

```ts
import { createLocalClient } from 'wigolo-sdk/local';

const { client, owned, close } = await createLocalClient();
try {
  const page = await client.fetch({ url: 'https://example.com' });
} finally {
  await close(); // stops the daemon only if this call spawned it
}
```

Per-call overrides: `client.research({ question }, { timeoutMs: 120_000, signal })`. Errors are typed: `WigoloApiError` (non-2xx, with `status`, `error_reason`, `retryAfter`) and `WigoloConnectionError`; a degraded 2xx never throws — inspect in-body `warning`/`error`.

## Python — `wigolo`

```bash
pip install wigolo
```

```python
from wigolo import Client

with Client(base_url="http://127.0.0.1:3333") as client:
    res = client.search(query="local first web search", max_results=5)
    page = client.fetch(url="https://example.com")
```

Zero runtime dependencies (stdlib only), fully typed, and an `AsyncClient` with the identical surface. Embedded local mode probes or spawns a daemon for you:

```python
from wigolo import local_client

with local_client() as client:   # reuse a healthy daemon, or spawn one
    print(client.health())
```

Config resolution is explicit argument > env > default: `base_url`/`WIGOLO_BASE_URL`, `token`/`WIGOLO_API_TOKEN`, `local`/`WIGOLO_LOCAL=1`, `port`/`WIGOLO_LOCAL_PORT`, `command`/`WIGOLO_CLI`. Errors: `WigoloAPIError` / `WigoloConnectionError`.

**Embedded-mode security note (both SDKs):** `WIGOLO_CLI` names the binary the SDK will spawn — an exec-from-env vector. In untrusted environments, strip it and pass a trusted `command` explicitly; point it at the actual server binary, not an `npx` wrapper, so `close()` reaches the process that owns the port.

## Timeouts

Client per-tool defaults mirror the server's **unscaled** per-route deadlines (TS defaults: 75s search/cache/find_similar, 135s fetch/extract/watch, 315s crawl/research/agent; Python mirrors per-tool). If your server runs with `WIGOLO_SERVE_TIMEOUT_SCALE` above 1, raise the client timeout to match or the client may abort a request the server would still complete. Note `stream` on `research`/`agent` is accepted but inert over REST — responses return whole.

## LangChain — `wigolo-langchain`

```bash
pip install wigolo-langchain
```

```python
from wigolo_langchain import WigoloMcpClient, WigoloSearchRetriever

async with WigoloMcpClient() as client:      # subprocess MCP client (npx wigolo)
    retriever = WigoloSearchRetriever(client=client, max_results=5,
                                      include_domains=["docs.python.org"])
    docs = await retriever.ainvoke("Python asyncio tutorial")
```

Also `WigoloSearchTool` / `WigoloFetchTool` as LangChain `BaseTool`s for agents — errors come back as clean JSON strings instead of raising into the agent loop.

## CrewAI — `wigolo-crewai`

```bash
pip install 'wigolo-crewai[crewai]'
```

```python
from crewai import Agent
from wigolo_crewai import wigolo_tools

tools = wigolo_tools()   # spawns an embedded local daemon by default
researcher = Agent(role="Web Researcher", goal="Find well-sourced answers",
                   backstory="...", tools=tools)
```

Five tools: `wigolo_search`, `wigolo_fetch`, `wigolo_research`, `wigolo_crawl`, `wigolo_extract`. Target a running server with `wigolo_tools(base_url=..., token=..., local=False)`.

## LlamaIndex — `wigolo-llamaindex`

```bash
pip install wigolo-llamaindex
```

```python
from llama_index.core import VectorStoreIndex
from wigolo_llamaindex import WigoloMcpClient, WigoloWebReader

async with WigoloMcpClient() as client:
    reader = WigoloWebReader(client=client)
    docs = await reader.aload_data(urls=["https://react.dev/learn"])
    index = VectorStoreIndex.from_documents(docs)
```

`WigoloWebReader` turns URLs into `Document`s (with `render_js`, `section`, `max_chars`, `use_auth` knobs); `WigoloSearchReader` does the same from a search query.

## Vercel AI SDK — `wigolo-vercel-ai-sdk`

```bash
npm install wigolo-vercel-ai-sdk ai zod
```

```ts
import { generateText } from 'ai';
import { WigoloMcpClient, createWigoloTools } from 'wigolo-vercel-ai-sdk';

const client = new WigoloMcpClient();
await client.connect();

const { text } = await generateText({
  model: yourModel,
  tools: createWigoloTools(client),   // webSearch, webFetch, webCrawl, findSimilar, research, agent
  prompt: 'Find the latest React Server Components docs and summarize them',
});

await client.disconnect();
```

Individual factories (`createWebSearchTool`, `createWebFetchTool`, ...) exist when you want a subset.

[← Docs index](./README.md) · [Next: Self-hosting](./self-hosting.md)
