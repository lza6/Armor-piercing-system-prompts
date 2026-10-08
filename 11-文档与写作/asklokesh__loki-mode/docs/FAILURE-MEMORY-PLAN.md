# Failure-Memory Loop - Implementation Plan

Release: next PATCH/MINOR (v7.18.4 or v7.19.0)
Status: DESIGN ONLY. No implementation code, no version bump, no commit in this plan.
Author: Architect (read-only planning pass; line numbers re-grepped on the live tree; key paths verified by running the real Python modules).

## Goal

Crashes and iteration failures become durable lessons that get injected into the
NEXT iteration's prompt, so Loki stops repeating the same mistake. Default-on
(`LOKI_FAILURE_MEMORY=1`, set to `0` to opt out). Local, zero new setup, zero
network. Builds on the already-shipped Phase 0 crash capture.

---

## IMPORTANT: this plan deviates from the task's literal Connector B design - with evidence

The task instructed Connector B to call `retrieve_anti_patterns(top_k=3)` and the
overall design to rely on consolidation turning failures into anti-patterns. I
implemented that path and tested it against the REAL modules. It does NOT close
the loop within a run. The deviation below is evidence-driven, not a preference.

### Why `retrieve_anti_patterns` cannot retrieve the failure within a run

Traced and then verified by running the modules:

1. Every ordinary non-zero iteration calls `loki_crash_capture` with a FIXED
   `error_class="IterationError"` (run.sh:12030-12038). So all plain failures
   carry the same error class.
2. If a failure were consolidated, `extract_anti_patterns` (consolidation.py:570)
   builds the anti-pattern body from `action_log[-3:]` and `resolutions`. But
   `auto_capture_episode` never populates `action_log` (run.sh:9454-9491) and our
   `ErrorEntry.resolution` is empty, so the ONLY non-empty searchable field is
   `pattern="Avoid: IterationError"` (consolidation.py:626-627). The rich
   `message` we compose in Connector A is DISCARDED by `extract_anti_patterns`.
3. `retrieve_anti_patterns(query=goal+" "+phase)` keyword-scores those words
   against `"avoid: iterationerror"` (retrieval.py:1567-1588). A real goal
   ("build a todo REST API") shares ZERO tokens with "IterationError" -> score 0
   -> not returned. Embeddings would also score near-zero. Keyword dominates on
   local setups (no `embedding_engine`).
4. All plain failures collapse into one `IterationError` group, so there is not
   even per-failure discrimination.

Empirical confirmation (ran the real `MemoryRetrieval` against a seeded failed
episode): `retrieve_anti_patterns('build a todo REST API ACT', top_k=3)`
returned `[]`. A direct recency read of the same store returned the lesson with
its full message. See "Verification performed" below.

### The fix: recency-scoped direct read (Connector B, revised)

Within a run the goal is constant, so the correct retrieval key is RECENCY
("what did I just fail at"), not goal-similarity. Connector B reads recent
FAILURE episodes directly from storage and formats their `errors_encountered`
(including the rich `message` Connector A composes), reusing the exact storage
API consolidation already uses (`list_episodes(since=)` + `load_episode`,
consolidation.py:172-182). The literal `retrieve_anti_patterns` call is KEPT as
a best-effort cross-run secondary (mostly empty locally; harmless), but it is NOT
what closes the loop.

This deviation should be visible to the reviewer: the task said "call
`retrieve_anti_patterns`"; verification shows that mechanism does not retrieve
within a run, so the within-run loop is closed by a recency read instead.

---

## Final design (validated against source + a live module run)

- CONNECTOR A - failure ingestion (run.sh `auto_capture_episode`): when
  `exit_code != 0` and the knob is on, read this iteration's scrubbed
  `.loki/crash/*.json`, map the whitelisted fields into an `ErrorEntry`, and
  attach it to the LIVE failed episode's `errors_encountered` BEFORE
  `engine.store_episode(trace)`. If telemetry is off (no crash file), SYNTHESIZE
  a minimal ErrorEntry from non-sensitive fields so the loop works regardless of
  telemetry state.
