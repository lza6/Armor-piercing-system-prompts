# Uncertainty-Gated Escalation (Loki Mode v7.19.2)

Design only. No implementation code lands with this document. Every hook point
below was read from live source; line numbers drift, so the verified anchors in
section 1 are the contract a dev re-confirms before editing.

## Goal

When Loki is likely stuck or thrashing, escalate to the human PROACTIVELY via
the EXISTING pause + notify + handoff machinery, instead of silently burning
iterations until max-iterations. No new metacognition: reuse three proxy signals
that already exist. Escalate only when at least two of the three co-occur for N
consecutive rounds. Default-on, opt-out with LOKI_UNCERTAINTY_ESCALATION=0,
byte-identical when off.

## Architectural spine: split DECISION from ACTION

This is the load-bearing decision and everything else falls out of it.

- DECISION: a pure-ish function `uncertainty_should_escalate` lives in
  `autonomy/completion-council.sh` next to the other `council_*` state helpers.
  It reads ONLY persisted state (`state.json`, `convergence.log`, and its own
  `.loki/state/uncertainty.json`), mutates only its own state file, and returns
  rc 0 (escalate now) / rc 1 (do not). It fires NO notifications and touches NO
  PAUSE file. This makes it sourceable and testable exactly like
  `council_evidence_gate` (completion-council.sh:907): a test writes a fake
  `state.json` into a throwaway dir and asserts the return code, with zero real
  side effects on the developer's machine.
- ACTION: the run.sh call site (new region right after
  `council_track_iteration`, run.sh:12389-12391) interprets rc 0 and performs
  the side effects: loud terminal line, `write_structured_handoff`,
  `notify_intervention_needed`, write a `signals/UNCERTAINTY_ESCALATION` marker,
  and `touch .loki/PAUSE`. It also emits the perpetual-mode honesty line.

Consequence: the two code slices live in DIFFERENT files (decision in
completion-council.sh, action in run.sh), so the dev fleet can build them in
parallel without collision.

---

## 1. Verified hook points (read from live source)

All paths relative to repo root `/Users/lokesh/git/loki-mode`.

### Proxy 1 - circuit-breaker no-change counter
- Var declared: `autonomy/completion-council.sh:70` (`COUNCIL_CONSECUTIVE_NO_CHANGE=0`).
- Incremented: `completion-council.sh:178`; reset: `:180`. Driven by a combined
  hash of `git diff --stat HEAD` + staged diff + last commit hash
  (`:165-182`).
- Limit knob: `COUNCIL_STAGNATION_LIMIT` (`:56`, default 5).
- Persisted: written into `state.json` as `consecutive_no_change`
  (`completion-council.sh:232` -> `:237` json.dump). THIS is what the decision
  function reads (not the live shell var, which is out of scope in a sourced
  test).
- Updated every iteration via `council_track_iteration` (run.sh:12390).

### Proxy 2 - file-churn oscillation / reverts
- Existing data: `convergence.log` is appended at
  `completion-council.sh:215` with line format
  `timestamp|iteration|files_changed|consecutive_no_change|done_signals`.
  CRITICAL: `files_changed` (`:208`) is a COUNT
  (`git diff --name-only HEAD | wc -l`), NOT file identities. A count cannot
  detect "same files back and forth."
- The combined diff hash exists at `completion-council.sh:175`
  (`combined_hash`), persisted only transiently in the shell var
  `COUNCIL_LAST_DIFF_HASH` (`:73`, `:182`) - immediate-repeat only.
