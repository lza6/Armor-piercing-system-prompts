# Vercel AI SDK tools

[`wigolo-vercel-ai-sdk`](https://www.npmjs.com/package/wigolo-vercel-ai-sdk)
wraps wigolo as ready-to-register [Vercel AI SDK](https://ai-sdk.dev) tools.
`WigoloMcpClient` spawns and talks to a wigolo instance over the MCP protocol
on stdio (`npx wigolo` by default — no server to run), and `createWigoloTools`
hands you the tool set any AI SDK model can call.

## Run it

```bash
npm install
npm run demo     # tsc && node dist/tools.js
```

Requires node >= 20. No LLM key needed for this demo — it connects, registers
the tools, prints them, and disconnects.

## What you'll see (real output)

```text
connected to wigolo over MCP (stdio)

registered 6 tools:

- webSearch(query, max_results, include_domains, exclude_domains, ...)
    Search the web for information on any topic.
- webFetch(url, section, render_js, max_chars, ...)
    Fetch a specific web page and return clean markdown content.
- webCrawl(url, strategy, max_depth, max_pages, ...)
    Crawl a website starting from a URL.
- findSimilar(url, text, max_results)
    Find pages semantically similar to a given URL or text from the local cache.
- research(topic, max_depth, max_sources)
    Deep multi-step research on a topic.
- agent(goal, max_steps)
    Autonomous web agent that breaks down complex goals into search/fetch/extract steps.

disconnected — wigolo subprocess stopped
```

## Using the tools with a model

With any AI SDK provider wired up, giving the model live web access is one
property on `generateText` / `streamText`:

```ts
import { generateText } from 'ai';
import { anthropic } from '@ai-sdk/anthropic'; // or any AI SDK provider
import { WigoloMcpClient, createWigoloTools } from 'wigolo-vercel-ai-sdk';

const client = new WigoloMcpClient();
await client.connect();

const { text } = await generateText({
  model: anthropic('claude-sonnet-4-5'),
  tools: createWigoloTools(client),
  maxSteps: 5,
  prompt: 'What changed in the latest TypeScript release? Cite sources.',
});

await client.disconnect();
```

The model decides when to call `webSearch` / `webFetch` / `research`; wigolo
does the gathering locally — ML-reranked search, clean markdown extraction, a
persistent knowledge cache — and returns structured, citable results the model
can quote. You can also cherry-pick single tools (`createWebSearchTool(client)`
et al.) instead of registering all six.

## Notes

- The subprocess is plain stdio MCP — the same wigolo that Claude Code, Cursor
  and friends register directly. This package just adapts it to the AI SDK's
  `tool()` interface with zod-typed parameters.
- Point the client at a specific wigolo entry with
  `new WigoloMcpClient({ command, args })` (the demo reads
  `WIGOLO_MCP_COMMAND` / `WIGOLO_MCP_ARGS` for this).
- Prefer HTTP instead of a subprocess? Use [rest-curl](../rest-curl/) or the
  typed SDKs: [sdk-typescript-research](../sdk-typescript-research/).
