# Evidence Receipt: honest build applicability (done once, for all customers)

## The defect (founder-reported, 2026-07-03)

A sophisticated user built a Node CLI (invoice tool) and saw the Evidence Receipt say
"Build: not run - build not run", "Live health check: not captured", "No screenshot",
headline "Working, with gaps". He concluded *nothing was validated* -- and that read is the
bug: it destroys trust in genuinely-validated work.

## Ground truth (from the persisted proof of run-20260703034719)

- `facts.tests` = `verified`, exit 0 -- tests GENUINELY ran and passed.
- `facts.security` = `ran:true, clean` -- security GENUINELY ran, real clean scan.
- `facts.build` = `ran:false, status:not_run, command:""` -- and this is counted as a GAP
  by `_compute_degraded` (proof-generator.py:903-908), which forces "WITH GAPS".
- `facts.quality_gates` = `[]` (EMPTY) -- verify.sh's deterministic build gate does NOT
  reach the SaaS proof.

## Root cause (systemic, every build, every customer)

`build-results.json` has **NO writer**. `_collect_build` (proof-generator.py:319) reads
`.loki/quality/build-results.json`, which nothing ever writes, so `facts.build` defaults to
`not_run` (line 333) for EVERY build. The sibling collectors DO have writers in run.sh
(`enforce_static_analysis` 8357, `run_secure_scan` 8720, `enforce_test_coverage` 9170 ->
test-results.json / security-findings.json / static-analysis.json). Build was never wired.

So "Build: not run" is not a per-project decision -- it is a permanent, structural
mislabel. A build that has no build step (a CLI) and a build that genuinely skipped a real
build both read identically as "not run" and both count as a gap.

## The fix (honest, no fabrication, no mocking -- the founder's hard constraint)

Add the missing writer as a sibling collector, with a POSITIVE applicability determination:

1. **run.sh `enforce_build_check()`** (new, sibling to enforce_static_analysis): detect the
   stack's build command (npm "build" script / go build / cargo build -- same detectors as
   verify.sh:250-291). Then:
   - build command exists -> RUN it -> record `{ran:true, command, exit_code, status:
     verified|failed}`.
   - genuinely NO build step for this stack -> record `{ran:false, applicable:false,
     status:not_applicable, command:""}`. This is a POSITIVE "no build phase" signal, not an
     absent file.
   Write `.loki/quality/build-results.json`. Call it where the sibling collectors are called.

2. **proof-generator.py `_collect_build`**: pass through `applicable:false` -> status
   `not_applicable`. CRITICAL: `not_applicable` ONLY from an explicit `applicable:false` in
   the file. File ABSENT or unknown status -> `not_run` (honest gap), NEVER N/A. Absent must
   never mean N/A (that would flip un-built projects green -- fake-green).

3. **proof-generator.py `_compute_degraded`**: `not_applicable` is NOT a gap (excluded from
   the weak set for build). A failed/skipped/not_run build stays a gap. Mirror-image guard:
   a real build script that FAILS still emits `failed` -> gap -> never green.

4. **proof-generator.py status normalizer (line 215)**: `not_applicable` passes through.

5. **SPA adapter.ts**: `not_applicable` renders "N/A - no build step for this stack" (a
   neutral/pass tier, NOT a warn/gap), distinct from "not run".

## Anti-fake-green invariants (the trust moat)

- N/A ONLY on positive `applicable:false`. Absent file / unknown status -> honest gap.
- A build script that got skipped or failed STILL shows a gap and CANNOT go green.
- The directional regression test (write FIRST): Node project WITH a build script whose
  build FAILS -> `facts.build=failed` -> degraded -> headline NOT green. And: build gate
  MISSING -> still a gap, not N/A.
- N/A does not gate the seal any more than a gap does; it is honest applicability, not a pass.

## Measurable before/after

Before: invoice CLI -> "Build: not run", headline "Working, with gaps".
After: fresh invoice CLI build -> "Build: N/A (no build step for this stack)", tests passed,
security scanned-clean, headline VERIFIED (no un-closed applicable gaps).

## Constraints

Verdict-core: council 3/3 + local-ci + 14 version locations. Verify via
function-extraction/golden harness -- NEVER run the engine build from the worktree. Run the
EXISTING tests/test_proof_generator.py green before AND after (it has build-results.json
fixtures that this change touches). Deploy to live SaaS is `bun install -g loki-mode@<ver>`.
