---
name: vc-risk-evidence-pack
description: "Define and generate the manual-first evidence pack for high-risk work. Covers 6 high-risk class definitions and the 5-artifact schema required before finalizing, pushing, or handing off."
argument-hint: "[risk class and work description]"
trigger_keywords: risk evidence, high-risk pack, evidence pack, risk gate, adversarial validation
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-risk-evidence-pack

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Generate and validate the manual-first evidence pack required before finalizing, pushing, or handing off high-risk implementation work.

## When To Invoke

- **VALIDATE Layer 1 security surface check** — when the plan touches a high-risk class, flag the need for an evidence pack before routing to execute-agent.
- **EXECUTE before marking high-risk work complete** — execute-agent must produce or verify the evidence pack exists before reporting DONE on any high-risk class.
- **code-reviewer as pre-PR quality gate** — code-reviewer checks for the evidence pack presence before approving changes that touch a high-risk surface.

## 6 High-Risk Class Definitions

From `process/development-protocols/orchestration.md` ("High-Risk Execution Handoff") and `process/development-protocols/implementation-standards.md` ("Risky Work Evidence Contract") — both sources agree:

1. **auth or identity** — authentication flows, session tokens, user identity resolution, Clerk JWT handling, or any surface that determines who the caller is.
2. **billing or credits** — billing events, credit balance mutations, Stripe charge flows, OpenRouter credit accounting, credit transaction records, or subscription state changes.
3. **schema/data migration or destructive data mutation** — Prisma migrations, raw SQL mutations, destructive writes that delete or overwrite persistent data, or schema changes to existing models.
4. **public API or external contract changes** — tRPC procedure signature changes visible to the frontend, Hono route contract changes consumed by external callers, webhook shape changes, or any API surface that third parties or the client app depend on.
5. **deploy/runtime/container/proxy/gateway behavior** — Dockerfile changes, supervisord config, start.sh, container service ports, Bun server entry, Hono route registration, Caddy proxy config, worker-node provisioning, or any change that affects how a running service starts or routes traffic.
6. **permission, secret, or trust-boundary logic** — instance token gating, `verifyInstanceOwnership`, BYOK secret fetch paths, MITM proxy key injection, Bright Data credential handling, or any logic that controls what a caller is allowed to access.

## 5-Artifact Schema

Per **task-folder artefact colocation**, all artifacts go inside the selected plan's task folder (e.g. `process/features/{feature}/active/{slug}_{date}/harness/` or `process/general-plans/active/{slug}_{date}/harness/`), so the whole pack moves with the plan as a unit. Legacy path `reports/harness/` is deprecated for new writes; never write the pack to a sibling `reports/` dir or any ad-hoc location. The validator script lives at `.claude/skills/vc-risk-evidence-pack/scripts/validate-risk-artifacts.mjs`.

### 1. `risk-gate.json`
Records the risk class, work description, and approver identity before work begins or is finalized.

```json
{
  "riskClass": "<one of the 6 classes above>",
  "workDescription": "<short description of the change>",
  "approver": "<person or agent that reviewed the risk classification>",
  "mustStopBeforeFinalize": true
}
```

### 2. `context-snippets.json`
Relevant code snippets with exact file and line citations for every surface the change touches in the high-risk class.

```json
{
  "snippets": [
    {
      "file": "packages/api/src/router/billing.ts",
      "lines": "120-145",
      "description": "verifyInstanceOwnership call before secret read",
      "content": "<excerpt>"
    }
  ]
}
```

### 3. `verification.json`
Documents every verification step taken and its result. Steps must cover both the happy path and at least one failure or boundary case.

```json
{
  "steps": [
    {
      "step": "<what was verified>",
      "command": "<command run, if applicable>",
      "result": "PASS | FAIL | SKIP",
      "notes": "<observations>"
    }
  ]
}
```

### 4. `review-decision.json`
The explicit reviewer decision record. Must contain APPROVE or REJECT with a written rationale — no implicit approvals.

```json
{
  "reviewer": "<name or agent>",
  "decision": "APPROVE | REJECT",
  "rationale": "<written reason>",
  "timestamp": "<ISO 8601 date>"
}
```

### 5. `adversarial-validation.json`
Required when the path is high-risk or attack-sensitive (e.g. auth bypass, privilege escalation, secret exfiltration). Documents adversarial scenarios considered and whether each was ruled out.

```json
{
  "scenarios": [
    {
      "scenario": "<attack or misuse description>",
      "ruled_out": true,
      "rationale": "<why this path is not exploitable>"
    }
  ]
}
```

## Auto-Stop Rule

If risk is `high`, do not treat the work as ready to finalize until the evidence pack exists and the reviewer decision is recorded.

If the evidence pack is missing, say so **explicitly** — do not proceed silently, do not imply the work is proven, and do not report DONE.

Verbatim from `implementation-standards.md`:

> Auto-stop rule:
>
> - if risk is `high`, do not treat the work as ready to finalize until the evidence pack exists and the reviewer decision is recorded
> - if the evidence pack is missing, say so explicitly instead of implying the work is proven
>
> This contract is manual-first and opt-in by risk class. It is not a default blocking hook.

Verbatim from `orchestration.md`:

> If the risk gate says `mustStopBeforeFinalize: true`, do not imply the work is fully proven until the pack exists and the reviewer decision is present.
> Keep this manual-first. Do not invent a blocking hook or alternate workflow owner.

## Validation Checklist

Steps to confirm each artifact is complete before handoff:

- [ ] `risk-gate.json` populated with correct risk class, work description, approver, and `mustStopBeforeFinalize` flag
- [ ] `context-snippets.json` includes all affected file:line citations for every surface touching the high-risk class
- [ ] `verification.json` documents each test step and result, covering happy path and at least one boundary/failure case
- [ ] `review-decision.json` has explicit APPROVE or REJECT with written rationale and timestamp — no implicit approvals
- [ ] `adversarial-validation.json` present if the path is attack-sensitive (auth bypass, privilege escalation, secret exfiltration, trust-boundary violation)

## High-Risk Work Evidence Contract

Verbatim from `process/development-protocols/implementation-standards.md` ("Risky Work Evidence Contract"):

> For high-risk work, use a manual-first evidence pack before calling the change ready for finalize, push, or human handoff.
>
> High-risk classes include:
>
> - auth or identity flows
> - billing, payments, or credit accounting
> - schema/data migrations or destructive writes
> - public API or external contract changes
> - deploy/runtime/container/proxy/gateway behavior
> - permission, secret, or trust-boundary logic
>
> Preferred artifact set inside the selected plan's task folder (`{slug}_{date}/harness/`):
>
> - `risk-gate.json`
> - `context-snippets.json`
> - `verification.json`
> - `review-decision.json`
> - `adversarial-validation.json` for high-risk or adversarial paths
>
> Auto-stop rule:
>
> - if risk is `high`, do not treat the work as ready to finalize until the evidence pack exists and the reviewer decision is recorded
> - if the evidence pack is missing, say so explicitly instead of implying the work is proven
>
> This contract is manual-first and opt-in by risk class. It is not a default blocking hook.
