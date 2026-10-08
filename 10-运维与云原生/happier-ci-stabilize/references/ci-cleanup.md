# CI and test cleanup without weakening ship evidence

## Keep three explicit profiles

| Profile | Default use | Contents |
| --- | --- | --- |
| `fast` | pull requests and ordinary pushes | affected contracts, typechecks, units/integration, and essential smoke |
| `release` | candidate and nightly release validation | artifact identity, install/binary smoke, and risk-selected continuity; never a second source-CI run |
| `deep` | scheduled/manual certification and stable-release evidence | extended databases, platforms, compatibility, stress, and other expensive coverage |

Use one canonical change classifier and one final result aggregator. A profile is a maintained policy, not another copied workflow. A full/deep run remains available, but unrelated paths should not select it by default.

## Evidence-led cleanup targets

Consolidate or remove a test/check when current evidence shows it is one of:

- a duplicate of the same observable contract at the same boundary;
- a structural assertion over YAML/source ordering that does not protect a security or operational invariant;
- wording, logging, formatting, incidental call-count, or constant-value policing with no public contract;
- a suite-local mock that duplicates a canonical testkit boundary;
- an obsolete compatibility case with no released producer/consumer or reachable migration direction;
- an aggregator that reruns work instead of aggregating prior outcomes;
- a timeout permutation that does not exercise a materially different lifecycle.

Strengthen or relocate the canonical owner-level test before deleting overlapping coverage. Preserve one discriminating test for each real happy, failure, cancellation/recovery, compatibility, security, and platform contract selected by risk.

## Repeat-offender cleanup

If the same family escapes twice:

1. stop patching individual assertions;
2. identify the shared owner or harness that allowed drift;
3. extend/consolidate that owner;
4. migrate overlapping local variants;
5. add one test proving the shared harness reaches the deciding branch;
6. run a broader lane that can expose state leakage or cleanup failures.

Mocks represent external boundaries, not internal policy. A fake protocol server must evolve with the protocol methods it claims to implement. Prefer typed fixtures/builders and one boundary harness over repeated inline response objects.

For UI tests, inventory repeated `vi.mock(...)` targets before adding another local mock. High raw counts are a drift signal, not an instruction to rewrite everything: migrate a family when a reproduced failure shows its local variant is stale, when the same fixture changes in lockstep, or when a canonical testkit already owns the boundary. Keep real reducers, providers, registries, and internal orchestration active beneath the mocked system boundary.

When a real third-party boundary retains initial props or other mount-time state, make the shared boundary fake retain that state too. Then exercise every live rendering topology that changes ownership (for example, a component-owned virtualizer and a parent-provided virtualizer). A test that only asserts which wrapper exists can stay green while the sibling topology renders stale data; update the input and assert the user-visible output or retained boundary state changes.

A green lane should not emit React `act(...)`, unhandled rejection, leaked-handle, or open-resource warnings. Await mocked asynchronous boundary settlement inside the owning test helper/`act`, and fix lifecycle cleanup at its owner. Do not globally suppress warnings: they often reveal that a test asserted before the product-visible state settled.

## Timeout policy

Do not raise timeouts globally. First classify:

- deterministic assertion/configuration failure: fix the contract;
- deadlock/leaked handle: fix lifecycle ownership;
- resource exhaustion or runner stall: fix resource isolation or runner selection;
- legitimate measured operation near the limit: raise only the owning timeout with a success-runtime baseline and bounded ceiling;
- external asynchronous service: use the service's supported polling/recovery semantics and preserve its existing submitted work.

A larger timeout is valid only when the operation is making observable progress or measured successful executions need the budget. It is not a substitute for progress evidence.

Before treating a local timeout as product evidence, check whether the development host is saturated by other agents or suites. A shared VM with runnable-process load materially above its CPU allocation cannot establish a normal success-runtime baseline. Preserve other agents' processes, rerun the smallest owner test when capacity is healthy, or use the exact custom hosted lane; do not encode shared-host contention into repository-wide timeouts.

## Workflow simplification

- Keep one canonical command per lane and let local/manual/automatic workflows call it.
- Keep runner-pool selection as an input to the reusable workflow. Blacksmith is a manual accelerator for approved non-secret Linux lanes, not a fork of CI; the same job graph and commands must continue to work on GitHub-hosted runners.
- Use matrices only for real platform/configuration differences.
- Size a shard matrix from the canonical test inventory and recent elapsed-time evidence. Verify every configured shard is non-empty with the runner's list mode, and remove empty shards rather than paying a full checkout/install/browser setup for no coverage. More shards are not automatically faster when runner capacity is lower than the matrix or when the framework partitions by test count instead of measured duration.
- Keep result aggregators tiny and free of dependency installation.
- Keep independent source-test jobs independent and let the terminal result job aggregate their recorded conclusions. A failing unit, typecheck, or integration lane must not prevent unrelated E2E shards from reporting, but publication and trust-dependent jobs still require their real prerequisites.
- Inside a matrix, run cross-cutting package guards once unless each matrix part truly owns different input. Give the other parts a canonical narrower command rather than paying twice for an identical guard. Remove explicit test processes whose selected path/pattern matches zero tests; a successful zero-test invocation is setup cost, not coverage.
- Reuse immutable prepared inputs when that avoids repeated lifecycle installs without turning caches into evidence.
- Cache only reproducible inputs; never let a cache become the authority for generated-output freshness.
- Do not add retries around release mutations unless the operation is proven idempotent or state reconciliation precedes retry.

Measure cleanup by fewer competing owners, fewer repeated fixtures, faster time to the first deciding failure, and fewer expensive full reruns—not by raw test-count reduction.

When a live E2E surface is stale, use boundary evidence in order: confirm the producer state changed, confirm a fresh transport request and response occurred, then inspect the active DOM/rendered bytes. If the producer and transport are fresh but the active DOM is old, correct and test the renderer boundary; do not add more polling, invalidate unrelated stores, or raise the assertion timeout.

For virtualized browser surfaces, `locator.isVisible()` proves CSS visibility, not viewport intersection, and a row locator may resolve to a different recycled node between assertions. Assert viewport intersection explicitly, read related identity/position facts atomically when they must describe one render, and anchor selectors in semantic content. Treat framework- or library-generated DOM-id prefixes as implementation detail; assert the stable owner-defined suffix or data contract instead of the generated prefix.