- DECISION (see section 5 limits): proxy 2 is implemented as DIFF-HASH
  RECURRENCE-AT-DISTANCE. We persist a small ring buffer (last
  ~6 hashes) of `combined_hash` in `uncertainty.json`. Proxy 2 fires when the
  current hash equals a hash seen 2+ rounds back (A -> B -> A pattern). The
  immediate repeat (A -> A) is already proxy 1, so recurrence-at-distance is the
  genuine oscillation/revert signal. This is a tiny, justified addition (one
  bounded array in an existing JSON file), NOT heavy new tracking. The hash to
  read is the same `combined_hash` proxy 1 already computes; the decision
  function recomputes it cheaply from `git diff --stat HEAD` or, preferably,
  `council_track_iteration` writes it into `state.json` (`last_diff_hash`) so the
  decision function stays pure (no git calls). See slice A for which.

### Proxy 3 - persistent council split
- approve_count computed in `council_vote` (`completion-council.sh:270`,
  tallied `:388`, anti-sycophancy adjust `:417`).
- effective_threshold: `completion-council.sh:293`
  (`(COUNCIL_SIZE * 2 + 2) / 3`, the ceiling(2/3) formula).
- Persisted: each council round appends to `state['verdicts']`
  (`completion-council.sh:449-455`) with keys `iteration`, `timestamp`,
  `approve`, `reject`, `result` (`APPROVED`/`REJECTED`). NOTE: threshold is NOT
  stored. That is fine: `result == "REJECTED"` already encodes
  `approve < threshold`. A split round = `result == "REJECTED" AND approve >= 1`
  (council could not converge: at least one approver, still short of threshold).
  Do NOT go looking for a stored threshold; it is not there by design.
- CADENCE: `verdicts` only appends when the council actually VOTES, which is
  every `COUNCIL_CHECK_INTERVAL` OR when the circuit breaker forces a vote
  (`council_should_stop`, completion-council.sh:2045-2051; circuit check
  :2039-2043). So proxy 3 is STALE between votes. This is acceptable because in
  the stuck regime we care about, proxy 1 going hot
  (`consecutive_no_change >= COUNCIL_STAGNATION_LIMIT`) is exactly what TRIPS the
  circuit breaker (`council_circuit_breaker_triggered`,
  completion-council.sh:252) and forces a council vote, which refreshes proxy 3.
  Verified: `council_should_stop` sets `should_check=true` when
  `circuit_triggered=true` (:2047-2048). Document the between-votes staleness as
  a known limit (section 5).

### notify_intervention_needed
- `autonomy/run.sh:2328`. Signature: `notify_intervention_needed "$reason"`;
  thin wrapper over `send_notification "Intervention Needed" "$reason"
  "critical"`.

### PAUSE consume / clear path (perpetual-mode crux)
- Consumer: `check_human_intervention` (run.sh:12701), PAUSE branch
  `:12708`.
- Perpetual auto-clear: `:12711-12730`. In perpetual mode PAUSE is
  auto-cleared (`:12727 rm -f`) and `notify_intervention_needed` STILL fires
  (`:12726`). Only `BUDGET_EXCEEDED` (`:12712`) is carved out from
  auto-clear.
- Non-perpetual: PAUSE triggers `handle_pause` (run.sh:12842) and waits
  (`:12732-12742`).
- Consumed once per loop turn from the main loop: `check_human_intervention`
  is called at run.sh:11528, return-code switch `:11530-11533`
  (1 = restart loop, 2 = stop).
- IMPLICATION: escalation only WRITES PAUSE. The existing consumer halts (or, in
  perpetual mode, auto-clears + notifies). Perpetual degrade is therefore FREE -
  no new consumer logic. We detect perpetual at OUR site using the same vars
  (`AUTONOMY_MODE` / `PERPETUAL_MODE`, run.sh:12711) only to print the honest
  "notify-only; PAUSE will not halt this run" line.

### write_structured_handoff
- `autonomy/run.sh:8816`. Verified single live definition (the
  "active definition is below" comment at :8811 refers to
  `load_handoff_context`, not a second handoff def; grep shows one
  `write_structured_handoff()`). Signature:
  `write_structured_handoff "$reason"`; writes
  `.loki/memory/handoffs/<ts>.json` + `.md`.

