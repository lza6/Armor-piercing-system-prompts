# Complete and efficient failure collection

## Collect one exact attempt

After the workflow attempt is terminal:

```bash
node .agents/skills/happier-ci-stabilize/scripts/collect-actions-failures.mjs \
  --repo happier-dev/happier \
  --run-id <run-id>
```

For a specific rerun attempt:

```bash
node .agents/skills/happier-ci-stabilize/scripts/collect-actions-failures.mjs \
  --repo happier-dev/happier \
  --run-id <run-id> \
  --attempt <attempt-number>
```

The collector writes under `/tmp` by default:

- `summary.json`: exact run identity, all job conclusions, failed steps, and compact excerpts;
- `jobs/<job-id>.log`: complete log for every failed, cancelled, timed-out, or action-required job.

Use `--out <absolute-directory>` only when another temporary location is needed. Do not write diagnostic logs into the repository.

If `collectionComplete` is false, treat the output as a progress snapshot, not the complete failure set. Wait for independent jobs with a 5-20 minute cadence unless a named stop condition applies.

When push concurrency uses `cancel-in-progress: false`, later SHAs queue instead of destroying the evidence of an active run. Let the oldest useful run reach terminal collection when it can still reveal previously unknown clusters. After its complete failure set is retained, cancel only a superseded run that is already proven unable to satisfy the requested outcome and is blocking the corrected exact SHA; do not accumulate a queue of known-doomed full runs.

## Read compactly without losing evidence

1. Start from `summary.json`, not raw run-level output.
2. Remove aggregator-only failures from the causal count while retaining them in the inventory.
3. Group identical stack origins, failed commands, error codes, and stale generated-output messages.
4. Open the full retained log only for each distinct causal cluster.
5. Record the earliest causal error in a job; teardown and aggregate exit-code messages are usually propagation.

GitHub annotations are useful indexes but can omit child-process stderr, collapse repeated failures, or show only one test. The full job log remains deciding evidence. When a workflow uploads a richer failure artifact, inspect it only when the log does not decide the cause; avoid downloading unexpectedly huge artifacts before checking their metadata.

For Playwright, `error-context.md`, the trace, screenshots, and boundary-process logs can distinguish an absent product result from a stale locator or fixture mode. Inspect the smallest deciding artifact first. A DOM snapshot that shows a healthy surface in the wrong selected mode is test setup evidence, not a reason to raise the wait.

## Compare with the last known green basis

For deterministic failures, compare the failing SHA with the most recent green SHA and identify changes in:

- the failing production owner;
- its canonical tests and testkit;
- shared fixtures/generators;
- workflow lane inputs, runner image, action versions, and environment;
- external dependencies or published artifacts.

A previous green run on a different SHA proves the old basis, not the new commit set. Conversely, dozens of passing sibling jobs on the failing SHA narrow the affected corridor and should be preserved as evidence.

## Do not serially rediscover downstream errors

Before dispatching another full run:

- reproduce every collected deterministic cluster locally where feasible;
- run all corrected focused suites together;
- run every affected package/lane locally or through a focused manual workflow;
- inspect skipped downstream jobs and determine whether they were unreachable or unnecessarily coupled;
- preflight release configuration and generated outputs that do not require publication credentials.

One run cannot expose a job whose real input artifact was never produced. Use immutable-candidate resume after fixing control/validation bytes so downstream validation can execute without rebuilding already verified products. Do not rerun the whole graph merely to retry a terminal, independent failing lane.
