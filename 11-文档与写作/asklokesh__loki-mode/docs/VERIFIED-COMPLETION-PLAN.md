# Verified Completion Plan (v7.19.1, MINOR)

Status: DESIGN ONLY. No implementation, no version bump, no commit.
Author: Architect (Loki Mode "verified completion" release)
Target version: 7.19.1 (current VERSION is 7.19.0)

---

## 1. Goal and threat model

Loki must PROVE a run is actually done before the completion council lets the run
STOP. It must refuse to accept a fabricated "done." Concretely, block the
completion-approval path unless there is REAL on-disk evidence that:

- (a) **files actually changed** -- a nonzero git diff between the run-start SHA
  and HEAD, AND
- (b) **tests actually passed** -- a green test-results signal (where a test
  suite exists).

This attacks the #2 documented user trust-killer: agents claiming "done" when
nothing shipped.

Default-on, opt-out via `LOKI_EVIDENCE_GATE=0`, because a false block would stop
a legitimate completion (high cost). The design is built around the principle:
**block only on positive evidence of fabrication; treat inconclusive as
pass-through.**

### Honest limit (state plainly)

This gate proves "something changed and the test suite is green." It does NOT
prove PRD-semantic correctness -- it cannot tell whether the right thing was
built, only that *a* thing was built and tests pass. Semantic judgment stays
with the council votes and the Devil's Advocate. The evidence gate is a cheap,
deterministic floor under the expensive, fallible LLM votes -- not a replacement
for them.

---

## 2. Verified terrain (re-grepped; line numbers drift)

Confirmed by reading source, not assumed:

- **Bash is the live council.** `autonomy/run.sh` sources
  `autonomy/completion-council.sh` (run.sh:617-619) and calls
  `council_should_stop` (run.sh:12382). `loki-ts/src/runner/council.ts` is a
  port slice ("second slice of completion-council.sh port", council.ts:1-9) and
  is NOT wired into the runtime loop. **No Bun/TS change is needed for this
  release** (see Section 7).

- **`council_evaluate`** (completion-council.sh ~1511-1591): Phase 1
  `council_reverify_checklist` (~1519); Phase 2 `council_checklist_gate`
  (~1522, HARD gate, `return 1` blocks STOP); Phase 3 aggregate votes; Phase 4
  unanimous + Devil's Advocate. Returns 0 = COMPLETE/STOP, 1 = CONTINUE.

- **`council_checklist_gate`** (~804-894): reads
  `.loki/checklist/verification-results.json` and `.loki/checklist/waivers.json`;
  `return 0` = pass (no file => no gate, backwards compatible), `return 1` =
  block; on block writes `$COUNCIL_STATE_DIR/gate-block.json` (atomic
  temp+mv); on pass removes a stale `gate-block.json`. **This is the pattern we
  clone.** `COUNCIL_STATE_DIR` = `<loki_dir>/council` (set at
  completion-council.sh:119).

- **`council_should_stop`** (~1811-1914): the only place that writes the
  `COMPLETED` marker on real approval is the `if council_evaluate` branch
  (completion-council.sh:1863). The two force-stop safety valves -- stagnation
  (1899-1903) and done-signal (1907-1911, "agent keeps saying done") -- `return 0`
  but do **NOT** write `COMPLETED`. They are *give-up / resource-protection*
  exits, not *approved-done* claims. (Verified: the only `COMPLETED` writes are
  completion-council.sh:1863 and run.sh:12773; neither valve writes it.)

- **Second approval path (force-review).** `run.sh:12762-12784` handles a
  dashboard-triggered `COUNCIL_REVIEW_REQUESTED` signal. It already calls
  `council_checklist_gate` before approving (run.sh:12766) and writes `COMPLETED`
  directly (run.sh:12773), bypassing `council_evaluate`. **This is a second
  insertion point** the gate must cover for parity (see Section 1, insertion B).

- **Per-iteration SHA exists; run-start SHA does NOT.** run.sh:11560 captures
  `_LOKI_ITER_START_SHA=$(git rev-parse HEAD)` per attempt; there is no run-wide
  baseline. We must capture one (Section 2-design).