### Loop point for the escalation check
- Slot the ACTION immediately AFTER `council_track_iteration` in the main loop:
  run.sh:12388-12391. At this point proxy 1 and proxy 2 are freshly written for
  this iteration, and proxy 3 is fresh exactly when it matters (circuit-forced
  vote). This is BEFORE the completion-promise / council checks
  (run.sh:12408+), so escalation is evaluated every iteration.

### Mirror precedent (action shape)
- Gate-escalation block run.sh:12308-12318 is the precedent to clone: write a
  `signals/` marker (`:12310`), call a handoff hook with its own opt-out
  (`:12314`), then `touch .loki/PAUSE` (`:12317`). Our action mirrors this with
  `write_structured_handoff` + `notify_intervention_needed` +
  `signals/UNCERTAINTY_ESCALATION` + `touch .loki/PAUSE`.

---

## 2. Escalation decision function design

### Inputs (all read from persisted state, no live shell vars)
1. `p1` = proxy 1 hot: from `state.json.consecutive_no_change`. Hot when
   `>= LOKI_UNCERTAINTY_NOCHANGE_MIN` (default = `COUNCIL_STAGNATION_LIMIT` - 1,
   i.e. "approaching circuit-breaker"). Reading slightly below the breaker limit
   lets us escalate BEFORE the breaker forces an end-state.
2. `p2` = proxy 2 hot: diff-hash recurrence-at-distance. Hot when the current
   `last_diff_hash` matches a hash at distance >= 2 in the ring buffer.
3. `p3` = proxy 3 hot: persistent split. Read the last `K` entries of
   `state.json.verdicts`; count consecutive trailing rounds where
   `result == "REJECTED" AND approve >= 1`. Hot when that run length
   `>= LOKI_UNCERTAINTY_SPLIT_ROUNDS` (default 2).

### Co-occurrence + N-round debounce
- Per round (= per iteration; "round" is defined as one main-loop iteration),
  compute `hot_count = p1 + p2 + p3`.
- `co_occur = (hot_count >= 2)`.
- Maintain `consecutive_co_occur` in `uncertainty.json`:
  - if `co_occur`: increment; else reset to 0.
- Escalate (rc 0) when `consecutive_co_occur >= LOKI_UNCERTAINTY_ROUNDS`
  (the N knob, default 2; recommended range 2-3) AND not already escalated this
  episode (debounce flag, below).
- A single noisy proxy can NEVER escalate alone (requires hot_count >= 2).

### Debounce (escalate once per stuck-episode)
- `uncertainty.json` carries `escalated_episode: true|false`.
- On escalate, set `escalated_episode = true` and record
  `escalated_at_iteration`.
- Suppress re-fire while `escalated_episode == true`.
- RE-ARM (reset `escalated_episode = false` and `consecutive_co_occur = 0`) when
  `co_occur` becomes false in any later round (a proxy cleared => the episode is
  considered resolved; a new stuck episode may legitimately re-escalate). State
  the reset condition explicitly so a dev does not "helpfully" keep it latched.

### State persistence
- File: `.loki/state/uncertainty.json` (singular; the `uncertainty-*.json` glob
  in the brief maps to this one file - keep it single to avoid an unbounded
  directory). Schema:
  ```json
  {
    "schema_version": "1.0.0",
    "consecutive_co_occur": 0,
    "escalated_episode": false,
    "escalated_at_iteration": 0,
    "diff_hash_ring": ["<h>", "<h>", "..."],
    "last_round_iteration": 0,
    "last_proxies": {"p1": false, "p2": false, "p3": false}
  }
  ```
- Ring buffer bounded to 6 entries (constant). All writes atomic temp+mv,
  mirroring evidence-block.json (`completion-council.sh:1059-1086`).