- CONNECTOR B - failure-aware retrieval (run.sh `retrieve_memory_context`): read
  recent FAILURE episodes directly and append a clearly labeled
  `PAST FAILURES TO AVOID:` block (error_type + composed message) to the memory
  context that `build_prompt` carries into the next iteration. Keep a best-effort
  `retrieve_anti_patterns` secondary for cross-run lessons.
- Connector C (per-iteration consolidation) is DROPPED. With the recency read it
  is not load-bearing (it could not produce a retrievable lesson anyway, per the
  evidence above), and the existing end-of-run consolidations
  (run.sh:12289/12338/12680) still provide cross-run semantic durability.
  Dropping it removes the perpetual-mode-volume, lock-contention, and
  index-staleness risks entirely and net-reduces code.
- Gate: `LOKI_FAILURE_MEMORY` (default 1). Both connectors no-op when `0`.
- Dual-route: bash is the engine. The Bun `build_prompt.ts` `retrieveMemoryContext`
  is an intentional empty stub; only static-line parity (if any) + fixture
  refresh applies - recommended: add no static line, so zero Bun change.

---

## Verification performed (so the reviewer can trust the deviation)

Ran the real `memory` modules against a temp `.loki/memory`:
- Stored a failed `EpisodeTrace` (outcome="failure", goal="build a todo REST API")
  with `ErrorEntry(error_type="IterationError", message="phase=ACT; signature:
  handler > parse > json.loads; fp=abc123def456", resolution="")`.
- `MemoryStorage.list_episodes(since=now-24h)` -> 1 episode; filtered
  outcome=="failure" -> 1; surfaced the lesson WITH its full message. (Connector B
  recency read works.)
- `MemoryRetrieval.retrieve_anti_patterns("build a todo REST API ACT", top_k=3)`
  -> `[]`. (Confirms the literal path does not retrieve within a run.)
- Confirmed `loki_crash_capture` fires on EVERY non-zero, non-signal iteration
  exit at run.sh:12030-12038 (before `auto_capture_episode` at 12255), so a
  scrubbed crash file normally exists for Connector A when telemetry is on.

---

## Exact files and functions to change

Line numbers re-grepped on the current tree; they drift - re-`grep -n` before editing.

### 1. autonomy/run.sh

#### 1a. CONNECTOR A - `auto_capture_episode` (def run.sh:9303; Python heredoc 9428-9524)

The episode is built and stored in ONE Python heredoc (`EpisodeTrace.create` at
9454; `engine.store_episode(trace)` at 9491). Inject the `ErrorEntry` INSIDE this
heredoc, after `trace.outcome = outcome` (9460) and before `store_episode` (9491).
Do not load-modify-restore on disk; that would race the store.

Bash, in the function body before the heredoc env block (~9420), gated:

```
# CONNECTOR A: locate this iteration's scrubbed crash file (failure only).
local _crash_json=""
if [ "${LOKI_FAILURE_MEMORY:-1}" != "0" ] && [ "$exit_code" -ne 0 ] \
    && [ -d "$target_dir/.loki/crash" ]; then
    _crash_json=$(ls -t "$target_dir/.loki/crash/"*.json 2>/dev/null | head -1 || true)
fi
```

Pass `_LOKI_CRASH_JSON="$_crash_json"`, `_LOKI_FAILURE_MEMORY="${LOKI_FAILURE_MEMORY:-1}"`,
and the already-available `_LOKI_RARV_PHASE` / `_LOKI_EXIT_CODE` into the heredoc
env block (9420-9427). Inside the heredoc, between 9460 and 9491:

```
if os.environ.get('_LOKI_FAILURE_MEMORY', '1') != '0' and outcome == 'failure':
    try:
        from memory.schemas import ErrorEntry
        crash_json_path = os.environ.get('_LOKI_CRASH_JSON', '')
        _err_type = 'IterationError'
        _message = ''
        if crash_json_path:
            with open(crash_json_path, 'r', encoding='utf-8') as _cf:
                _crash = json.load(_cf)
            _err_type = (_crash.get('error_class')
                         or _crash.get('friction_kind') or 'IterationError')
            _sig = _crash.get('stack_signature') or []
            _sig_str = ' > '.join(str(s) for s in _sig[:5]) if isinstance(_sig, list) else str(_sig)
            _phase = _crash.get('rarv_phase') or rarv_phase or ''
            _parts = []
            if _phase: _parts.append('phase=' + str(_phase))
            if _crash.get('friction_kind'): _parts.append('friction=' + str(_crash['friction_kind']))
            if _sig_str: _parts.append('signature: ' + _sig_str)
            if _crash.get('fingerprint'): _parts.append('fp=' + str(_crash['fingerprint'])[:12])
            _message = '; '.join(_parts) or 'iteration failed'
        else:
            # Telemetry-independent fallback: no crash file (e.g. telemetry off).
            # Synthesize from non-sensitive fields only. Nothing raw, no scrub needed.
            _ec = os.environ.get('_LOKI_EXIT_CODE', '')
            _message = 'phase=' + str(rarv_phase or '') + '; exit=' + str(_ec)
        trace.errors_encountered.append(ErrorEntry(
            error_type=str(_err_type), message=_message, resolution=''))
    except Exception:
        pass  # never block episode capture
```

Notes:
- `trace.outcome` is already "failure" for non-zero exit (9411-9413, 9460), so
  the failed-episode filter (consolidation.py:192) and Connector B's recency read
  both see it. No extra outcome wiring.
- REUSE the scrubbed crash file; never re-capture, never read raw. The fallback
  branch uses only `rarv_phase` + `exit_code` (no log text, no paths) so it is
  safe with NO scrubbing and works when telemetry is off.

#### 1b. CONNECTOR B - `retrieve_memory_context` (def run.sh:9031; Python heredoc 9045-9071)

Add `_LOKI_FAILURE_MEMORY="${LOKI_FAILURE_MEMORY:-1}"` to the env block at
9043-9044. After the existing `RELEVANT MEMORIES:` loop (9063-9068) and before
`PYEOF` (9071), add the recency read + best-effort secondary:

```
    if os.environ.get('_LOKI_FAILURE_MEMORY', '1') != '0':
        try:
            from memory.storage import MemoryStorage as _MS
            from memory.schemas import EpisodeTrace as _ET
            from datetime import datetime as _dt, timezone as _tz, timedelta as _td
            _s = storage if 'storage' in dir() else _MS(f'{target_dir}/.loki/memory')
            _since = _dt.now(_tz.utc) - _td(hours=24)
            _lessons = []
            for _eid in _s.list_episodes(since=_since, limit=50):
                _data = _s.load_episode(_eid)
                _ep = _ET.from_dict(_data) if isinstance(_data, dict) else _data
                if getattr(_ep, 'outcome', '') != 'failure':
                    continue
                # Carry the episode timestamp so we can sort by true wall-clock
                # recency. list_episodes is newest-DAY-first, but within a day
                # files sort by a random uuid suffix in the id, NOT by time, so
                # slicing the raw order would drop the most-recent same-day
                # failure once a run has >3 in one day. Sort by timestamp instead.
                _ts = getattr(_ep, 'timestamp', None)
                _ts_key = _ts.isoformat() if hasattr(_ts, 'isoformat') else str(_ts or '')
                for _e in getattr(_ep, 'errors_encountered', []):
                    _lessons.append((_ts_key, _e.error_type, _e.message))
            # newest first by true wall-clock timestamp, then take 3
            _lessons.sort(key=lambda _x: _x[0], reverse=True)
            _lessons = [(_t, _m) for (_k, _t, _m) in _lessons[:3]]
            if _lessons:
                print('')
                print('PAST FAILURES TO AVOID:')
                for _t, _m in _lessons:
                    _line = '- ' + str(_t)[:80]
                    if _m:
                        _line += ': ' + str(_m)[:160]
                    print(_line)
        except Exception:
            pass
        # Best-effort cross-run secondary (mostly empty locally; harmless).
        try:
            _anti = retriever.retrieve_anti_patterns((goal + ' ' + phase).strip() or goal, top_k=3)
            for _a in _anti[:3]:
                _w = _a.get('what_fails') or _a.get('incorrect_approach') or _a.get('pattern', '')
                if _w:
                    print('- (prior) ' + str(_w)[:120])
        except Exception:
            pass
```