- **`_git_diffstat` is NOT a bash helper.** It is a Python function inside
  `autonomy/lib/proof-generator.py:199`, reading `_LOKI_ITER_START_SHA` and
  diffing `base..HEAD` (committed only). It is not callable from bash. (See
  Section 3 deviation note for the mechanism we actually use.)

- **The authoritative green-test signal is `.loki/quality/test-results.json`,
  NOT `verification-results.json`.** Written by `enforce_test_coverage`
  (run.sh:6220-6396), shape:
  `{"timestamp","runner","pass":true|false,"min_coverage","summary"}`, with the
  special **no-suite** case `{"runner":"none","pass":true,"summary":"No test
  runner detected"}` (run.sh:6373-6379). `enforce_test_coverage` runs earlier in
  the same iteration (run.sh:12231, gated by `PHASE_UNIT_TESTS`, default true),
  before the council check (run.sh:12382), so this file is reasonably fresh.
  (See Section 3 deviation note for why we do NOT use verification-results.json
  for the test signal.)

---

## 3. Deviations from the task's stated terrain (flagged, not papered over)

The task instructed reuse of two things that, on reading source, do not work as
described. Stating both:

### Deviation A -- test-green source

Task said: read tests-green from `.loki/checklist/verification-results.json`,
"reuse the existing parse." **Insufficient.** That file (written by
checklist-verify.py:336-358) stores per item only `id`, `title`, `priority`,
`status` (`verified|failing|pending`). It does NOT store the check `type`, so it
cannot distinguish a real test (`tests_pass`/`command` check) from a
`file_exists` check. Using it would conflate "a file exists" with "tests pass."

**Decision:** use `.loki/quality/test-results.json` (`runner`/`pass`) as the
authoritative green-test signal. It records the actual runner that ran and a
boolean pass, and explicitly encodes the no-suite case (`runner:"none"`). This is
still "reuse existing on-disk evidence" -- just the correct file.

### Deviation B -- git diff helper

Task said: reuse `_git_diffstat`. **Not callable from bash** (it is a Python
function in proof-generator.py keyed on `_LOKI_ITER_START_SHA`, the per-iteration
baseline, and counts only committed changes).

**Decision:** the gate computes the diff inline with
`git diff --numstat <start-sha> HEAD`, matching the existing council convergence
convention already in completion-council.sh (`git diff --stat HEAD` at ~165,
`git diff --name-only HEAD` at ~208). We diff against the **run-start** SHA, not
the per-iteration SHA, because Loki auto-commits per iteration -- by the time the
council runs, the per-iteration working tree is clean and `git diff HEAD` is
empty even on a productive run. The run-start baseline is the only SHA that
answers "did this run ship anything." We count committed changes
(`<start-sha>..HEAD`), which is correct post-auto-commit; if uncommitted changes
remain they are additive evidence, not required.

---

## 4. Design

### 4-design.1 -- Capture the run-start SHA (fresh-run aware)

Add, in `run_autonomous()` (run.sh ~11412, AFTER `load_state`, BEFORE the
`while [ $retry -lt $MAX_RETRIES ]` loop at run.sh:11495), a capture that
persists to `.loki/state/start-sha`.

**Critical lifecycle rule (do NOT "set if absent" alone).** A naive "set only if
the file is missing" makes the gate toothless on any repo Loki has run before:
the stale baseline from the *first* run persists, so on every later run
`base..HEAD` shows the entire prior history => nonzero diff => gate passes
trivially even if the new run shipped nothing. The baseline must be (re)captured
on a **fresh run** and preserved only on a **genuine resume**.

The fresh-vs-resume signal already exists: `load_state` (run.sh:9790-9834)
restores `ITERATION_COUNT` from `.loki/autonomy-state.json`. After `load_state`:

- `ITERATION_COUNT == 0` => fresh run (new invocation, or state was
  reset/corrupted) => **recapture** start-sha (overwrite).
- `ITERATION_COUNT > 0` => genuine resume of an in-flight run => **keep** the
  existing start-sha (do not move the baseline mid-run).

This also naturally handles the "previously COMPLETED, now re-run" case: a
re-run after completion starts a fresh invocation with `ITERATION_COUNT == 0`
(the prior `.loki/COMPLETED` is removed on the reset path at run.sh:3186), so the
baseline is recaptured at HEAD-of-this-run.