### Knob-first byte-identical guard
First line of `uncertainty_should_escalate`, BEFORE any read or write:
```
[ "${LOKI_UNCERTAINTY_ESCALATION:-1}" = "0" ] && return 1
```
(rc 1 = do-not-escalate; mirrors `council_evidence_gate`'s knob-first guard at
completion-council.sh:909). When off: zero file reads, zero writes, zero state
file creation => byte-identical.

### Knobs summary (all opt-out / tunable, none required)
- `LOKI_UNCERTAINTY_ESCALATION` (default 1) - master on/off.
- `LOKI_UNCERTAINTY_ROUNDS` (default 2) - N consecutive co-occurrence rounds.
- `LOKI_UNCERTAINTY_NOCHANGE_MIN` (default `COUNCIL_STAGNATION_LIMIT - 1`) - p1
  threshold.
- `LOKI_UNCERTAINTY_SPLIT_ROUNDS` (default 2) - p3 split run length.

---

## 3. Disjoint dev slices (parallel-safe)

Binding constraints for EVERY slice: no version bumps (do not touch VERSION /
CHANGELOG), no git commits, no emojis, no em-dashes or en-dashes (ASCII hyphen
only), atomic temp+mv for all state writes, knob-first opt-out where the slice
touches the hot loop.

### Slice A - decision function + state schema (completion-council.sh)
- Region: add `uncertainty_should_escalate` and a tiny
  `_uncertainty_read_state` / `_uncertainty_write_state` pair near the other
  `council_*` state helpers (after `council_circuit_breaker_triggered`,
  i.e. around completion-council.sh:265, BEFORE `council_vote` at :270).
- Also add ONE line inside `council_track_iteration` to persist
  `state['last_diff_hash'] = combined_hash` (extend the python block at
  completion-council.sh:224-238 by adding the env var + one assignment) so the
  decision function reads the hash from state.json and stays pure (no git in the
  decision path). This is the only edit inside an existing function; keep it to a
  single key add to minimize collision with run.sh slice.
- Owns: `.loki/state/uncertainty.json` schema, ring buffer, co-occurrence +
  debounce logic, all four knobs' defaults.
- File-region disjoint from slice B (different file).

### Slice B - action + wiring (run.sh)
- Region: new block right after `council_track_iteration` call
  (run.sh:12389-12391).
- Logic:
  ```
  if type uncertainty_should_escalate >/dev/null 2>&1 && uncertainty_should_escalate; then
      # loud line (section 6), write_structured_handoff "uncertainty_escalation",
      # notify_intervention_needed, signals/UNCERTAINTY_ESCALATION marker,
      # touch .loki/PAUSE, perpetual honesty line.
  fi
  ```
- Clone the GATE_ESCALATION shape (run.sh:12308-12318) for marker + handoff +
  touch ordering.
- Perpetual detection: read `AUTONOMY_MODE` / `PERPETUAL_MODE`
  (same as run.sh:12711) ONLY to print the honest notify-only line.
- File-region disjoint from slices A, C, D.

### Slice C - tests (tests/test-uncertainty-escalation.sh)
- New file. Sources the real `uncertainty_should_escalate` from
  completion-council.sh, stubs `log_*`, runs per-case throwaway dirs. Models
  tests/test-evidence-gate.sh exactly. Asserts decision-only (no real notify /
  no real PAUSE because it calls the DECISION function, not the run.sh action).
- File-region disjoint (new file).

### Slice D - docs + knob registration
- Register the four knobs in the config-comment block (the env-var doc region
  around run.sh:91-128 and the yaml mapping near :282/:424) and
  `autonomy/config.example.yaml`. Add a short section to the user-facing docs.
- Keep edits to comment / config blocks; do not touch the hot loop. If this
  collides with slice B's run.sh edits, sequence D after B (the only soft
  dependency). Otherwise fully disjoint.

Recommended parallelism: A, C, D in parallel; B after A's function signature is
agreed (C can mock the signature meanwhile). 4 slices, 3 files + 1 new test +
docs.

