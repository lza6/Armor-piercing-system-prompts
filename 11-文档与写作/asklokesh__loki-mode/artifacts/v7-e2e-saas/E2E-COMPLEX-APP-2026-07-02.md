# Autonomi-SaaS complex-app E2E (v7.117) - 2026-07-02

The founder's named validation: a GENUINELY complex app (real HTTP API + PostgreSQL + Redis
cache) built through the FULL autonomi-saas product path, then ground-truthed. Every fact below
is observed from logs/artifacts, not fabricated. n=1.

## Stack (all real, verified via HTTP)
web :5180 (200), BFF :8788 (remapped from 8787 - an lm-gtm-agent claude-sidecar shadowed 8787;
a DIFFERENT project, NOT killed; remap is reversible via BFF_HOST_PORT), worker, postgres/redis/
minio (compose, healthy), host loki engine :57374 running v7.117.0 (latest, installed this session).
Observability dashboard :58080 kept up.

## The build (real product path: web -> BFF -> worker -> engine.startBuild)
Spec: Task Management REST API, Express + PostgreSQL persistence (migration) + Redis cache-aside,
10 acceptance criteria incl. cache-hit-vs-miss AND cache optional-degrade (Redis-down still works).
- buildId e25fc013-91eb-4c85-aa67-7fa3106fcfdf, projectId 0d722695, runId run-20260702165352.
- Worker: job_received -> running -> run_id_harvested (real product path confirmed).
- The engine built a genuinely MODULAR app: src/db/{pool,tasksRepo,migrate}.js, src/cache/redisClient.js,
  src/middleware/validateTask.js, src/routes/tasks.js, migrations/001_create_tasks.sql, 3 test files
  incl. cache-degrade.test.js. 2 commits (implement + a test-glob fix). Cost ~$5.38, ~25min, sonnet-5+opus.

## CORRECTION (2026-07-02, later same day): the original RESULT below was WRONG on the central facts

The first version of this artifact (preserved below, struck through) claimed "ACT iterations=1, the
loop did NOT run a recovery iteration, single-pass ceiling." A follow-up root-cause pass against the
AUTHORITATIVE logs and DB (not the inner-agent stream, not self-report) proved that claim FALSE.
Per the no-fabrication rule, the corrected, fully-observed account is here at the top; the original
is retained (struck) so the error and its correction are both on the record.

### What ACTUALLY happened (every fact observed from RARV loop log / signals / autonomi-saas DB)
- The RARV loop ran **2 iterations, not 1** (`.loki/logs/background-20260702-125346.log` shows
  `iteration="1"` and `iteration="2"`; `.loki/autonomy-state.json` iterationCount=2).
  - Iter 1: agent built the app; post-iteration deterministic gate ran -> "Test suite gate: node-test
    FAILED" (env: no DB running) -> "Completion claim rejected: code review BLOCKED" -> the loop logged
    "Starting next iteration..." It DID iterate. The gate chain (run.sh ~17870+) correctly falls
    through and keeps iterating on a rejected claim.
  - Iter 2: agent fixed the two gate failures; wrote a fresh `.loki/signals/COMPLETION_REQUESTED` at
    17:24:53 UTC. (The `.loki/signals/TESTS_FAILED` from iter 1 at 17:09:13 was never cleared -- but
    it is MOOT, see below.)
- **v7.114 rank-8/rank-15 are EXONERATED on the convergence-regression charge.** The engine never
  wrote a terminal (`autonomy-state.json` status=running, no proof.json). It did not prematurely
  terminate; it did not give up. There is no single-pass ceiling -- the loop iterated and self-healed.
- **No proof.json is the trust core working CORRECTLY, not a bug.** The post-iteration-2 deterministic
  gate never ran because the engine process (PID 39264) was manually SIGKILLed at ~17:25 (mid iter-2
  wrap-up). No independent verification -> no certification. Correct by design.
