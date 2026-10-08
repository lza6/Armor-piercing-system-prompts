---
name: vc-test-coverage-plan
description: "Use when creating a test plan for a blast radius. Assigns all 4 tiers (fully-automated, hybrid, agent-probe, known-gap) with exact commands, what each proves, and gap resolution options."
argument-hint: "[blast radius description or plan file path]"
trigger_keywords: test coverage plan, test tiers, blast radius coverage, gap resolution, TDD plan
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-test-coverage-plan

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Generate a TDD-first full test plan per blast radius area. Assigns all 4 test tiers with exact commands, what each proves, what it does NOT prove, and explicit resolution options for every gap.

## Boundary vs vc-feasibility-test

This skill is **POST-decision**: the design is already chosen and you are assigning
coverage tiers across a known blast radius. If instead an approach cannot be decided
*because* a runtime/library/external mechanism is unverified — that is a PRE-decision
question and belongs to `vc-feasibility-test` (a one-shot empirical probe producing a
VIABLE/NOT-VIABLE/INCONCLUSIVE VERDICT), run before SPEC/INNOVATE locks. Do not use
test tiers to answer "does this mechanism work at all?".

## When To Invoke

- **PLAN phase** — populate the Verification Evidence section of a new plan
- **VALIDATE Section III** — generate the full test plan after V2 fan-out, before writing the validate-contract
- **EXECUTE phase** — as test gates at the end of each plan section and as a regression suite after all sections complete

## Context Discovery (MANDATORY FIRST — do this before anything else)

This skill MUST NOT infer tiers, commands, or runners from training data. Before reading the
plan or naming a single area:

1. Invoke `vc-context-discovery` to load the relevant context group files.
2. Read `process/context/tests/all-tests.md` **and follow its downstream routing chain** to the
   relevant deeper test docs (`tests/container-e2e.md`, `tests/browser-automation.md`,
   `tests/live-e2e.md`, etc.). The entry point is a router, not full knowledge — reading only
   the router and skipping the chain is insufficient.
3. Discover the existing test files inside the blast radius (real runners + real commands +
   real fixtures — not guesses).

**Hard stop (mirrors `vc-plan-agent` TIER_ASSIGNMENTS_BLOCKED):** if the `all-tests.md` routing
chain was not loaded, or existing blast-radius test files were not discovered, STOP and emit
`TIER_ASSIGNMENTS_BLOCKED` — report BLOCKED with "Test context chain not loaded; returning to
RESEARCH to load all-tests.md and discover existing test files. Do not generate tier assignments
from training data." Do NOT proceed to the waterfall. Every `Command / Steps` cell below must be
an exact command sourced from the loaded test context, never an inferred placeholder.

## Test Tier Decision Waterfall

For each area in the plan's blast radius, assign a tier using this waterfall:

1. **Fully-automated** — if a deterministic command exists that exercises the area end-to-end
   without human judgment. Must be runnable in CI without setup beyond env vars.
   Examples: `pnpm test`, `bun test`, `node validate-script.mjs`, grep checks.

2. **Hybrid** — if the test requires a precondition (running container, live DB, specific
   env) that is not always available in CI, but the test itself is deterministic once set up.
   Record the precondition explicitly. Examples: container E2E tests, DB migration checks.

3. **Agent probe** — if the area requires judgment that cannot be mechanically asserted.
   Describe the probe scenario and what the agent should judge. Examples: UI visual
   regression, prose quality, API response plausibility.

4. **Known gap** — if no test exists and none can be added within the blast radius of this
   plan. Document the gap explicitly. Do not use this tier to avoid writing tests.

## High-Risk Classes

These classes always require at least a hybrid test gate (no known-gap allowed without
explicit documented rationale):

- auth or identity flows
- billing, payments, or credit accounting
- schema/data migrations or destructive writes
- public API or external contract changes
- deploy/runtime/container/proxy/gateway behavior
- permission, secret, or trust-boundary logic

Required table format for high-risk class areas:

| Area | High-risk class | Minimum tier | Gap rationale if known-gap accepted |
|---|---|---|---|
| [e.g. Auth/identity flow] | auth/identity | Hybrid | [If known-gap: must state why hybrid is impossible and what alternative coverage exists] |
| [e.g. Billing credit deduction] | billing/credits | Hybrid | — |

## Hybrid Failure Resolution Priority

When a hybrid test fails during or after EXECUTE:

1. **Fix now** — if the failure is in the blast radius of the current plan and the fix is
   small. Fix, re-run the hybrid gate, confirm green, then continue.
2. **New phase plan** — if the failure requires work that is outside the current phase scope
   but has a clear fix. Create a follow-up phase plan and document the gap.
3. **Update existing phases** — if the failure is in a phase that is still active and the
   fix can be absorbed without scope expansion. Route the fix back to that phase.
4. **Backlog note** — if the failure is in a known-gap area, the fix is non-trivial, and
   deferral is acceptable. Write a backlog artifact. Do not silently absorb it.

## Per-Area Test Plan Output Format

Produce one block per area in the blast radius. Area = package, service, or logical surface (e.g. `packages/api` — new route, `packages/ui` — UI component).

