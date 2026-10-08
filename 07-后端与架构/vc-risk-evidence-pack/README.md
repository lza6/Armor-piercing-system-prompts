---
name: report:readme
description: "Readme"
date: 04-06-26
metadata:
  node_type: memory
  type: report
---
# Risk Evidence Harness

Manual-first durable proof pack for risky product changes.

Use this only when the selected plan or change touches a high-risk class such as:

- auth or identity flows
- billing, payments, or credit accounting
- database schema/data migrations or destructive data writes
- public API or external contract changes
- deploy/runtime/container/proxy/gateway behavior
- security-sensitive permission or secret handling

## Goal

Make risky work reviewable and resumable through small durable artifacts instead of relying only on narrative updates.

This harness is intentionally:

- manual-first
- opt-in by risk class
- non-blocking by default
- compatible with RIPER-5 and existing project plans

## Storage Rule

Use the reports folder that belongs to the selected plan:

- general work: `.claude/skills/vc-risk-evidence-pack/scripts/`
- feature-scoped work: inside the task folder: `process/features/{feature}/active/{slug}_{date}/harness/`

Create one subfolder per task or report set when needed.

## Artifact Set

### `risk-gate.json`

Minimal classification and stop decision.

Required fields:

- `task`
- `planPath`
- `riskLevel` (`low` | `medium` | `high`)
- `riskClasses` (array)
- `mustStopBeforeFinalize` (boolean)
- `humanApprovalRequired` (boolean)
- `why`

### `context-snippets.json`

Proof inputs collected before implementation or final review.

Recommended fields:

- `task`
- `artifacts`
- `logs`
- `queries`
- `diffSummary`
- `rootCause`

### `verification.json`

What was actually tested.

Required fields:

- `task`
- `commands`
- `manualChecks`
- `result`

### `review-decision.json`

Reviewer outcome for risky work.

Required fields:

- `task`
- `decision` (`approved` | `approved-with-concerns` | `rejected`)
- `blockingFindings`
- `nonBlockingFindings`
- `notes`

### `adversarial-validation.json`

Optional but recommended for high-risk paths.

Use for:

- auth bypass attempts
- boundary-value or malicious-input probes
- rollback/failure-mode checks
- data-leak or privilege-escalation scenarios

## Manual Validation Helper

Use:

```bash
node .claude/skills/vc-risk-evidence-pack/scripts/validate-risk-artifacts.mjs <artifact-dir>
```

This helper validates shape only. It does not enforce adoption or hook into CI by default.