`build_prompt` captures this function's stdout into `memory_context` at
run.sh:9968. Confirm `list_episodes`/`load_episode` exist (storage.py:477/447 -
verified) and that the recency read surfaces an episode stored last iteration by
`engine.store_episode` (same-day `episodic/<date>/`; verified by live run).

### 2. loki-ts/src/runner/build_prompt.ts - PARITY ONLY (no logic port)

`retrieveMemoryContext` (build_prompt.ts:371-378) is an intentional empty stub
(returns "" unless `.loki/memory/index.json` exists, and "" even then; comment
374-377). Called once at build_prompt.ts:976. The DYNAMIC failure block is
environment-derived and already excluded from parity (stub returns "" and bash
returns "" when Python errors). Recommendation: add NO static instruction line ->
zero Bun change, parity stays green. If product insists on an explicit directive
line, add ONE static line, mirror it exactly in build_prompt.ts, and refresh
`loki-ts/tests/fixtures/build_prompt/*` via the repo's existing fixture-refresh
path (do not hand-edit fixtures). Update
`loki-ts/tests/parity/build_prompt.test.ts` only in that case.

---

## ErrorEntry field mapping (from the scrubbed crash whitelist)

Whitelist source: `autonomy/lib/crash_redact.py` `_WHITELIST` (lines 45-61).
`ErrorEntry` shape: `error_type`, `message`, `resolution` (schemas.py:105-117).
On-disk crash JSON is the post-scrub whitelist dict (crash_capture.py:194-200).
For ordinary failures the crash file is written at run.sh:12033 with
`error_class="IterationError"`.

| ErrorEntry field | Source crash field(s) | Mapping rule |
|------------------|------------------------|--------------|
| `error_type` | `error_class` -> else `friction_kind` -> `"IterationError"` | `error_class` is the sanitized class token (for ordinary failures it is "IterationError"; for friction records "Friction" with `friction_kind` carrying the kind). Becomes the displayed label. |
| `message` | composed from `rarv_phase` + `friction_kind` + `stack_signature` (first 5 frames) + `fingerprint` (first 12 chars) | THIS is the discriminating, retrievable content (Connector B surfaces it directly; `extract_anti_patterns` would have thrown it away). All from whitelisted fields; never raw stack text. |
| `resolution` | none at capture time | `""` (tolerated downstream). |

Telemetry-off fallback (no crash file): `error_type="IterationError"`,
`message="phase=<rarv_phase>; exit=<exit_code>"`, `resolution=""`. Uses only
non-sensitive fields -> no scrubbing required, no leak.

Not mapped (already on episode or not useful): `os`, `arch`, `loki_version`,
`node_version`, `bun_version`, `exit_code` (episode outcome already encodes
failure), `project_id_hash`, `rules_version`, `redactions_count`, `captured_at`.

All mapped values originate from the WHITELISTED file (or non-sensitive fallback
fields), so no new scrubbing is required and docs/PRIVACY.md is preserved.

---

## PAST FAILURES TO AVOID block: injection point and format

- Producer: `retrieve_memory_context` Python heredoc (run.sh, after 9068).
- Consumer: captured into `memory_context` at run.sh:9968, embedded in the prompt.
- Format (no emojis, no em dashes):

```
PAST FAILURES TO AVOID:
- <error_type, <=80 chars>: <message: phase / signature / fp, <=160 chars>
- ...
- (prior) <cross-run anti-pattern, <=120 chars>   # only if any exist
```

- Placement: AFTER `RELEVANT MEMORIES:` and the managed-store block, so positive
  memories come first and failures read as constraints. Capped at 3 recent
  lessons + up to 3 cross-run secondaries to bound prompt growth.

---

## Default-on knob wiring (`LOKI_FAILURE_MEMORY`)