- **The app the engine produced INDEPENDENTLY PASSES** (behavioral proof, NOT the agent's prose):
  re-ran the built test suite in a CLEAN env against a freshly-created compose Postgres DB +
  compose Redis: **18/18 pass, exit 0**. Then a STRONG cache-aside proof: created a task (row
  confirmed directly in Postgres), warmed the cache, DELETED the Postgres row directly, GET again
  -> still returned the task -> served from Redis. Real PG persistence + real cache-aside, verified.

### The REAL finding (reframed, HIGH): the WORKER reported a TIMEOUT as a definitive `failed`
Authoritative autonomi-saas `builds` row (observed):
`status=failed, created_at=16:53:45, started_at=16:53:46, finished_at=17:13:51`, `proofs=0`.
- The worker's proof-wait budget is `DEFAULT_MAX_ATTEMPTS=600 x DEFAULT_POLL_INTERVAL_MS=2000ms`
  = **~20 min** (worker/src/build/state-machine.ts:151-152). Build started 16:53:46; the worker
  gave up and wrote `failed` at **17:13:51** = exactly the ~20-min budget.
- The engine was STILL LEGITIMATELY WORKING: its iter-2 COMPLETION_REQUESTED came at **17:24:53**,
  **~11 min AFTER the worker had already declared the build `failed`**. The worker did not die (it
  wrote a clean `finished_at` + `failed`); it timed out on a still-running build.
- **On-thesis defect:** hitting `maxAttempts` routes through `failWith()` (state-machine.ts:452-455
  -> :366 `transition("failed")` + `setBuildStatus("failed")`), so a budget-exceeded TIMEOUT is
  written as a **definitive negative (`failed`)**. The honest terminal for "ran out of wait budget,
  no verdict yet" is `inconclusive`/`timed_out`, NOT `failed`. This is the exact INVERSE of the
  anti-fake-green principle: a false-NEGATIVE (a working build reported as failed) is arguably a
  worse adoption killer than a false-positive. The engine's own code already distinguishes these
  (state-machine.ts:165,199-201 "inconclusive -> failed ... Genuinely unknown only ... NOT a
  fake-green"); the timeout path bypasses that distinction.

### Honest scope of this corrected result
This run CANNOT tell us whether the engine would have emitted a PASSING proof, because it was
manually killed before its post-iter-2 gate ran. It CAN tell us: (a) the engine iterated and
self-healed (no single-pass ceiling; v7.114 exonerated); (b) the app it produced independently
passes 18/18 + a real cache-aside proof; (c) the worker reported a still-running build as a
definitive `failed` after a ~20-min timeout, which is a false-negative trust defect in the SaaS
worker (a DIFFERENT component + class than originally filed).

<details><summary>ORIGINAL (WRONG) result section -- retained for the record, do not trust</summary>

~~## RESULT: build reported FAILED and that was CORRECT~~
~~1. inner agent OVER-CLAIMED "18 tests all passing"; 2. deterministic gate re-ran clean -> tests~~
~~FAILED -> TESTS_FAILED; 3. no proof.json; 4. worker polled 600 times (~10min) and reported failed.~~
~~Table claimed: ACT iterations=1, Self-heal rounds=0 (no recovery iteration), Deliverable works=NO.~~
~~Finding 2 claimed a "pre-existing single-pass convergence gap (run.sh:9560-9575, v7.48.0)".~~
~~ALL OF THE ABOVE IS FACTUALLY WRONG per the corrected account above: there were 2 iterations, the~~
~~loop self-healed, the app works (18/18 clean-env), and the failure was a WORKER timeout, not an~~
~~engine gate. The git-blame of 9560-9575 was the wrong lines (that path WRITES the signal; it does~~
~~not decide iterate-vs-terminate).~~
</details>

## Measured vs OUR prior baseline (CORRECTED; NOT vs competitors - head-to-head is founder-gated #8)
| Metric | Prior (URL-shortener, simple) | This (Task API + PG + Redis, complex) |
|---|---|---|
| Wall-clock (engine, to manual kill) | ~7 min | ~31 min (killed mid iter-2 wrap-up, did NOT finish naturally) |
| RARV iterations | 1 | 2 (iter-1 tests failed on env -> iter-2 self-healed) |
| Self-heal | n/a | YES (iter-2 resolved both iter-1 gate failures) |
| Engine terminal | completion_promise_fulfilled | none written (status=running; manually SIGKILLed) |
| App independently works | YES (ground-truthed) | YES (18/18 clean-env + delete-row-then-GET cache proof) |
| SaaS build row outcome | (n/a, simple ran direct) | `failed` -- but a WORKER ~20-min timeout false-negative, not an engine failure |
Honest: the complex app did NOT one-shot (iter-1 env test failure); it self-healed by iter-2 and the
deliverable independently passes. The "failed" the founder would have seen is a worker-side timeout
false-negative, not the engine failing to converge.

## Findings filed (CORRECTED; severities honest)
1. POSITIVE, sub-question #90 CLOSED-as-assessed (no code change): iter-1's gate correctly caught the
   env-dependent test failure and rejected the completion claim. #90 asked whether the v7.114 rank-8
   build_prompt "verify once" line (run.sh:14201) induced the over-claim. ASSESSMENT: no. Read plainly,
   the line names "the completion gates (tests, checklist, evidence) are the authority on done" and
   tells the agent to DEFER to the gate (which runs in the clean env) rather than re-verify redundantly
   -- it points the agent TOWARD the clean-env gate, not away. The "could mean don't re-run in a clean
   env" reading is strained and, on n=1, is not evidence. And the gate DID catch it + the loop recovered.
   Disposition: the system worked; the causal claim is unprovable and moot on n=1. NO prompt reword
   shipped: it is unmeasurable before/after on one run (violates the "measurable per commit" directive),
   parity-locked with loki-ts/src/runner/build_prompt.ts (2-file, whole-fleet blast radius), and the
   fleet converges fine. Skipping it is the disciplined call.
2. **REFRAMED -> #91 (HIGH), now a WORKER false-negative, NOT an engine single-pass ceiling:** the
   autonomi-saas worker writes a ~20-min proof-wait TIMEOUT as a definitive `failed` (state-machine.ts
   maxAttempts -> failWith). A still-running, self-healing, ultimately-working complex build was
   reported `failed` 11 min before the engine even claimed completion. Fix class: (a) map
   budget-exceeded to `inconclusive`/`timed_out`, not `failed` (honesty, on-thesis); (b) make the
   budget adaptive to build activity (a build actively writing iterations/heartbeat should extend the
   wait) rather than a fixed ceiling any complex build will blow. The ORIGINAL "engine single-pass
   convergence gap / run.sh:9560-9575 / v7.48.0" framing was WRONG (wrong lines, wrong component,
   wrong conclusion) and is withdrawn.
3. POST-SESSION HANG -> #92 (unchanged): the loki-run wrapper PID stayed alive, ignored SIGTERM
   (needed SIGKILL). Distinct from #91.

## Honest scope
Measures OUR run only, n=1. No competitor was run (never fabricate a head-to-head). The
"self-healing + error monitoring" build slice is grounded in finding 2 (the real recovery gap this
run exposed), not speculative.