**Area: [package/service name]**

| Tier | Scenario | Command / Steps | What it proves | What it does NOT prove |
|---|---|---|---|---|
| Fully-automated | [e.g. Route returns 200 with correct shape] | `[exact command]` exits 0 | [Specific outcome proved] | [Explicit gap] |
| Fully-automated | [e.g. Route returns 401 on missing token] | Same suite, auth-rejection case | [Specific outcome proved] | [Explicit gap] |
| Hybrid | [e.g. Integration with real DB] | `[exact command]` — precondition: [what must be running/set] | [Specific outcome proved] | [Explicit gap] |
| Agent probe | [e.g. Visual or behavioral judgment] | [Step-by-step scenario for the agent] | [What the agent judges] | [What cannot be automated] |
| Known-gap | [e.g. Load behavior under concurrent requests] | — | — | Cannot be tested within this plan's scope |

Rules:
- Include a row for every tier that applies. Omit a tier row only if the tier genuinely does not apply — do not omit to avoid work.
- Known-gap rows must have `—` in the Command/Steps column and a brief reason in the "What it does NOT prove" column.
- Fully-automated commands must be exact and runnable — do not use placeholder `[command]` in a real output.

## Gap Resolution Options Format

After the per-area table, list every gap with four resolution choices:

| Gap | Resolution options |
|---|---|
| [Gap 1 description] | A) [Write new test — estimated effort]. B) [Set up infra — what and how]. C) [Accept as known-gap — rationale]. D) [Backlog artifact — what to create]. |
| [Gap 2 description] | A) [Option]. B) [Option]. C) [Option]. D) [Option]. |

Resolution option rules:
- **A — Write new test**: state file location and estimated effort (e.g. "30 min, new file `packages/api/src/__tests__/route-shape.test.ts`").
- **B — Set up infra**: name what infra is needed and how (e.g. "seed DB fixture via `pnpm db:seed:test`").
- **C — Accept as known-gap**: rationale is required — never blank. High-risk class gaps need especially strong rationale.
- **D — Backlog artifact**: state what to create and where (e.g. "`prod-migration-smoke-test_NOTE_[date].md` in `process/features/development-process/backlog/`").

## Missing Test Areas Format

Areas with no coverage possible at any tier within this plan's scope:

| Area | Why untestable in this plan | Resolution chosen |
|---|---|---|
| [e.g. Production migration path] | Requires prod-like Postgres; outside phase scope | Backlog: [artifact name] |
| [e.g. Token expiry mid-session] | Requires Clerk test tenant with configurable JWT TTL | Backlog: [artifact name] |
| [e.g. Cross-instance isolation] | Requires 2+ live running instances | Deferred to [program/phase name] |

## Execution Protocol

0. **Run Context Discovery (MANDATORY FIRST) above** — load the `all-tests.md` routing chain and discover existing blast-radius test files. If not loaded, emit `TIER_ASSIGNMENTS_BLOCKED` and STOP; do not continue.
1. Read the plan file at the provided path (or parse the blast radius description if no path given).
2. Extract the blast radius areas from the plan + the loaded test context — never infer commands/runners from training data.
3. For each area, run the Test Tier Decision Waterfall.
4. Flag any area that matches a High-Risk Class — enforce hybrid minimum.
5. Produce the per-area test plan block (5-column table + gap resolution table).
6. Produce the missing test areas table.
7. If invoked during VALIDATE (Section III), embed output directly into the validate menu under "III. Test Coverage Plan" — do not write a separate file.
8. If invoked during PLAN or EXECUTE, output directly in chat for the caller to copy into the plan file.

## TDD Stub Output Requirement

For every **Fully-automated tier row** in the per-area output table, append immediately after that
row's 5-column entry an inline failing test skeleton in plain text:

```
Failing stub:
test("should [behavior from Scenario column]", () => {
  throw new Error("NOT IMPLEMENTED — TDD stub for: [behavior]")
})
```

Rules for the stub:

- The stub is destined for the validate-contract's Test Gates section, to be consumed by
  execute-agent as the red-first starting point (Mode A hard gate). It is **NOT** an on-disk
  `.test.ts` file — do not write it to disk during VALIDATE or PLAN phase.
- The stub content must match the Scenario cell verbatim so execute-agent can find it by scenario name.
- **Hybrid / Agent-Probe / Known-Gap tiers do NOT receive stubs.** A literal red E2E or
  container test is too costly to mandate; hybrid stubs are advisory only.

Clarification note: `vc-test-coverage-plan` retains its exhaustive behavior-inventory framing —
"each row is a behavior to COVER, not a test to write upfront." The stubs make the coverage
intent machine-executable at EXECUTE time, not upfront test implementation. The four tier words
(Fully-automated / Hybrid / Agent-Probe / Known-Gap) remain verbatim; this requirement is additive
to the per-area output format.

## Absorption Note

This skill absorbs `vc-test-tier-selector` if that skill existed. If `vc-test-tier-selector` still exists on disk as a separate folder under `.claude/skills/`, treat this skill as its canonical replacement and note the duplication in the phase report. Do not route new work to `vc-test-tier-selector`.