Capture details:
- `git rev-parse HEAD` in `${TARGET_DIR:-.}`; on non-git or failure, write an
  empty file, which the gate treats as inconclusive => pass-through.
- Export `_LOKI_RUN_START_SHA` for the current process so the gate reads it
  without a file round-trip; the file is the durable source of truth across
  resumes.

Pseudocode (illustrative, not final code):

```
local _start_sha_file=".loki/state/start-sha"
mkdir -p ".loki/state"
if [ "${ITERATION_COUNT:-0}" -eq 0 ] || [ ! -s "$_start_sha_file" ]; then
    # Fresh run (or no baseline yet): (re)capture HEAD as the run baseline.
    (cd "${TARGET_DIR:-.}" && git rev-parse HEAD 2>/dev/null) > "$_start_sha_file" 2>/dev/null || true
fi
# else: genuine resume (ITERATION_COUNT > 0) -- keep the existing baseline.
_LOKI_RUN_START_SHA="$(cat "$_start_sha_file" 2>/dev/null || echo "")"
export _LOKI_RUN_START_SHA
```

Edge case: a brand-new repo with zero commits has no HEAD => empty SHA =>
inconclusive => pass-through (do not block a legit first-commit run on a baseline
we never had).

### 4-design.2 -- `council_evidence_gate` (cloned from `council_checklist_gate`)

New function in completion-council.sh, placed immediately after
`council_checklist_gate` (after ~894). Contract identical to the checklist gate:

- `return 0` => gate passes (OK to complete).
- `return 1` => gate blocks (treated by callers as CONTINUE / block-stop).

Behavior:

1. **Knob first (exact-as-today when off).** If `LOKI_EVIDENCE_GATE` (default 1)
   is `0`, `return 0` immediately -- before any file read or write -- so behavior
   is byte-for-byte today's behavior.

   ```
   [ "${LOKI_EVIDENCE_GATE:-1}" = "0" ] && return 0
   ```

