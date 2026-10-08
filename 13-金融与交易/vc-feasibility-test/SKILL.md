---
name: vc-feasibility-test
description: "Use when a SPEC, INNOVATE, or VALIDATE (Layer 2) approach hinges on an unverified runtime/library/external mechanism: run a probe from the 8-family taxonomy and produce a VIABLE/NOT-VIABLE/INCONCLUSIVE VERDICT artifact."
argument-hint: "[hypothesis] [task-folder] [probe-family]"
trigger_keywords: feasibility, spike, verify assumption, probe mechanism, empirical check, unverified mechanism, does this work, runtime quirk, api shape check
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "2.0.0"
---

# vc-feasibility-test

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

One-shot empirical probe skill. Used when SPEC or INNOVATE encounters an approach
hinging on an unverified external/runtime/library mechanism — the question is
**"does this mechanism actually work the way the design assumes?"**, asked
**before** the decision locks.

## Boundary vs vc-test-coverage-plan

These two skills are complementary and must not be confused:

- **`vc-feasibility-test` (this skill) = PRE-decision.** "Does the mechanism work at
  all?" Run *before* SPEC/INNOVATE locks an approach, when the answer is unknown.
  Output: a one-shot VERDICT artifact.
- **`vc-test-coverage-plan` = POST-plan.** "How do I cover this blast radius across
  the 4 test tiers?" Run *after* a plan exists, when the design is already chosen.
  Output: a per-area tier table.

If the approach is already decided and you are assigning test tiers → use
`vc-test-coverage-plan`. If you cannot decide *because* a mechanism is unverified →
use this skill first.

## When To Invoke

- When vc-spec-agent, vc-innovate-agent, or a vc-validate-agent Layer 2 dimension agent emits `VC-FEASIBILITY-PROBE-NEEDED`
- When a mechanism cannot be verified from source code alone
- One-shot: not an iterative loop (use vc-autoresearch for iteration)

## Skill Executor

Always executed by vc-debugger (via the `VC-FEASIBILITY-PROBE-NEEDED` signal routing
in orchestration.md). SPEC, INNOVATE, and VALIDATE Layer 2 agents do not run probes themselves.

## Probe-Method Taxonomy (pick one family)

Every probe belongs to one of these 8 families. Name the chosen family in the VERDICT.
Each family has a default cost/safety class (see next section) — the probe inherits it.

| # | Family | What it probes | Typical method | Default cost class |
|---|---|---|---|---|
| 1 | **Local process / Node script** | pure library/runtime behavior in isolation | run a `.mjs`/Bun script, regex/parse check, call the lib fn directly | cheap-local |
| 2 | **Unit/integration test harness** | behavior under the project's own test runner | `pnpm --filter … test` (Vitest) or `bun test` on a scratch case | cheap-local |
| 3 | **tRPC / Prisma / DB query** | route shape, query behavior, index/constraint semantics | hit a tRPC route or run a Prisma/raw-SQL query against a test DB | needs-container *(only if it needs the live app DB)* / else cheap-local |
| 4 | **External API shape capture** | real response shape/behavior of a 3rd-party API | one live request to OpenRouter / Stripe / Composio / Clerk / Bright Data | **needs-live-provider** |
| 5 | **Container exec / internal-port curl** | in-container service behavior, proxy injection, file-server, supervisord | `docker exec … curl http://localhost:{port}` on a **disposable** container | **needs-container** |
| 6 | **Browser / CDP capture** | anti-detect quirks, CDP events, SPA nav, popup behavior | Playwright/CDP client, `page.on(...)`, snapshot | needs-browser |
| 7 | **WS / SSE handshake & timing** | gateway WS framing, SSE delivery/reconnect, JSONL shape | raw `ws`/`EventSource` client + frame/timing capture | needs-container *(if against in-container service)* / else cheap-local |
| 8 | **Cloudflare worker runtime** | KV staleness, step-replay/idempotency, edge JWT verify | `wrangler dev` + curl, deploy a throwaway worker | needs-cf |

If none of the 8 fit, the question is probably not a feasibility probe — reconsider
whether `vc-research-agent` (unknown *context*) or `vc-test-coverage-plan` (known
design) is the right tool instead.

## Probe Cost / Safety Class (MANDATORY GATE)

Every VERDICT declares one cost class. The class governs whether the probe may run
unattended or needs explicit opt-in. **A probe that cannot be run within its safety
gate produces an `INCONCLUSIVE` verdict — it is never silently skipped or faked.**