- Default 1 (on). Opt out with `LOKI_FAILURE_MEMORY=0`. Read via
  `${LOKI_FAILURE_MEMORY:-1}` (matches existing default-on knobs like
  `LOKI_INTELLIGENT_USAGE`, run.sh:12333).
- Gates (no-op when 0): Connector A crash lookup + ErrorEntry injection (bash
  guard + heredoc env check); Connector B recency read + secondary (heredoc env
  check).
- When off: failures do not attach an ErrorEntry; no `PAST FAILURES TO AVOID:`
  block is emitted; behavior reverts to current.
- INDEPENDENCE from telemetry (decided, not just documented): the crash-file
  WRITE at run.sh:12030 is gated by `loki_collection_enabled` (crash.sh:30). So a
  user with telemetry OFF but `LOKI_FAILURE_MEMORY=1` (default) would otherwise
  get a silently empty loop. Connector A's synthesized-fallback branch closes
  that gap, so the feature works regardless of telemetry state. Document the
  interaction AND ship the fallback.

---

## New tests

Reuse patterns from `tests/integration/test_rarv_c_memory_flow.sh` (behavioral
simulation exercising the real Python modules), `tests/crash/`, and
`tests/test-crash-cli.sh`.

### Test 1 (PRIMARY) - end-to-end, driven by the REAL input and the REAL query
New file: `tests/integration/test_failure_memory_loop.sh`

This test must NOT seed a query-matching record and must NOT put the error class
in the query (that is the mask that hid the original retrieval bug):
1. Input via the real path: write the crash file the way run.sh:12033 does
   (`error_class="IterationError"`, a stack producing a `stack_signature`), via
   `autonomy/lib/crash_capture.py` so the whitelist is authentic.
2. Connector A: build + store a failed `EpisodeTrace` with the ErrorEntry mapped
   from that crash file. Assert `errors_encountered` non-empty and `message`
   contains the signature/fingerprint.
3. Connector B: run the recency-read body with a goal that shares NO tokens with
   the error class (e.g. "build a todo REST API"). Assert stdout contains
   `PAST FAILURES TO AVOID:` AND the stack_signature/fingerprint text. If green
   only when the error class is in the query, the test is wrong.
4. Telemetry-off fallback: with no crash file, assert Connector A synthesizes an
   ErrorEntry (`phase=...; exit=...`) and Connector B still emits the block.

### Test 2 - knob off is inert
`LOKI_FAILURE_MEMORY=0`: no ErrorEntry attached; no `PAST FAILURES TO AVOID:`.

### Test 3 - Connector A mapping unit (Python, tests/memory/)
Feed crash JSON shapes (IterationError, Friction, ScrubError minimal, and the
no-file fallback) and assert the mapping (error_class vs friction_kind fallback;
empty resolution; message composed only from whitelisted/non-sensitive fields).

### Test 4 - privacy regression (tests/crash/ negative style)
Assert the ErrorEntry message and the rendered block contain none of: home path,
repo owner/name, email, IPv4/IPv6 (cannot, since inputs are whitelisted or the
non-sensitive fallback - guard test).

### Test 5 - Bun parity (only if a static line is added)
If a static directive line is added, extend
`loki-ts/tests/parity/build_prompt.test.ts` and refresh fixtures. If kept purely
dynamic (recommended), assert the existing parity suite passes unchanged.

---

## CHANGELOG entry (with honest "NOT tested" section)