---

## 4. Test plan (model: tests/test-evidence-gate.sh)

Harness: source the real completion-council.sh with `log_*` stubbed; call
`uncertainty_should_escalate` inside per-case `mktemp -d` dirs, each writing its
own `.loki/state/uncertainty.json` + `.loki/council/state.json` +
`.loki/council/convergence.log`. Assert BOTH rc and the mutated
`uncertainty.json` side effects. Loud SKIP (exit 0) if the function is not yet
defined (mirrors evidence-gate's absent-impl banner). Each case sets
`COUNCIL_STATE_DIR` and `ITERATION_COUNT` explicitly.

Cases:
1. PROXY READ - p1 only hot: `consecutive_no_change` >= min, hash unique,
   verdicts approved. Assert `last_proxies.p1 == true`, others false, rc 1
   (NO escalate on 1 proxy). Proves proxy 1 is read.
2. PROXY READ - p2 only hot: write a recurrence-at-distance hash ring
   (A,B,A), unique p1/p3. Assert `p2 == true`, rc 1. Proves proxy 2 is read
   from the ring, and that immediate-repeat (A,A) does NOT count as p2.
3. PROXY READ - p3 only hot: verdicts trailing K = REJECTED with approve>=1 for
   SPLIT_ROUNDS rounds. Assert `p3 == true`, rc 1. Proves proxy 3 reads
   `result`/`approve` (and does NOT require a stored threshold).
4. CO-OCCURRENCE x N escalates: set p1 + p3 hot for N consecutive calls
   (loop the function N times, advancing iteration). Assert rc 0 on the Nth
   call, `escalated_episode == true`. Proves >=2-for-N escalates.
5. 1-PROXY-NEVER: keep only one proxy hot for many rounds. Assert rc 1 every
   round, `escalated_episode == false`. Proves a single noisy proxy cannot
   escalate.
6. DEBOUNCE (no re-fire): after case-4 escalation, call again with the SAME hot
   proxies. Assert rc 1 (suppressed) while `escalated_episode == true`. Proves
   escalate-once-per-episode.
7. RE-ARM: after escalation, feed one round with co_occur false (clear a proxy),
   assert `escalated_episode == false` + `consecutive_co_occur == 0`; then feed
   N hot rounds again, assert rc 0. Proves reset-on-clear and re-escalation of a
   new episode.
8. OPT-OUT BYTE-IDENTICAL: `LOKI_UNCERTAINTY_ESCALATION=0`. Assert rc 1 AND that
   `.loki/state/uncertainty.json` is NOT created / NOT modified (snapshot the
   dir before/after; mtime + existence). Proves byte-identical when off.
9. PERPETUAL DEGRADE-TO-NOTIFY: this is a run.sh ACTION behavior, so test it as a
   thin integration shim: stub `notify_intervention_needed`, `handle_pause`,
   `handle_dashboard_crash` to record calls; set `AUTONOMY_MODE=perpetual`;
   `touch .loki/PAUSE`; call the real `check_human_intervention`
   (run.sh:12701). Assert PAUSE is auto-cleared AND notify was called (proves
   the degrade path is the EXISTING consumer at run.sh:12725-12727, so escalation
   degrades to notify-only under perpetual). This case sources run.sh's
   `check_human_intervention` with its deps stubbed, or asserts via a focused
   harness; if sourcing run.sh wholesale is impractical, assert the contract by
   reading the consumer branch and documenting it as a code-path test.

All cases: throwaway git repos isolated via `GIT_CONFIG_GLOBAL=/dev/null`
(mirror test-evidence-gate.sh:107-115). Skip-not-fail on missing git/python3.

---

## 5. Honest limits

- PERPETUAL-MODE = NOTIFY-ONLY. If Loki runs in perpetual / auto-continue mode,
  the existing consumer (`check_human_intervention`, run.sh:12725-12727)
  auto-clears PAUSE and continues. Escalation therefore DEGRADES to a
  notification (notify still fires) plus a handoff doc; it does NOT halt the run.
  We detect this at the action site and print it honestly. We deliberately do
  NOT add a no-auto-clear carve-out for our marker (the BUDGET_EXCEEDED carve-out
  at run.sh:12712 shows it is technically possible) because that is scope creep
  and would break "byte-identical when off." Out of scope for v7.19.2; candidate
  follow-up.
- PROXY 2 IS COUNT-BLIND BY ORIGIN. `convergence.log` stores `files_changed` as
  a count (completion-council.sh:208), not identities, so it cannot by itself see
  "same files back and forth." We approximate oscillation with diff-hash
  recurrence-at-distance, which catches A -> B -> A state cycling but CANNOT
  distinguish a genuine revert from a coincidental return to an identical tree
  state, and will MISS oscillation that changes content each pass (hash differs
  every round). It is a heuristic, not a true revert detector.
- PROXY 3 STALENESS BETWEEN VOTES. The verdicts array only updates on actual
  council votes (every `COUNCIL_CHECK_INTERVAL` or circuit-forced). Sampled every
  iteration, p3 can be stale between votes. We rely on the circuit-breaker
  coupling (proxy 1 hot forces a vote, refreshing p3) so p3 is fresh exactly in
  the regime we escalate on; outside that regime p3 may lag by up to
  `COUNCIL_CHECK_INTERVAL` iterations.
- PROXIES FALSE-FIRE AND MISS. All three are heuristics. A legitimately hard
  refactor that produces no net diff for several rounds while the council
  remains split can false-fire; a fast-thrashing failure that keeps changing
  different files with shifting hashes can be missed. Requiring >=2 co-occurring
  for N rounds reduces, but does not eliminate, false fires. The cost of a false
  fire is bounded: one notification + one handoff + one PAUSE (auto-cleared in
  perpetual), opt-out at the site.
- THESE ARE PROXIES, NOT TRUE METACOGNITION. The system does not know it is
  stuck; it infers stuckness from three correlated symptoms of stuckness. There
  is no model of confidence, no self-estimate of progress. This is intentional
  (no new metacognition) and is the honest ceiling on what this feature can
  claim.

---

## 6. Rails (the v7.19.1 evidence-gate rails, mirrored)

A default-on hook in the hot loop must be bounded, loud, and self-rescuing.

- BOUNDED: the decision function does O(1) work - reads two small JSON files,
  scans the last K verdicts and a 6-entry ring. No git subprocess in the decision
  path (hash comes from state.json via slice A's one-line add). No network. No
  unbounded loop. Cannot hang. The action runs at most ONCE per stuck episode
  (debounce), not every iteration.
- LOUD TERMINAL LINE at the escalation site (run.sh, slice B):
  ```
  log_error "[Uncertainty] Escalating to human: >=2 of 3 stuck-signals co-occurred for N rounds (no-change / oscillation / council-split). PAUSE written; handoff saved."
  log_warn  "[Uncertainty] To opt out of proactive escalation: set LOKI_UNCERTAINTY_ESCALATION=0"
  ```
  And, only when perpetual, the honesty line:
  ```
  log_warn  "[Uncertainty] Perpetual mode: PAUSE will be auto-cleared; this is notify-only and will NOT halt the run."
  ```
- OPT-OUT NAMED AT THE SITE: the opt-out env var is printed on the line above,
  right where escalation happens, so a terminal user with no dashboard can
  self-rescue in one step (mirrors completion-council.sh:1055).
- KNOB-FIRST: `LOKI_UNCERTAINTY_ESCALATION=0` short-circuits the decision
  function before any read/write (section 2), and `type ... >/dev/null` guards
  the run.sh call so an unbuilt function is a silent no-op. Byte-identical when
  off, proven by test case 8.
