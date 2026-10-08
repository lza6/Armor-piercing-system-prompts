# Autonomi-SaaS E2E real-build run log (2026-07-02)

REAL browser-driven end-to-end test of the autonomi-saas full stack building an app via
the loki engine. No stubs. Every fact below is observed, not fabricated.

## Stack (7 services + engine, all verified via real HTTP)
web :5180 (200), bff :8787 (/health {"ok":true,"model":"opus"}), worker, postgres/redis/minio
(healthy), minio-setup (exited clean). loki engine (loki dashboard) :57374 (200). loki v7.109.0.
Claude auth loggedIn=true. observability dashboard :58080 kept up.

## Real-user browser flow (Chrome) - WORKS
Sign-in ("Continue without email" DEV path, frictionless) -> build UI with cost estimate
$0.20-$1.50 -> submitted a URL-shortener spec.

## Issues hit + fixed like a real user
1. FAIL: engine.startBuild "workspace param is not enabled" -> FIX: restart engine with
   LOKI_WORKSPACE_ROOTS set.
2. FAIL: "workspace is outside allowed roots" -> FIX: set LOKI_WORKSPACE_ROOTS to match the
   worker's root (/Users/lokesh/git/autonomi-workspaces, per .env ENGINE_WORKSPACE_ROOT).
3. SUCCESS: build ran end-to-end. run_id run-20260702035846-20879-6555.

## RESULT (real, measured)
- Verdict: completion_promise_fulfilled, exit 0. ACT iterations: 1.
- Wall-clock: submit 03:58:28Z -> engine lastRun 04:05:34Z = ~7 minutes (single iteration),
  consistent with the honest single-iteration floor (docs/SPEED-DIAGNOSIS-2026-07-01.md).
- Generated app (modular, real): src/{server,app,store,validate}.js, src/routes/{shorten,
  redirect,stats}.js, public/index.html, README/USAGE/HANDOFF, express.

## PROVE-IT-TO-MYSELF (ran the real output; did not trust the verdict)
- npm test (jest): 15/15 pass across 3 suites.
- Booted the app (node src/server.js), exercised every endpoint via HTTP:
  POST /shorten -> 200 {code, shortUrl}; GET /:code -> HTTP 302 redirect to the original URL;
  GET /api/stats/:code -> {clicks:1, createdAt}; GET / -> 200. ALL WORK.
- The "we prove it works" verdict matched reality for this build.

## Error-rate + fixes for THIS run (honest)
- Infra config issues before a build could run: 2 (both operator-config, fixed).
- Build-internal failures: 0. Self-heal rounds: 0 (succeeded first pass). false-heal: 0.

## Host-engine-mode "Build log disconnected" is a DOCUMENTED REDUCTION, not a bug
The BFF build-tailer streams the log by tailing <ws>/.loki/events.jsonl directly. In
host-engine mode (default) the workspace is on the HOST, not mounted into the BFF container,
so the live stream shows "disconnected" - the SAME documented reduction as the Code tab being
honest-EMPTY (docs/ENGINE-RUN-MODES.md). NOT a defect. The live watch-stream WORKS in compose
mode (--profile with-engine, shared volume). A future ENHANCEMENT for host-engine live stream:
engine exposes an HTTP log-stream endpoint the BFF consumes over ENGINE_URL (net-new feature,
engine + saas, founder-adjacent) - recorded honestly, not "fixed" as a bug.

## Honest scope
Measures OUR run only. No competitor was run (docs/ACCURACY-MEASUREMENT-HONEST-2026-07.md;
head-to-head is founder-gated decision #8).