```
### Added
- Failure-memory loop (LOKI_FAILURE_MEMORY, default on): iteration failures and
  crashes are surfaced into the next iteration's prompt under a
  "PAST FAILURES TO AVOID:" heading (error type + sanitized phase/stack-signature/
  fingerprint), so Loki stops repeating the same mistake. Local-only, zero new
  setup, zero network. Reuses Phase 0 scrubbed crash files (no re-capture, no raw
  data); works even with telemetry off via a non-sensitive fallback. Opt out with
  LOKI_FAILURE_MEMORY=0.

### Changed
- auto_capture_episode attaches a scrubbed (or non-sensitive fallback) ErrorEntry
  to the failed episode's errors_encountered (Connector A).
- retrieve_memory_context surfaces the most recent failure lessons by recency
  (Connector B), with a best-effort cross-run anti-pattern secondary.

### NOT tested (honest disclosure)
- Not validated on a real multi-iteration live run against a paid provider; the
  end-to-end test is a behavioral simulation against the real Python modules, not
  a full runner boot.
- Lesson usefulness is heuristic: the message carries phase + stack signature +
  fingerprint but no auto-derived fix/resolution, so guidance is "what failed,"
  not "how to fix." Whether that measurably reduces repeats is not quantified.
- Cross-run anti-pattern retrieval (the retrieve_anti_patterns secondary) is
  known to rarely match goal+phase queries (error class shares no goal tokens);
  it is kept best-effort and is not the loop-closer. Not precision-tested.
- Crash-file-to-episode matching uses most-recent-by-mtime; not tested under
  rapid multi-crash iterations.
- Bun route: failure-memory is intentionally not implemented (stub unchanged).
```

---

## Risks

| # | Risk | Likelihood | Impact | Mitigation |
|---|------|-----------|--------|------------|
| 1 | Crash-file-to-episode mismatch when multiple crash files exist in one iteration | Medium | Low | Connector A picks most-recent-by-mtime; the per-iteration crash write (12033) runs just before capture. Connector B's recency read is on EPISODES (not crash files), so even a wrong crash file only mislabels one lesson, not the loop. Future: match by fingerprint/timestamp. |
| 2 | Within-run loop closure (anti-pattern unreachable via goal query) | Was HIGH | High | RESOLVED by switching Connector B to a recency-scoped direct episode read (verified by live module run). The literal retrieve_anti_patterns path returned []; recency read returned the lesson. |
| 3 | Telemetry-off silently empties the loop (no crash file) | Was HIGH | High | RESOLVED by Connector A's non-sensitive synthesized-fallback ErrorEntry. Feature is now independent of telemetry state. |
| 4 | Retrieval relevance: recency may surface a failure unrelated to the current sub-goal | Low | Low | Within a run the goal is roughly constant, so recent failures are relevant by construction. Capped at 3. |
| 5 | Lesson quality is thin (no auto-resolution) | Medium | Medium | message carries phase + stack_signature + fingerprint (discriminating). Flagged NOT tested for repeat-reduction. Future: thread a resolution/fix once available. |
| 6 | Prompt bloat | Low | Low | <=3 recent + <=3 cross-run lines, each bounded (<=240 chars). No static line (Bun parity unchanged). |
| 7 | Privacy: lesson leaking raw data | Low | High | Inputs are whitelisted crash fields or non-sensitive fallback (phase/exit only). Guard test (Test 4) asserts no path/repo/email/IP. Local only, no egress. |
| 8 | Perpetual-mode volume / consolidation lock contention | Eliminated | n/a | Connector C (per-iteration consolidation) was DROPPED; only the existing end-of-run consolidations remain. Index-staleness (BUG-MEM-002) also moot for this feature since Connector B reads episodes, not the vector index. |

---

## Sequencing

1. Connector A (run.sh `auto_capture_episode` heredoc + bash crash lookup + fallback).
2. Connector B (run.sh `retrieve_memory_context` recency read + env + secondary).
3. Tests 1-4 (the simulation must pass with a non-matching goal query before any Bun work).
4. Bun parity decision (recommended: no static line -> no Bun change). Only if a
   static line is added: mirror in build_prompt.ts + refresh fixtures + Test 5.
5. CHANGELOG entry. (Version bump and commit are out of scope for this plan.)

## Critical Files for Implementation
- /Users/lokesh/git/loki-mode/autonomy/run.sh
- /Users/lokesh/git/loki-mode/memory/storage.py
- /Users/lokesh/git/loki-mode/memory/schemas.py
- /Users/lokesh/git/loki-mode/autonomy/lib/crash_redact.py
- /Users/lokesh/git/loki-mode/memory/retrieval.py
