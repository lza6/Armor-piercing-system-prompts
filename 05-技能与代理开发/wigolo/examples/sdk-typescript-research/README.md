# TypeScript SDK — research in ~20 lines

[`wigolo-sdk`](https://www.npmjs.com/package/wigolo-sdk) is a thin, zero-dependency
TypeScript client for the wigolo daemon's REST API. Its `wigolo-sdk/local`
subpath adds **embedded local mode**: `createLocalClient()` reuses a daemon
that's already listening — or spawns `wigolo serve` for you on a loopback port
and stops it when you're done. No setup step, nothing left running.

## Run it

```bash
npm install
node research.mjs
node research.mjs "your own question here"
```

Requires node >= 20 and a wigolo CLI the SDK can spawn (`npm i -g wigolo`, or
set `WIGOLO_CLI` to a path / JSON argv array).

## What you'll see (real output, trimmed)

```text
daemon: spawned for this run (stopped on exit)

question: how does http caching interact with service workers
topics:   how does http caching interact with service workers | ... tutorial guide | ... best practices

top highlights (12 total):

- On the other side, a service worker has a specific task and it is to act as a
  proxy between our web application and the network. It can intercept http
  requests and serve the responses from the network or from a local cache, ...
  source: https://dev.to/paco_ita/service-workers-and-caching-strategies-explained-step-3-m4f

- *   Service Worker Caching and HTTP Caching serve different purposes and use cases.
  *   Service Worker Caching doesn't need to be consistent with HTTP Caching expiry.
  *   HTTP Caching still plays an important role in the cache layers, ...
  source: https://dev.to/jonchen/service-worker-caching-and-http-caching-p82

- Service workers are a powerful browser API that sits between your web
  application and the network, acting as a programmable network proxy. ...
  source: https://jsguides.dev/tutorials/browser-apis/browser-service-workers/
```

## The whole client, in one glance

```js
import { createLocalClient } from 'wigolo-sdk/local';

const { client, owned, close } = await createLocalClient();
try {
  const res = await client.research({ question: '...', depth: 'quick' });
  // res.brief: { topics, highlights, key_findings, sections, ... }
} finally {
  await close(); // stops the daemon only if this call spawned it
}
```

- One method per tool: `search`, `fetch`, `crawl`, `cache`, `extract`,
  `findSimilar`, `research`, `agent`, `diff`, `watch` — plus `health()`,
  `listTools()`, `openapi()`.
- Already running a daemon (`wigolo serve`)? Skip local mode and construct
  `new WigoloClient({ baseUrl })` from the main `wigolo-sdk` entry — it is
  edge-safe (no node built-ins).
- Responses are the daemon's wire shapes, verbatim. The SDK adds no retries,
  no re-ranking, no surprises.

Full client docs: the [wigolo-sdk README](https://www.npmjs.com/package/wigolo-sdk).
Python version of this example: [sdk-python-agent](../sdk-python-agent/).