2. **Evidence check (a) -- nonzero diff vs run-start SHA (committed UNION working tree).**
   - Resolve base: `_LOKI_RUN_START_SHA`, else `cat .loki/state/start-sha`.
   - If no git repo (`git rev-parse --is-inside-work-tree` fails) => **inconclusive
     => pass-through** (cannot prove fabrication).
   - **Do NOT count committed-only.** Loki's per-iteration auto-commit is NOT
     guaranteed -- run.sh itself guards for this ("Also include unstaged changes
     (in case auto-commit didn't run)", run.sh:9392). A dirty working tree full
     of real edits is legitimate work, not fabrication; committed-only would
     false-block it. Count the UNION of four sources, block only when ALL are
     empty:
       - committed since baseline: `git diff --name-only <base> HEAD`
         (when base is empty/invalid, fall back to `git diff --name-only HEAD`,
         mirroring proof-generator.py's own shallow/first-commit fallback),
       - unstaged: `git diff --name-only HEAD`,
       - staged: `git diff --cached --name-only`,
       - untracked new files: `git ls-files --others --exclude-standard`. A
         greenfield first run creates brand-new files that are not yet committed,
         staged, or visible to `git diff HEAD`; without this fourth source the
         union would be empty and the gate would false-block legitimate new work.
         `--exclude-standard` respects .gitignore so build artifacts and
         node_modules do not count as evidence.
   - The union EXCLUDES any path under `.loki/` (Loki's own runtime state). The
     gate's own inputs live there (`.loki/quality/test-results.json` is always
     present at gate time) and several `.loki/*` files are not gitignored, so
     counting them would make the gate toothless: the union would never be empty.
     Loki's runtime state is not project work / completion evidence.
   - If the union of changed paths is empty => **DIFF EVIDENCE FAILS** (nothing
     shipped anywhere). This strictly reduces false-blocks and cannot let
     fabrication through: if nothing was built, all four sources are empty.

3. **Evidence check (b) -- tests green.**
   - Read `.loki/quality/test-results.json`. Missing or unparseable =>
     **inconclusive => pass-through** for the test dimension (mirrors checklist
     gate's "no file = no gate").
   - `runner == "none"` => **no suite exists => pass-through** for the test
     dimension (the no-suite case is explicitly legitimate).
   - `runner != "none"` AND `pass == false` => **TEST EVIDENCE FAILS**
     (a runner ran and was red).
   - `runner != "none"` AND `pass == true` => test evidence GREEN.

4. **Block decision (truth table, Section 5).** The gate `return 1` (blocks) iff:
   - DIFF EVIDENCE FAILS (empty diff vs run-start, where git+base were
     available), OR
   - TEST EVIDENCE FAILS (a runner actually ran and was red).
   Otherwise `return 0`.

5. **On block:** write `$COUNCIL_STATE_DIR/evidence-block.json` (atomic
   temp+mv, mirroring gate-block.json at ~858-885) with the reason(s), then
   `return 1`. Schema (Section 6).

6. **On pass:** if `$COUNCIL_STATE_DIR/evidence-block.json` exists, `rm -f` it
   (mirrors checklist gate cleanup at ~890-892), then `return 0`. Stale block
   reports must not linger and mislead the dashboard.

Implementation note: like `council_checklist_gate`, do the JSON parse in an
inline `python3 -c` with the file paths passed via env (`_TR_FILE`,
`_START_SHA`) -- never string-interpolated into the script -- matching the
existing safe-parse convention.

### 4-design.3 -- Insertion point A: `council_evaluate`

In `council_evaluate`, immediately AFTER the checklist gate block
(completion-council.sh:1522-1525) and BEFORE the threshold/aggregate computation
(~1527):

```
# Phase 2.5 (v7.19.1): evidence hard gate -- block completion unless there is
# real evidence that files changed AND tests are green.
if ! council_evidence_gate; then
    log_info "[Council] Completion blocked by evidence hard gate"
    return 1  # CONTINUE - cannot complete without real evidence
fi
```

This makes it a hard pre-vote gate, sequenced exactly like the checklist gate:
checklist gate -> evidence gate -> votes -> DA. Members never vote when the
evidence gate blocks, so no LLM cost is spent rubber-stamping a fabricated done.

### 4-design.4 -- Insertion point B: force-review path (parity)

The dashboard force-review path (run.sh:12762-12784) is a second approval path
that writes `COMPLETED` (run.sh:12773) and already gates on
`council_checklist_gate` (run.sh:12766). Add the evidence gate alongside it so
the two approval paths are symmetric:

```
if type council_checklist_gate &>/dev/null && ! council_checklist_gate; then
    log_info "Council force-review: blocked by checklist hard gate"
elif type council_evidence_gate &>/dev/null && ! council_evidence_gate; then
    log_info "Council force-review: blocked by evidence hard gate"
elif type council_vote &>/dev/null && council_vote; then
    ... existing approval (writes COMPLETED) ...
```

Without insertion B, a user could click "force review" on the dashboard to
bypass the evidence gate entirely.

---

## 5. What it blocks / what it must NOT falsely block (truth table)

| Scenario | Diff (start..HEAD) | test-results.json | Gate | Rationale |
|---|---|---|---|---|
| Legit completion: real changes + green tests | nonzero | runner=X, pass=true | PASS | the happy path |
| Greenfield first run: only untracked new files (no commit/stage yet) | nonzero (untracked) | any non-red | PASS | new files are real work; counted via `git ls-files --others --exclude-standard` |
| Fabricated "done", nothing built (not even untracked) | empty | any | **BLOCK** | nothing shipped anywhere |
| Real changes but tests red | nonzero | runner=X, pass=false | **BLOCK** | a runner ran and failed |
| Docs-only change, no test suite | nonzero (docs files) | runner=none, pass=true | PASS | nonzero diff; no suite to fail |
| Project with no test suite, real code | nonzero | runner=none, pass=true | PASS | code shipped; tests not expected |
| No git repo | inconclusive | any non-red | PASS | cannot prove fabrication |
| Empty/missing run-start SHA (new repo, zero commits) | inconclusive | any non-red | PASS | never had a baseline |
| test-results.json missing/unparseable | nonzero | inconclusive | PASS | mirror "no file = no gate" |
| `LOKI_EVIDENCE_GATE=0` | n/a | n/a | PASS (no read/write) | exactly today's behavior |

The only two BLOCK rows are positive fabrication evidence: empty diff, or a
runner that actually ran and was red. Everything inconclusive passes through.

---

## 6. evidence-block.json schema (mirror gate-block.json)

Written to `$COUNCIL_STATE_DIR/evidence-block.json` (`<loki_dir>/council/`),
atomic temp+mv, so the dashboard and handoff can surface *why* a run did not
complete:

```json
{
    "status": "blocked",
    "blocked": true,
    "blocked_at": "2026-06-07T00:00:00Z",
    "iteration": 12,
    "reason": "no_evidence_of_completion",
    "checks": {
        "diff": {"ok": false, "base_sha": "abc123", "files_changed": 0, "sources": "committed|unstaged|staged union empty"},
        "tests": {"ok": true, "runner": "pytest", "pass": true}
    },
    "failures": ["empty git diff vs run-start SHA (nothing shipped)"]
}
```

`reason` is one of `empty_diff`, `tests_red`, or `empty_diff_and_tests_red`.
`failures` is a short human-readable list (cap 5, like gate-block). On gate pass
the file is removed.

---

## 7. Dual-route (Bun/TS) parity

**No Bun change needed.** The live runtime is the bash council (run.sh sources
completion-council.sh and calls council_should_stop). `loki-ts` council.ts is an
unwired port slice. State this in the CHANGELOG NOT-tested section so a future
TS port author knows to mirror `council_evidence_gate` when the TS council is
made live. Adding a TS stub now would be dead code with no runtime to exercise
it.

---

## 8. Tests

New shell test `tests/test-evidence-gate.sh`, following the source-extraction
pattern of `tests/test-pytest-gate-timeout.sh` (awk-extract the function from
completion-council.sh into a minimal harness with stubbed `log_*`,
`COUNCIL_STATE_DIR`, and `ITERATION_COUNT`) OR an end-to-end fixture that sets up
a throwaway git repo + `.loki/quality/test-results.json` and calls the sourced
function directly. Cases:

1. **Empty diff -> blocked.** Repo with run-start SHA == HEAD (no commits since),
   test-results green => `council_evidence_gate` returns 1; `evidence-block.json`
   written with `reason: empty_diff`.
2. **Real diff + green tests -> allowed.** Commit a change after start-sha,
   `runner=pytest,pass=true` => returns 0; no `evidence-block.json` (and a
   pre-existing one is removed).
3. **Real diff + red tests -> blocked.** `runner=pytest,pass=false` => returns 1;
   `reason: tests_red`.
4. **No-test project -> not falsely blocked.** Real diff,
   `runner=none,pass=true` => returns 0.
5. **No git repo -> not falsely blocked.** Run in a non-git dir => returns 0.
6. **Knob off -> behaves as before.** `LOKI_EVIDENCE_GATE=0` with an empty diff =>
   returns 0 and writes NO file.
7. **Stale block cleanup.** Pre-create `evidence-block.json`, then call with
   passing evidence => file removed, returns 0.
8. **Repeat-run baseline recapture.** Simulate a completed prior run (stale
   `.loki/state/start-sha` pointing at an old SHA), invoke `run_autonomous`
   with `ITERATION_COUNT == 0`, commit nothing new => start-sha is recaptured at
   current HEAD and the empty-diff path blocks (proves the gate is not toothless
   on run 2+). With `ITERATION_COUNT > 0` (resume), the baseline is preserved.

Register in `tests/run-all-tests.sh`. Skip gracefully (exit 0 with SKIP) when
`git` or `python3` is unavailable, matching existing test conventions.

---

## 9. CHANGELOG entry (honest) + NOT-tested

Under `## [7.19.1] - <date>`:

```
### Added
- Verified completion / evidence hard gate (default-on, opt out with
  `LOKI_EVIDENCE_GATE=0`). The completion council now refuses to approve STOP
  unless there is real on-disk evidence that the run actually shipped: a nonzero
  git diff between the run-start SHA (newly captured to `.loki/state/start-sha`)
  and HEAD, AND a green test signal from `.loki/quality/test-results.json` where
  a test suite exists. Cloned from the existing `council_checklist_gate` pattern
  and slotted into `council_evaluate` right after the checklist gate, plus the
  dashboard force-review approval path for parity. On block it writes
  `.loki/council/evidence-block.json` (mirroring gate-block.json) so the
  dashboard/handoff can surface why. Attacks the "agent claims done when nothing
  shipped" trust-killer. Blocks only on positive fabrication evidence (empty
  diff, or a runner that actually ran and was red); every inconclusive case (no
  git repo, no baseline, missing/unparseable test results, no test suite,
  docs-only changes) passes through so a legitimate completion is never falsely
  stopped.

### Honest limits / NOT tested
- This gate proves "something changed and tests are green," NOT PRD-semantic
  correctness. Semantic judgment remains with the council votes + Devil's
  Advocate.
- The force-stop safety valves in `council_should_stop` (stagnation, repeated
  done-signals) are deliberately NOT gated: they are resource-protection exits
  that do NOT write the `COMPLETED` marker, so they cannot launder a fabricated
  "done" as an approved completion.
- Bun/TS council (`loki-ts`) is an unwired port slice; the live runtime is bash.
  No TS change shipped. A future TS port must mirror `council_evidence_gate`.
- Not exercised on Windows; relies on POSIX `git`/`python3`.
- Not exercised against shallow clones beyond the `git diff HEAD` fallback path.
```

---

## 10. Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| False block stops a legit run | Med | High | Block only on positive fabrication evidence; all inconclusive => pass; opt-out `LOKI_EVIDENCE_GATE=0`; evidence-block.json explains why so user can act fast |
| Run-start SHA never captured (no git / zero-commit repo) | Med | Med | Empty/missing baseline => inconclusive => pass-through (never block) |
| Run-start SHA reset on resume/pause | Med | High (would zero the diff window) | Capture once; only set `.loki/state/start-sha` if not already present |
| test-results.json stale (PHASE_UNIT_TESTS off, or gate skipped) | Med | Med | Stale/missing => inconclusive => pass; gate runs before council in the same iteration so normally fresh; note freshness dependency |
| Cannot detect "tests were expected" reliably | Med | Med | Use `runner` field: `none` = no suite (pass), non-`none` = a runner ran (its `pass` bool is authoritative). Do NOT infer test expectation from checklist results.json |
| Force-review path bypasses gate | High (without fix) | High | Insertion point B adds evidence gate to run.sh:12762-12784 alongside checklist gate |
| Interaction with checklist gate | Low | Low | Sequenced strictly after it; independent `return 1` semantics; each writes its own block file |
| Interaction with Devil's Advocate | Low | Low | Gate runs pre-vote; DA only runs on unanimous COMPLETE, which the gate can prevent from being reached. DA's own skeptical test/diff checks remain as a second layer |
| Diff against unreachable base (shallow) | Low | Low | Fall back to `git diff --numstat HEAD` (mirrors proof-generator.py); if that also fails => inconclusive => pass |
| Auto-commit makes `git diff HEAD` empty | High | High (would false-block) | Diff = UNION of `base..HEAD` + unstaged + staged; block only when all empty. Run-start baseline (not HEAD working tree) is what makes committed changes count post-auto-commit |
| Stale baseline on repeat runs (gate no-ops on run 2+) | High (without fix) | High (defeats feature) | Recapture start-sha when `ITERATION_COUNT == 0` (fresh run); keep only on genuine resume (`ITERATION_COUNT > 0`) |

---

## Critical files for implementation

- /Users/lokesh/git/loki-mode/autonomy/completion-council.sh  (clone council_checklist_gate -> council_evidence_gate; insertion A in council_evaluate)
- /Users/lokesh/git/loki-mode/autonomy/run.sh  (run-start SHA capture in run_autonomous; insertion B in force-review path)
- /Users/lokesh/git/loki-mode/tests/test-evidence-gate.sh  (new test, pattern from test-pytest-gate-timeout.sh)
- /Users/lokesh/git/loki-mode/CHANGELOG.md  (honest entry + NOT-tested)
- /Users/lokesh/git/loki-mode/tests/run-all-tests.sh  (register new test)
