# Test Coverage Plan

Routing chain: read `process/context/tests/all-tests.md` first to select the runner.

## Blast Radius Areas

- packages/api: new route
- packages/ui: new component

## Per-Area Test Plan

| Tier | Case | Command/Steps | Proves | Does NOT prove |
| --- | --- | --- | --- | --- |
| Fully-automated | packages/api route 200 | `pnpm --filter @your-app/api test` exits 0 | shape correct | load behavior |

## Gap Resolution Options

| Gap | Resolution options |
| --- | --- |
| Load behavior under concurrency | A) Write k6 test. B) Set up load infra. C) Accept as known-gap — out of scope. D) Backlog artifact. |
