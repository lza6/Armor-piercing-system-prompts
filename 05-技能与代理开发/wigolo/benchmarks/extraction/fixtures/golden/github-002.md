# Issue #4521: Memory leak when using streaming responses with AbortController

**State:** Open | **Labels:** bug, memory-leak, streaming | **Milestone:** v3.2.0

**Opened by** @memory_hunter on Dec 12, 2024

---

## Description

We are experiencing a significant memory leak when using streaming responses in combination with `AbortController`. After approximately 10,000 aborted requests over 2 hours, the Node.js process grows from 120MB to 2.4GB RSS.

### Environment

- Node.js: v20.11.0
- Package version: 3.1.4
- OS: Ubuntu 22.04 (Docker container)
- Heap snapshot analysis: Retained `ReadableStream` objects not being GC'd

### Reproduction Steps

1. Start the server with streaming enabled
2. Send requests with an `AbortController` that aborts after 100ms
3. Observe memory growth over time

### Minimal Reproduction

```typescript
import { createServer } from './server';

const server = createServer({ streaming: true });

async function bombardWithAborts() {
  for (let i = 0; i < 10000; i++) {
    const controller = new AbortController();

    // Abort after 100ms
    setTimeout(() => controller.abort(), 100);

    try {
      const response = await server.fetch('/api/stream', {
        signal: controller.signal,
      });

      // Read some chunks then abort
      const reader = response.body?.getReader();
      if (reader) {
        await reader.read(); // read one chunk
        await reader.cancel();
      }
    } catch (e) {
      // Expected: AbortError
    }

    if (i % 1000 === 0) {
      const mem = process.memoryUsage();
      console.log(`Iteration ${i}: RSS=${(mem.rss / 1024 / 1024).toFixed(0)}MB, Heap=${(mem.heapUsed / 1024 / 1024).toFixed(0)}MB`);
    }
  }
}

bombardWithAborts();
```

### Memory Growth Log

```
Iteration 0:    RSS=120MB, Heap=45MB
Iteration 1000: RSS=340MB, Heap=180MB
Iteration 2000: RSS=580MB, Heap=310MB
Iteration 3000: RSS=810MB, Heap=455MB
Iteration 5000: RSS=1240MB, Heap=720MB
Iteration 8000: RSS=1890MB, Heap=1100MB
Iteration 10000: RSS=2400MB, Heap=1450MB
```

### Heap Snapshot Analysis

Top retained objects after 10,000 iterations:

| Object Type | Count | Retained Size |
|------------|-------|---------------|
| ReadableStream | 9,847 | 890 MB |
| TransformStream | 9,847 | 340 MB |
| AbortSignal | 9,832 | 120 MB |
| TextEncoder | 9,847 | 45 MB |
| Uint8Array | 48,291 | 55 MB |

---

## Comments

### @core_maintainer - Dec 12, 2024

Thanks for the detailed report. I can reproduce this. The issue is in `StreamHandler.createResponse()` where we attach an abort listener but never remove it when the stream completes or errors:

```typescript
// Current code (leaky)
signal.addEventListener('abort', () => {
  stream.cancel();
});

// Fix: remove listener on stream close
const onAbort = () => stream.cancel();
signal.addEventListener('abort', onAbort);
stream.on('close', () => {
  signal.removeEventListener('abort', onAbort);
});
```

The `AbortSignal` holds a reference to the callback, which holds a reference to the stream, which holds the response buffers. None of them get GC'd.

### @memory_hunter - Dec 13, 2024

Confirmed that the suggested fix resolves the leak. After applying the patch locally:

```
Iteration 0:    RSS=120MB, Heap=45MB
Iteration 5000: RSS=145MB, Heap=52MB
Iteration 10000: RSS=148MB, Heap=54MB
```

Memory stays flat. Would love to see this in a patch release.

### @core_maintainer - Dec 13, 2024

PR incoming. Also adding:
1. A `WeakRef` wrapper for the stream reference in the abort handler
2. A test that monitors memory growth during aborted streaming
3. Documentation note about proper stream cleanup

### @devops_sarah - Dec 14, 2024

This is affecting us in production. We have a workaround using a periodic `gc()` call but it causes latency spikes. Would really appreciate a quick patch release.

### @core_maintainer - Dec 14, 2024

Fix is in #4525. Will be in v3.1.5 which we'll cut today.

---

## Referenced Pull Requests

- #4525 - fix: remove abort listeners on stream close (merged)
- #4527 - test: add memory leak regression test for streaming (merged)
- #4528 - docs: add streaming cleanup best practices (open)

## Related Issues

- #4102 - High memory usage under concurrent connections
- #4389 - Stream responses not cleaned up on client disconnect
- #3891 - AbortController memory management improvements
