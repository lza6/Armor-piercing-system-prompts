---
name: loom-triggers
description: Wait for an external fact through CodexLoom and resume the same long-lived Agent when it changes. Use when work is blocked on a GitHub pull request, workflow run, deployment, approval, artifact, or another provider-owned state; when polling or sleeping would keep a Turn open; when a Schedule would only guess when a condition changes; or when an Agent receives an external_trigger envelope. Do not use for time-based recurring work or human decisions.
---

# Loom Triggers

Use a Trigger when existing work cannot continue until a specific external fact changes. A Trigger is a durable wake-up condition for the same long-lived Agent, not a new task and not authoritative proof that the work is ready.

Do not keep a Turn alive with `sleep`, shell loops, repeated API reads, or periodic self-messages. Do not replace an external condition with a Schedule. Schedules answer "when should this run?"; Triggers answer "which external fact should wake this work?"

## Arm a condition

Inspect the configured sources first:

```bash
loom trigger source list
```

GitHub pull request examples:

```bash
loom trigger add github pull-request OWNER/REPO#1970 \
  --from AGENT \
  --on merged \
  --resume "Fetch the target branch, verify the expected contract is present, then continue the original delivery flow." \
  --expires 14d

loom trigger add github pull-request OWNER/REPO#1971 \
  --from AGENT \
  --on head-changed \
  --expect-head FULL_SHA \
  --resume "Re-read the pull request HEAD and invalidate evidence tied to the previous candidate." \
  --expires 14d
```

GitHub workflow run example:

```bash
loom trigger add github workflow-run OWNER/REPO/RUN_ID \
  --from AGENT \
  --on success \
  --on failure \
  --resume "Read the current run and required checks for the candidate SHA, then decide whether the original work can proceed." \
  --expires 2d
```

The resume instruction must name the first authoritative read and the original next step. Capture exact provider identifiers and immutable candidate facts where available. GitHub sources are scoped by Resource Owner; Loom normally selects the source whose scope matches `OWNER/REPO`. Pass `--connection CONNECTION_ID` only when multiple sources intentionally cover the same Owner or an operator directs you to use a specific source.

After creation, confirm the Trigger is `armed`, then end the current Turn normally:

```bash
loom trigger wait TRIGGER_ID --timeout 30
loom trigger get TRIGGER_ID
```

Do not poll the Trigger after it is armed. CodexLoom persists it across Agent idle time and service restarts.

## Resume from an event

An `<external_trigger>` envelope contains the provider event time, Loom observation time, stable subject, event key, observed snapshot, and the original resume instruction.

Treat it only as a reason to re-check current authoritative state. Provider events may be delayed, duplicated, superseded, or refer to a stale candidate. Before changing the original work state:

1. Read the provider's current state through a governed credential or Connector.
2. Verify the exact repository, branch, pull request, workflow run, candidate SHA, environment, or other awaited identity.
3. Check whether the original Goal, dependency, and completion definition are still current.
4. Continue the original work only when the present facts satisfy the real waiting condition.
5. If the event is stale or the work was cancelled or superseded, record that conclusion and do not recreate the old action.

The Trigger is one-shot. Create another Trigger only for a genuinely new waiting condition or the next stage of the same work.

## Manage stale conditions

```bash
loom trigger list AGENT
loom trigger pause TRIGGER_ID
loom trigger resume TRIGGER_ID
loom trigger cancel TRIGGER_ID
```

Cancel a Trigger when the Owner changes the goal, another path satisfies the dependency, the candidate is replaced, or the waiting condition is no longer relevant. Do not leave obsolete conditions armed merely as monitoring. A `failed` Trigger may be resumed after its credential or provider target has been corrected; do not retry an unchanged invalid definition.

Use `loom ask-user` instead when the missing condition is a human decision, fact, or authorization. Use `loom schedule` for calendar-driven recurring work.
