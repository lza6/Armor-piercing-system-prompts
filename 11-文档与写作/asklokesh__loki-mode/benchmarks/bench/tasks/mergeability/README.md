# Scored mergeability benchmark (Rank 12)

The accuracy-proof instrument for loki change-mode output. Where the SWE-bench
route asks a binary "did the one test flip?", this route asks the reviewer's
real question -- MERGEABILITY: "would a maintainer merge this change?" -- and
answers it with a SCORE, not a boolean.

This is not a fork of the R2 harness. It EXTENDS `benchmarks/bench/runner.py` +
`bench_schema.py`: each task validates against the FROZEN
`validate_task_spec`, the fixture/prompt/held-out-overlay flow is the runner's,
and the rubric rides in `task.json` as an extra `rubric` key that the frozen
validator ignores but `compute_task_hash` folds into the reproducibility anchor.

## Scoring model (benchmarks/bench/mergeability_score.py)

```
score = 0                                   if ANY blocker check fails
      = (sum of PASSED non-blocker weights) / (sum of ALL non-blocker weights)
                                            otherwise, a value in [0, 1]
```

- A maintainer BLOCKS on some things (broken build, missing FAIL_TO_PASS test,
  a security or correctness regression) -> a failed blocker gates the score to 0.
- A maintainer DOCKS points on others (a missing edge case, a shared reference,
  a weak fallback) -> weighted non-blockers give partial credit.

`score_from_results(rubric, check_results)` is a PURE function of the maintainer
rubric and a held-out grader's per-check booleans. It structurally refuses to
read the adapter-output, the council verdict, or any loki self-report (it rejects
a `check_results` dict carrying a forbidden judgment key). That is what makes
"Loki NEVER grades itself" true on this route too: the only inputs are the
offline rubric and a grader that ran held-out checks OUTSIDE the agent.

## Credibility invariants (inherited + extended)

1. Held-out grader. `acceptance/rubric.py` is never shown to the agent; the
   runner overlays it into the workdir only AFTER the agent finishes, then it
   emits `{"checks": {id: bool}}`. The scorer runs it itself (the runner's
   `grade()` discards stdout) and scores that JSON.
2. Reverse-classical FAIL_TO_PASS. Each task's blocker `fail_to_pass` FAILS on
   the base fixture (RED) and PASSES only after the reference correct change
   (GREEN). A blocker that already passed on base would prove nothing (the exact
   trap `autonomy/completion-council.sh:1552 _loki_test_provenance` guards). Both
   shipped tasks' non-vacuity is proven in `tests/test_bench_mergeability.py`.
3. CI-safe + deterministic. Grader and scorer are pure stdlib, no network, no
   install, no tokens. The scorer is EXACTLY reproducible: two runs on the same
   input are byte-identical (tolerance is only for real-engine stochasticity).
4. Anti-tamper. An agent that overwrites the in-workdir grader cannot inflate its
   score: the held-out overlay restores the real grader after the agent runs.

## Tasks

| id | blocker (FAIL_TO_PASS) | weighted non-blockers |
|---|---|---|
| `slugify-empty-crash` | empty/whitespace/None -> `"untitled"` (base: `""` or crash) | preserves-existing(4), collapses-separators(3), strips-result(2), idempotent(1) |
| `config-merge-mutation` | `deep_merge` does not mutate caller's base (base: mutates in place) | correct-result(4), no-shared-refs(3), override-untouched(2), handles-new-keys(1) |

The rubrics are authored from a maintainer's "what would I block on" view,
independent of loki's strengths -- the benchmark is not tuned to flatter loki.

## Run it

```
# score an adapter on a task (real adapter spends tokens; mock in tests does not)
python3 benchmarks/bench/mergeability_score.py score \
  benchmarks/bench/tasks/mergeability/slugify-empty-crash/task.json \
  --adapter loki --trials 2

# validate a rubric
python3 benchmarks/bench/mergeability_score.py validate-rubric \
  benchmarks/bench/tasks/mergeability/slugify-empty-crash/task.json

# CI-safe unit + integration proof (mocked adapter, no tokens)
python3 -m pytest -q tests/test_bench_mergeability.py
```

## Honest baseline

The INSTRUMENT is proven end-to-end with mocked adapters:

- reference-correct change   -> 100% mergeable, no blocker (both tasks)
- no-op change (bug remains) -> 0% mergeable, blocker fails (both tasks)
- partial change (blocker cleared, a non-blocker regressed) -> 0.7 on slugify

A REAL headline "loki is X% mergeable vs Anthropic Code Review" requires a real
engine run on this suite (tokens, an engine invocation) and a parallel run of the
competitor via an adapter. That number is deliberately NOT fabricated here. What
is proven now: the scorer discriminates (high for mergeable, 0 for unmergeable,
partial in between) and is exactly reproducible across runs and machines.
