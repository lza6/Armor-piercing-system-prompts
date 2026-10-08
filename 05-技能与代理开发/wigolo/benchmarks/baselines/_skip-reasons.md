# Pre-v1 Baseline Capture — Skip Reasons

## Date
2026-05-20 (UTC)

## Environment
- Branch: `feat/v1-engine`
- Host: macOS (Darwin 24.6.0), Node v22.14.0
- Runner: agent sandbox

## What was attempted

All three pre-v1 baseline captures were attempted from the sandboxed agent runtime:

| Bench | Script | Output |
|-------|--------|--------|
| search | `npm run bench:search` | `search-pre-v1.txt` (failure log) |
| extraction | `npm run bench:extraction` | `extraction-pre-v1.txt` (failure log) |
| rerank | (no existing bench script; would need ad-hoc capture) | skipped — see below |

## Why each bench was skipped

### search + extraction
Both scripts invoke `tsx` (`tsx benchmarks/.../runner.ts`). `tsx` creates a Unix domain socket
in `$TMPDIR/tsx-501/<pid>.pipe` for its child-process IPC channel. The macOS sandbox in which
this agent runs denies the `listen()` syscall on AF_UNIX sockets, producing:

```
Error: listen EPERM: operation not permitted /tmp/claude-501/tsx-501/<pid>.pipe
    at Server.setupListenHandle [as _listen2] (node:net:1915:21)
```

This reproduces regardless of `TMPDIR` location (tested `/tmp/claude-501` and a workspace-local
`./.tmp-bench`) — it is a syscall-level restriction, not a filesystem-permission issue.

Falling back to `node --experimental-strip-types` (Node 22 native TS execution) also fails
because the bench runners import compiled `.js` paths (e.g. `'../../src/logger.js'`) which
tsx normally rewrites on the fly; native strip-types does not.

These benches are **expected to run cleanly on a fully-warmed host** (developer machine or
CI) where tsx can create its IPC pipe. Both runners use locally captured fixtures
(`benchmarks/search/fixtures/queries.json` + `relevance.json`, `benchmarks/extraction/fixtures/manifest.json`)
so they do **not** require live network — the only blocker is sandbox IPC.

**Follow-up:** capture both baselines on the dev host before the v1 merge gate. Drop the
resulting `search-pre-v1.txt` / `extraction-pre-v1.txt` (and any JSON reports the runners
emit under `benchmarks/{search,extraction}/output/`) into this directory.

### rerank
No pre-existing `npm run bench:rerank` script ships in `package.json`. Per Task 1.5 the
suggested path was an ad-hoc `tests/helpers/capture-rerank-baseline.ts` script wired through
the same legacy `RerankProvider` adapter. Since that script would also need to be invoked via
`tsx` (it imports TypeScript-only modules from `src/`), it would hit the same sandbox IPC
block. Rather than build infrastructure that can't be exercised here, the rerank baseline is
deferred to the same pre-merge dev-host capture window.

The rerank corpus that capture should use:
- Query: pick the first entry from `benchmarks/search/fixtures/queries.json`
- Corpus: synthesize ~50 docs from `tests/fixtures/reranker-tokenizer-corpus.json` (existing
  reranker fixture; contains varied content) or top-N results from a `bench:search` run
- Metric: nDCG@5, nDCG@10, mean latency over 5 trials
- Output path: `benchmarks/baselines/rerank-pre-v1.json`

## Status

PARTIAL — environmental constraint, not a code issue. Plan Task 1.5 explicitly tolerates
this case: "the goal is HAVING a baseline if possible, not blocking on environmental
constraints." Baselines must be captured before the v1 merge gate on a fully-warmed dev host.
