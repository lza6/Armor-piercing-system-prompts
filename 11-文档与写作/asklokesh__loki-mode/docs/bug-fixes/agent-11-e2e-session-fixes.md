# Agent 11: Session Lifecycle E2E Testing - Bug Fixes

## Summary

Investigated and fixed 5 bugs in the session lifecycle (start, pause, resume, stop, restart, monitor). Also discovered and fixed 1 new bug. All 3 files modified pass syntax validation.

## Bugs Fixed

### BUG-ST-002: Pause signal not checked between quality gates
- **File**: `autonomy/run.sh` (lines ~9811, ~9833)
- **Problem**: Three quality gates (static analysis, test coverage, code review) run sequentially with no pause/stop check between them. If a user sends PAUSE during static analysis, execution continues through all remaining gates before the pause is processed on the next loop iteration. Code review alone can take 30+ seconds.
- **Fix**: Added pause/stop file checks between each quality gate. If a signal is detected, partial gate failures are saved and `continue` exits to the main loop, which will handle the pause on the next iteration.

### BUG-ST-004: Stop endpoint returns before processes are actually killed
- **File**: `dashboard/server.py` (line ~2863, `stop_session()`)
- **Problem**: The `/api/control/stop` endpoint sent SIGTERM via `os.kill(pid, 15)` and immediately returned `{"success": True, "message": "Stop signal sent"}`. The caller (dashboard UI) would show "stopped" while the process was still running and cleaning up. This could lead to users starting a new session while the old one was still shutting down.
- **Fix**: Added `await asyncio.sleep(0.5)` polling loop (up to 5s) that waits for the process to actually exit. If the process doesn't exit gracefully within 5s, escalates to SIGKILL. Response now includes `process_stopped` boolean and accurate message ("Session stopped" vs "Stop signal sent").

### BUG-ST-006: Resume doesn't validate checkpoint integrity
- **File**: `autonomy/run.sh` (`load_state()` at line ~7956)
- **Problem**: `load_state()` loaded `retryCount` and `iterationCount` from `autonomy-state.json` without validating that the file contained valid JSON or that the values were sane (non-negative integers). A corrupted or truncated state file (from a crash during save, disk full, etc.) could cause the shell to use non-numeric values, leading to arithmetic errors or infinite loops.
- **Fix**: Added pre-validation step using Python that checks: (1) file is valid JSON, (2) `retryCount` and `iterationCount` are numeric, (3) values are non-negative. If validation fails, backs up the corrupted file with a `.corrupt.<timestamp>` suffix and starts fresh with count=0.

### BUG-ST-007: Multiple concurrent pause signals cause state corruption
- **File**: `autonomy/run.sh` (`handle_pause()` at line ~10111)
- **Problem**: `handle_pause()` had no re-entrancy guard. If a signal handler triggered a second pause while one was already being handled (e.g., signal handler calling cleanup which checks pause state), two concurrent pause handlers could run, both trying to read/write PAUSE files and state. The function also did not save state on pause entry, so a crash during pause would lose the "paused" status.
- **Fix**: Added `_PAUSE_IN_PROGRESS` guard flag (checked at entry, cleared at all exit paths). Added `save_state` call at pause entry so the "paused" status persists across crashes.

### BUG-ST-008: Non-atomic session.json update in loki CLI
- **File**: `autonomy/loki` (`cmd_stop()` at line ~1354)
- **Problem**: While `run.sh` was already fixed to use atomic temp-file + `os.replace()` for session.json updates, the `loki` CLI `cmd_stop()` still used the old pattern: `f.seek(0); f.truncate(); json.dump(d, f)`. This is non-atomic -- if the process is killed between `truncate()` and the `json.dump()` completing, session.json is left empty or partially written. The next `loki status` would fail to parse it.
- **Fix**: Replaced with the same atomic pattern used in `run.sh`: `tempfile.mkstemp()` + `json.dump()` + `os.replace()`.

### BUG-ST-010 (NEW): ITERATION_COUNT spuriously incremented on pause resume
- **File**: `autonomy/run.sh` (`run_autonomous()` main loop at line ~9313)
- **Problem**: The main while loop incremented `ITERATION_COUNT` at the top of each iteration, BEFORE checking for pause/stop signals. When `check_human_intervention` returned 1 (pause handled, then resumed), the `continue` statement jumped back to the top of the loop, incrementing `ITERATION_COUNT` again without actually running an AI provider iteration. Same issue occurred with `check_budget_limit` returning true. Over a session with multiple pauses, this inflated the iteration count, causing premature `max_iterations_reached` exits and incorrect RARV tier selection.
- **Fix**: Moved pause/stop and budget checks BEFORE the `ITERATION_COUNT++` increment. Now the count only increments when an actual iteration will execute.

## Bugs Verified Already Fixed

### BUG-ST-001: save_state not atomic
- **Status**: Already fixed (line 7938). Uses temp file with PID suffix + `mv -f`.

### BUG-ST-003: ITERATION_COUNT not restored on resume
- **Status**: Already fixed (line 7964). Duplicate of BUG-RUN-003.

### BUG-ST-005: Gate escalation PAUSE writes to wrong path
- **Status**: Already fixed (line 9804). Writes to `${TARGET_DIR:-.}/.loki/PAUSE`.

## Files Modified

| File | Changes |
|------|---------|
| `autonomy/run.sh` | BUG-ST-002, BUG-ST-006, BUG-ST-007, BUG-ST-010 |
| `autonomy/loki` | BUG-ST-008 |
| `dashboard/server.py` | BUG-ST-004 |

## Validation

- `bash -n autonomy/run.sh` -- PASS
- `bash -n autonomy/loki` -- PASS
- `python3 -c "import ast; ast.parse(open('dashboard/server.py').read())"` -- PASS

## Edge Cases Considered

1. **Crash during save_state**: Atomic write via temp+mv means the file is either fully written or not written at all. No partial state.
2. **Concurrent stop+pause**: The pause handler checks for STOP file in its wait loop. If both arrive simultaneously, STOP takes precedence (handle_pause returns 1, which maps to return 2/stop in check_human_intervention).
3. **Disk full during session.json write**: `tempfile.mkstemp` will fail, caught by the `except (json.JSONDecodeError, OSError): pass` handler. The original file is untouched.
4. **OOM kill during pause**: State is saved to "paused" status at pause entry. On restart, `load_state()` will restore the paused state and the session will resume from the correct iteration.
5. **Rapid pause/resume cycling**: The `_PAUSE_IN_PROGRESS` guard prevents re-entrant pause handling. The iteration count fix prevents count inflation during rapid pause/resume cycles.