| Cost class | Safety gate | If gate not met |
|---|---|---|
| **cheap-local** | none — run freely (local script, test harness, parse check) | n/a |
| **needs-container** | use a **disposable** container only. NEVER `docker exec` the shared dev container (`app-*`) or shared Postgres. Disposable live-E2E containers need the disposable-cleanup env gate enabled. | verdict `INCONCLUSIVE`, note "no disposable container available" |
| **needs-live-provider** | requires explicit **double opt-in** from the user before any billed/live 3rd-party call (OpenRouter, Stripe, Composio, Bright Data, Clerk). Default local mode is BYOK Mistral. | verdict `INCONCLUSIVE`, note "live-provider opt-in not granted" |
| **needs-browser** | a browser/CDP session must be available; never drive a shared user session | verdict `INCONCLUSIVE`, note "no browser session available" |
| **needs-cf** | a `wrangler dev`/throwaway-worker sandbox; never touch a deployed production worker | verdict `INCONCLUSIVE`, note "no CF sandbox available" |

The emitted `VC-FEASIBILITY-PROBE-NEEDED` signal SHOULD carry the anticipated cost
class so the orchestrator can resolve the opt-in gate before dispatching vc-debugger
(see orchestration.md §VC-FEASIBILITY-PROBE-NEEDED Signal Routing).

## Probe Execution Steps

1. Read the hypothesis from the `VC-FEASIBILITY-PROBE-NEEDED` signal
2. Pick the probe **family** (1–8) and its **cost class**
3. Confirm the safety gate for that cost class is met. If not → write an
   `INCONCLUSIVE` verdict with the gate-not-met reason and stop. Do NOT escalate
   to a higher-cost probe or run against a shared resource.
4. Design the minimal probe within that family
5. Run the probe empirically — capture actual output, not expected output
6. Analyze the evidence: does it confirm or refute the hypothesis?
7. Assign a verdict: `VIABLE` | `NOT-VIABLE` | `INCONCLUSIVE`
8. Write the VERDICT artifact to the active task folder

## VERDICT Artifact Format

Filename: `{slug}_FEASIBILITY_{dd-mm-yy}.md`
Location: same active task folder as the SPEC/plan that triggered the probe

**Required frontmatter fields (MUST be present — validated by `validate-feasibility-verdict.mjs`):**

```yaml
---
slug: [task-slug]
date: YYYY-MM-DD
verdict: VIABLE | NOT-VIABLE | INCONCLUSIVE
originating-phase: spec | innovate | pvl
---
```

The `originating-phase:` field is REQUIRED. Valid values:
- `spec` — probe triggered by vc-spec-agent ([SP3])
- `innovate` — probe triggered by vc-innovate-agent ([I2.5])
- `pvl` — probe triggered by vc-validate-agent Layer 2 ([V2-PROBE])

Required sections (MUST be present — validated by `validate-feasibility-verdict.mjs`):

### Hypothesis
One-sentence statement of what is being tested.

### Mechanism Under Test
The specific external, runtime, or library behavior being probed.

### Probe Family
One of the 8 families above (e.g. `5 — Container exec / internal-port curl`).

### Probe Cost Class
One of: `cheap-local` | `needs-container` | `needs-live-provider` | `needs-browser` | `needs-cf`.
State whether the safety gate was met.

### Probe Method
The exact command(s) or steps run to test the hypothesis.

### Evidence Captured
The raw output from the probe (trimmed to relevant lines). For an `INCONCLUSIVE`
gate-not-met verdict, state explicitly that the probe was not run and why.

### Verdict
One of: `VIABLE` | `NOT-VIABLE` | `INCONCLUSIVE`

### Resulting Design Constraint
The "action consequence" of the probe, split into three explicit parts:

- **What this licenses:** what the approach is now allowed to depend on.
- **What this forbids:** what the approach must NOT depend on.
- **What remains uncertain (known-gap):** what the probe did not settle and must be
  treated as an open risk (for `INCONCLUSIVE`, this is the main content).

## Completion Signal

After writing the VERDICT artifact, emit:
```
VC-FEASIBILITY-VERDICT-READY: [verdict keyword] — [full path to VERDICT file]
```

Example:
```
VC-FEASIBILITY-VERDICT-READY: NOT-VIABLE — process/features/model-selector/active/model-selector_10-06-26/model-selector_FEASIBILITY_10-06-26.md
```

## Re-spawn Handoff

The orchestrator reads the VERDICT artifact and extracts a `Prior Feasibility:` summary.
Format passed to the re-spawned agent:

```
Prior Feasibility: [hypothesis] — verdict: [VIABLE|NOT-VIABLE|INCONCLUSIVE] — licenses: [one line] — forbids: [one line] — uncertain: [one line]
```

Example:
```
Prior Feasibility: Does the gateway forward params.provider.sort? — verdict: NOT-VIABLE — licenses: nothing new — forbids: any approach depending on params.provider.sort being forwarded (the layer strips it) — uncertain: whether a different forwarding field survives
```

The re-spawned SPEC, INNOVATE, or VALIDATE agent reads this block and uses the verdict to lock or reject the approach. When `originating-phase: pvl`, the re-spawned vc-validate-agent resumes from V1 and records resolved probes in a `## Feasibility Probes Resolved` subsection of the validate-contract (omitted when no probe ran).
