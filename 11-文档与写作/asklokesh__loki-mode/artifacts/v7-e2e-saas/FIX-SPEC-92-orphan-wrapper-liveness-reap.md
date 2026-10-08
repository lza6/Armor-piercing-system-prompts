# Fix spec: #92 -- loki-run wrapper orphans and persists for hours (self-reaping blind spot)

Component: **loki-mode** engine (autonomy/run.sh). Root cause + severity observed this session.

## Confirmed root cause (observed, not inferred)
- Two wrappers `/tmp/loki-run-NLeQEP.sh` were found alive with **PPID=1** (reparented to init: the
  launching session died), **etime 3h14m**, state `SN`, each with ONLY a `sleep` child (one `sleep 300`
  = the retry backoff, one `sleep 2`). No `claude -p` / node engine child anywhere under them.
- `cleanup_orphan_pids()` (run.sh:1441) reaps orphans, BUT it only scans the PID REGISTRY
  (`$PID_REGISTRY_DIR/*.json`). The wrapper registers its CHILDREN (claude, dashboard) but NOT ITSELF,
  so when the wrapper's OWN parent dies, nothing reaps the wrapper. Self-reaping blind spot.
- It is also only reactive ("Called on startup and by `loki cleanup`") -- the orphan persists until the
  NEXT run happens to sweep, which may be hours (observed: 3h+).

## Severity: MEDIUM (process hygiene), NOT a cost leak -- measured
The 3h+ orphan had ONLY `sleep` children across its whole lifespan; no `claude -p` was ever observed
running under it. Strong evidence the orphan spends its life in backoff sleep and makes NO API calls
(no token burn). (Honest scope: not exhaustively proven across every retry tick -- observed, not a
formal proof. If a future trace shows a retry firing `claude -p` on a dead session, re-rate to HIGH
cost leak.) So: hygiene / resource-cleanliness, not survival-cost.

## The WRONG fix (do NOT ship): a parentage trigger
Both "self-exit when parent dies" and "self-register the wrapper's PPID" key on PARENT-DEATH, which
CANNOT distinguish a crashed session from an intentional `nohup`/detach. Detached overnight builds
(launch, close laptop, parent shell dies, build continues) are THE primary autonomous use case:
- self-exit-on-parent-death -> closing the terminal kills a legitimate build (disqualifying).
- self-register-wrapper -> the NEXT concurrent run's cleanup_orphan_pids sees parent-dead and reaps a
  legitimately-detached run (removes today's accidental protection = regression).

## The RIGHT fix: a LIVENESS/PROGRESS predicate (nohup-safe, modular, testable)
Mirror the SaaS BFF `sweepStuckBuilds` predicate (spares "recently active", fails only "genuinely dead
AND no recent activity"). Reap a wrapper only when it is **orphaned AND has made no progress for N
minutes** (no engine child AND no advancing `.loki` activity: events.jsonl / signals / state mtime not
advanced within an idle budget). A productive detached run keeps writing activity -> spared; a dead
3-hour orphan writes nothing -> reaped. Proactive (no multi-hour window), nohup-safe, and a modularity
win: the SAME liveness concept on both the SaaS-worker side (#91) and the engine-wrapper side.

### Implementation shape (BINDING: extract a PURE predicate for hermetic testing)
1. `shouldReapOrphan(parent_alive, last_activity_ms, now_ms, idle_budget_ms) -> 0/1` -- a PURE bash
   function, NO side effects. Reap iff `parent_dead AND (now - last_activity) > idle_budget` AND no live
   engine child. Default idle_budget generous (e.g. 15-20 min) so a slow-but-live build is never reaped.
2. Register the wrapper's own PID in the registry (so cleanup_orphan_pids can SEE it) BUT gate the kill
   on `shouldReapOrphan`, not on parent-death alone -- this closes the blind spot without the
   detached-run regression (the liveness check spares live detached runs).
3. `last_activity_ms` = max mtime of `.loki/events.jsonl`, `.loki/signals/*`, `.loki/autonomy-state.json`.
4. Keep it in `cleanup_orphan_pids` (reactive next-run sweep) AND optionally a lightweight self-check
   before each backoff sleep in the wrapper's retry loop (proactive) -- both call the same predicate.

### Tests (RED first, hermetic -- function extraction, no real process death)
- shouldReapOrphan: parent_dead + stale activity + no child -> 1 (reap). parent_dead + FRESH activity
  -> 0 (spare a live detached run). parent_alive -> 0. parent_dead + within idle_budget -> 0.
- Extract shouldReapOrphan to a scratch harness (mirror the existing function-extraction test discipline);
  feed synthetic timestamps; assert reap/spare. Do NOT orchestrate real process death (flaky).

## Verification
- Unit: the extracted predicate passes the RED/GREEN table above.
- Behavioral: a detached (`nohup`) run that keeps writing `.loki` activity is NEVER reaped; a killed-parent
  orphan with no activity past the idle budget IS reaped on the next sweep.
- Regression: full local-ci green; no change to a normal foregrounded run's lifecycle.
