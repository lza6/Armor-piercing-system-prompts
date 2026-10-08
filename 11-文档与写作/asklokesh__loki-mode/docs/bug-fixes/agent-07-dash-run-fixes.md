# Agent 07: Dashboard + run.sh Integration Bug Fixes

## Area: Dashboard API (server.py) <-> Orchestrator (run.sh) Integration

## Known Bugs -- Verification Status

All 7 known bugs (BUG-RUN-001 through BUG-RUN-010) were already patched in the codebase
with fix comments. Verified each is correctly addressed:

| Bug ID | Description | Status |
|--------|-------------|--------|
| BUG-RUN-001 | Completion promise checks stale daily log | FIXED (line 9873: uses `$iter_output`) |
| BUG-RUN-002 | Rate limit detection greps stale daily log | FIXED (line 9910: uses `$iter_output`) |
| BUG-RUN-003 | ITERATION_COUNT never persisted across restarts | FIXED (line 7964: restored from state) |
| BUG-RUN-004 | Inconsistent JSON formats in state files | FIXED (queue normalization via jq at line 3308) |
| BUG-RUN-005 | OpenSpec queue has no deduplication | FIXED (line 8665: `existing_ids` check) |
| BUG-RUN-009 | Gate escalation PAUSE writes to wrong path | FIXED (line 9804: `touch .loki/PAUSE`) |
| BUG-RUN-010 | Retry counter increments on success | FIXED (lines 9852, 9898: `retry=0`) |

## New Bugs Found and Fixed

### BUG-NEW-001: WebSocket push inflates running_agents count
- **File:** `dashboard/server.py` line 366
- **Root cause:** `_push_loki_state_loop` counted `len(agents_list)` from the JSON
  without validating PIDs. Dead agents still appeared as running. The REST endpoint
  `get_status` correctly validated each PID with `os.kill(pid, 0)`.
- **Impact:** Dashboard WebSocket clients show ghost agents that are actually dead.
- **Fix:** Added PID validation loop matching `get_status` behavior.

### BUG-NEW-002: Dashboard drops tasks in object-format queue files
- **File:** `dashboard/server.py` line 1081
- **Root cause:** `list_tasks` only handled plain array `[...]` queue files. If a queue
  file was written in `{"tasks": [...]}` format (which `load_queue_tasks` in run.sh
  explicitly supports), all tasks were silently dropped.
- **Impact:** Tasks written by external tools using object format are invisible in dashboard.
- **Fix:** Added dict-unwrapping: `raw_items.get("tasks", [])` before array check.

### BUG-NEW-003: Per-iteration temp files leak on success paths
- **File:** `autonomy/run.sh` lines 9853 and 9899
- **Root cause:** The success `continue` paths (perpetual mode + normal success) skip
  `rm -f "$iter_output"`. Only the terminal completion paths (council/promise fulfilled)
  and the failure path clean up. Over hundreds of iterations, `.loki/logs/iter-output-*`
  files accumulate.
- **Impact:** Disk space leak proportional to iteration count. Each file contains full
  iteration output (can be MBs).
- **Fix:** Added `rm -f "$iter_output"` before both success `continue` statements.

### BUG-NEW-004: Event JSON emits floats as quoted strings
- **File:** `autonomy/run.sh` line 951
- **Root cause:** `emit_event_json` regex `^[0-9]+$` only matches integers. A value
  like `cost=3.14` is treated as a string and quoted (`"cost":"3.14"`), creating
  invalid typed JSON for consumers expecting numbers.
- **Impact:** Dashboard/OTEL consumers that parse event JSON get string types for
  float metrics (cost, duration, etc.).
- **Fix:** Changed regex to `^[0-9]+\.?[0-9]*$` to match both integers and floats.

### BUG-NEW-005: Dashboard stop leaves orphaned iter_output files
- **File:** `dashboard/server.py` line 2907
- **Root cause:** `stop_session` sends SIGTERM and marks session as stopped but does
  not clean up `.loki/logs/iter-output-*` temp files from the killed process.
- **Impact:** Orphaned temp files persist after dashboard-initiated stops.
- **Fix:** Added glob cleanup of `iter-output-*` files after SIGTERM.

### BUG-NEW-006: WebSocket broadcasts stale "running" status after crash
- **File:** `dashboard/server.py` line 382
- **Root cause:** `_push_loki_state_loop` determined status purely from
  `dashboard-state.json`'s `mode` field. If the process crashed (SIGKILL, OOM, etc.),
  the state file still said `"mode": "autonomous"`, so WebSocket clients saw "running"
  indefinitely. The REST `get_status` endpoint correctly cross-checked the PID.
- **Impact:** Dashboard UI shows session as running after crash until next full poll.
- **Fix:** Added PID liveness check before status determination. If PID is dead,
  status is forced to "stopped" regardless of state file contents.

## Integration Points Verified (No Bugs Found)

1. **Pricing tables match:** `_DEFAULT_PRICING` in server.py and `pricing` dict in
   run.sh `check_budget_limit()` have identical rates for all 6 models.

2. **Atomic state writes:** `save_state()` uses temp file + `mv` (atomic rename).
   `write_dashboard_state()` also uses temp + mv. Dashboard uses `_safe_json_read`
   with retry for race protection.

3. **Midnight-crossing:** `parse_claude_reset_time()` handles past-time correctly by
   adding 86400 seconds. No midnight bug.

4. **Session lifecycle:** `stop_session` creates STOP file + SIGTERM, `pause_session`
   creates PAUSE file, `resume_session` removes both. All match run.sh's
   `check_human_intervention()` expectations.

5. **Budget enforcement:** Both dashboard `/api/cost` and run.sh `check_budget_limit()`
   read from `.loki/metrics/efficiency/*.json` with matching cost calculation logic.

## Files Modified

- `autonomy/run.sh` -- 3 fixes (BUG-NEW-003 x2, BUG-NEW-004)
- `dashboard/server.py` -- 4 fixes (BUG-NEW-001, BUG-NEW-002, BUG-NEW-005, BUG-NEW-006)

## Validation

- `bash -n autonomy/run.sh` -- PASS
- `python3 -c "import ast; ast.parse(open('dashboard/server.py').read())"` -- PASS
