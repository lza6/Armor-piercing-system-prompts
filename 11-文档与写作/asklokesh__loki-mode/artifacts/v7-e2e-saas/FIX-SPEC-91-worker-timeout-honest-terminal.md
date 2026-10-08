# Fix spec: #91 -- worker reports a proof-wait TIMEOUT as a definitive `failed` (false-negative)

> STATUS: **SHIPPED** (2026-07-02, committed `9d78f46` in autonomi-saas, NOT pushed - founder pushes
> private repos). Scope A (honest `timed_out`/`inconclusive` terminal across worker+BFF+web, 26 files)
> is DONE and verified: worker 115/115, BFF 187->188, web 309->325, all tsc clean, plus a COMPOSED
> Postgres calibration (timed_out accepted + finished_at stamped + coherentStatus serves un-demoted).
> A consumer the web lane missed (LivePreview idle-fallthrough) was found via a completeness grep and
> fixed with its own RED/GREEN test. Scope B (adaptive/heartbeat proof-wait budget) remains DEFERRED
> as a separate design fork. This spec is retained as the design record.

Component: **autonomi-saas** (private), NOT loki-mode engine. Root cause fully observed (see
E2E-COMPLEX-APP-2026-07-02.md CORRECTION block).

## Confirmed root cause (observed, not inferred)
Worker container log (authoritative):
```
17:13:51.370 build.transition to:"failed" detail:"...produced no proof within 600 polls..."
17:13:51.381 worker.job_done finalState:"failed"
```
- The WORKER `failWith`-on-maxAttempts wrote `failed` (state-machine.ts:452-455 -> :366), at exactly
  start+20min (DEFAULT_MAX_ATTEMPTS=600 x DEFAULT_POLL_INTERVAL_MS=2000, :151-152).
- The BFF stuck-sweep (STUCK_BUILD_TIMEOUT_MS=20min, app.ts:1459+) did NOT fire (it runs every 5min
  and SPARES recently-active builds; this build was active). Discriminated by log fingerprint.
- The engine was still legitimately on iteration 2; its COMPLETION_REQUESTED came at 17:24:53, 11 min
  AFTER the worker had already declared `failed`. The app independently passes (18/18 clean-env + a
  delete-PG-row-then-GET cache-aside proof).

## The defect (on-thesis)
A budget-exceeded TIMEOUT is written as a definitive negative (`failed`) when the honest terminal is
`inconclusive` / `timed_out`. This is the INVERSE of anti-fake-green: a working build reported failed
= a false-NEGATIVE, a worse adoption killer than a false-positive. The engine's own normalization
code already distinguishes these (state-machine.ts:165,199-201: "inconclusive -> failed ... Genuinely
unknown only ... NOT a fake-green"); the maxAttempts path bypasses that distinction.

## Fix (scope A -- SHIP now, council-gated per SaaS discipline)
1. Add a `timed_out` terminal to `BuildState` (state-machine.ts:113-120). `builds.status` is free TEXT
   with NO CHECK constraint (infra/postgres/init/002_per_build_state.sql:26-27) -> NO DDL migration.
2. On `attempts >= maxAttempts`, do NOT call `failWith` (which forces `failed`). Emit a distinct
   `timed_out` terminal with `proofVerdict:"inconclusive"` and the same honest detail string. Keep the
   sandbox teardown + cost persistence that `failWith` does (see the :940 comment: "EVERY terminal ...
   do NOT route the non-pass/inconclusive cases through failWith").
3. Apply the SAME honest terminal to the BFF stuck-sweep auto-fail path (app.ts sweepStuckBuilds): a
   genuinely-dead no-verdict build should be `timed_out`/`inconclusive`, not `failed`. Fixing only the
   worker leaves the sweep able to false-`fail` a different stuck build -- scope to ALL no-verdict ->
   terminal paths or it is incomplete by construction.
4. Every `finalState` consumer: BFF `coherentStatus` (app.ts:2370-2396 -- a `timed_out` build with no
   proof must render as timed_out/inconclusive, never a fake pass and never a hard `failed`); the web
   UI build-status rendering (distinct copy: "Timed out -- no verdict yet", offer retry, NOT a red
   FAILED); build-tail stream selection (build-tail.ts:108,114).
5. Tests (RED first): a state-machine test that maxAttempts -> finalState `timed_out` +
   proofVerdict `inconclusive` (not `failed`); a sweep test that a no-verdict stuck build -> timed_out;
   a coherentStatus test that timed_out never becomes `passed`. Mirror existing state-machine.test.ts
   patterns (assert.equal(res.finalState, ...)).

## Fix (scope B -- DEFER, genuine design fork, own slice)
Adaptive / heartbeat proof-wait budget: a build actively writing iterations (events.jsonl / signals
mtime advancing) should EXTEND the wait instead of dying at a fixed 20-min ceiling that any complex
multi-iteration build blows. Product decision (how long is too long; cost ceiling; founder-visible
"still working" state) -> not bundled here.

## Verification (end-to-end, when scope A ships)
- Re-run the complex-app E2E; with the honest terminal, a build still running at 20 min surfaces as
  `timed_out`/inconclusive (retryable), never a red `failed`.
- Confirm no fake-green regression: a genuine BLOCK/REJECT proof still -> failed; a real pass -> passed.
- Full worker + BFF `bun test` green; council 3/3 (trust-core-adjacent -> full council, not lightweight).
