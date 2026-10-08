---
name: reference:example-validate-output
description: "Example Validate Output"
date: 04-06-26
metadata:
  node_type: memory
  type: reference
---
# Example Validate Output — Full Template Reference

This file is a reference example for the full V4 Validate Menu output that
`vc-validate-agent` must produce. It shows every section at maximum detail so
implementing agents and skill authors know exactly what is expected.

**How to use this file:**
- Use it to calibrate the validate menu format when updating `vc-validate-findings` skill
- Use it as the target shape when building `vc-generate-validate` or `vc-agent-strategy-compare` skills
- Placeholder values are in `[brackets]` — replace with real content for each plan

**Key design principle:**
The validate step exists to UPDATE the plan, not just audit it. Fixable concerns are
applied to the plan inline. The validate-contract is the full execute-agent handoff
document — not a compact summary.

---

## VALIDATE — V4 Menu

> **What V1-V3 did:**
> - **V1 (pre-check):** Confirmed plan file readable. Blast radius has [N] files across [N] packages. Computed [N]/7 strategy signals. [Note any inferences made, e.g. "Blast Radius inferred from Implementation Checklist — no dedicated section present."]
> - **V2 (fan-out):** Ran 4 Layer 1 dimension agents (infra fit, test coverage, breaking changes, security) + [N] Layer 2 per-section feasibility agents, all in parallel.
> - **V3 (synthesis):** Collected all [N] agent outputs. Counted: [N] FAILs, [N] CONCERNs, [N] PASSes. Applied test tier waterfall. Proposed plan fixes for [N] resolvable concerns. Net gate derived below.

---

## I. Validation Findings → Net Gate

> All concerns are shown with their proposed fix. When you accept at V5, the fixable ones
> are applied to the plan immediately. Only fundamental design problems require returning to PLAN.

### Layer 1 — Dimension Findings

**Infra / Setup Fit**

| Finding | Severity | Proposed fix |
|---|---|---|
| [File path listed in plan does not exist at that location — e.g. `packages/api/src/infra/proxy.ts` vs actual `packages/api/src/routes/proxy.ts`] | CONCERN | Apply to plan: correct path in Section [N] edit target. OR: execute-agent instruction: "confirm exact path at entry; update edit target; do not skip." |
| [Port number matches all-context.md] | ✅ PASS | — |
| [Service name consistent with supervisord.conf] | ✅ PASS | — |
| [Any other infra finding] | PASS / CONCERN / FAIL | [Proposed fix or none] |

**Test Coverage**

| Finding | Severity | Proposed fix |
|---|---|---|
| [No integration test exists for the new API response envelope shape] | CONCERN | A) Add to plan: new test file in Section [N] that asserts full envelope shape. B) Accept as known-gap with rationale. C) Create backlog artifact. |
| [Existing Vitest suite covers the UI component path] | ✅ PASS | — |
| [Hybrid gate precondition is realistic given test infra] | ✅ PASS | — |

**Breaking Changes**

| Finding | Severity | Proposed fix |
|---|---|---|
| [New route changes the response envelope; N downstream consumers not listed in blast radius] | CONCERN | Apply to plan: add consumers to blast radius section and implementation checklist. |
| [DB model addition is additive only — no existing table modified] | ✅ PASS | — |
| [No public API contract changes] | ✅ PASS | — |

**Security Surface**

| Finding | Severity | Proposed fix |
|---|---|---|
| [New route uses Clerk auth — no bypass found] | ✅ PASS | — |
| [No PII stored; no secrets written to disk] | ✅ PASS | — |
| [STRIDE scan clean] | ✅ PASS | — |
| [Any OWASP/STRIDE concern found] | CONCERN / FAIL | [Proposed fix — auth guard, input validation, etc.] |

---

### Layer 2 — Per-Section Feasibility

Repeat this block for every section or phase in the plan.

**Section [A] — [Section Name]**

| Question | Verdict | Detail |
|---|---|---|
| Mechanical feasibility | PASS / CONCERN / FAIL | [Are edit target strings present and uniquely matchable? Can create/write steps execute without collision?] |
| Plan gaps | PASS / CONCERN | [What is missing that should be here? Adjacent files or behaviors not listed?] |
| Conflicts | PASS / CONCERN / FAIL | [Anything contradicting current file state, other sections, or repo conventions?] |
| Highest-risk edit | [Description] | [Single highest-risk edit in this section and how execute-agent should sequence or mitigate it] |

Proposed fixes for this section:
- [Fix 1: apply to plan — e.g. add missing step to implementation checklist]
- [Fix 2: execute-agent instruction — e.g. "verify barrel export exists before writing"]

---

### Net Gate Derivation

| Layer 1 dimensions | Status |
|---|---|
| Infra fit | PASS / CONCERN / FAIL |
| Test coverage | PASS / CONCERN / FAIL |
| Breaking changes | PASS / CONCERN / FAIL |
| Security surface | PASS / CONCERN / FAIL |

| Layer 2 sections | Status |
|---|---|
| Section A — [name] | PASS / CONCERN / FAIL |
| Section B — [name] | PASS / CONCERN / FAIL |
| Section N — [name] | PASS / CONCERN / FAIL |

**Totals: [N] FAILs / [N] CONCERNs / [N] PASSes**

**→ Net Gate: [PASS / CONDITIONAL / BLOCKED]**

- **PASS:** 0 FAILs, 0 CONCERNs. All plan fixes applied. Proceed to EXECUTE.
- **CONDITIONAL:** 0 FAILs, [N] CONCERNs. [N] fixed in plan, [N] as execute-agent instructions, [N] as known-gaps. Proceed to EXECUTE with gaps on record.
- **BLOCKED:** [N] unresolved FAILs. [List each.] Return to PLAN — do not route to EXECUTE until each FAIL is resolved or explicitly converted to CONDITIONAL by user.

---

## II. Execution Strategy

### Signal Score

[N] signals present out of 7:

| Signal | Present |
|---|---|
| S1: Multi-package scope (3+ workspace packages) | ✅ / — |
| S2: Schema/API/auth surface touched | ✅ / — |
| S3: 3+ viable directions surfaced in INNOVATE | ✅ / — |
| S4: Phase program classification (3+ phases) | ✅ / — |
| S5: User requested depth explicitly | ✅ / — |
| S6: High-risk class in blast radius (auth, billing, schema, public API, container/gateway, secrets) | ✅ / — |
| S7: 5+ files in blast radius | ✅ / — |

Score: **[N] → threshold: 0-1 = sequential, 2-3 = parallel-subagents, 4+ = vc-team**
Dominant signal: [name the signal that most drives the recommendation]

### Strategy Options

| Strategy | Agent count calculation | Total | Cost guard | Fit for this plan |
|---|---|---|---|---|
| **Sequential** | 1 executor, all sections in order | **1 agent** | None | [Fit assessment] |
| **Parallel subagents** | 4 (Layer 1) + [N sections] + [N optional: reviewer, tester, security] | **[N] agents** | [Below/>30/>100] | [Fit assessment] |
| **Workflow (dynamic pipeline)** | [N pipeline steps] × [N agents/step] × [N iterations] | **[N] agents** | [Below/>30/>100] | [Fit assessment — right for deterministic pipelines, TDD fan-out loops, metric iteration] |
| **Agent team (vc-team)** | [N members] × [N rounds] | **[N] agent invocations** (+1.5× inter-agent overhead) | [N members vs 6-member threshold] | [Fit assessment — right when 2+ workstreams must communicate mid-execution] |

Cost guard rules:
- >30 agents → show breakdown so user can judge
- >100 agents → show breakdown AND ask for explicit confirmation before proceeding
- >6 team members → show member roles AND ask for explicit confirmation

Note: V5 "Accept" satisfies both the cost-guard confirmation and the plan-approval in one gate.

### Recommendation

**[Recommended strategy] — [N] agents**

[1-2 sentence rationale: why this strategy fits the specific plan. What makes the other strategies wrong or overkill for this case.]

Strategy-by-fit rules (not a fixed ranking):
- **Sequential:** right for trivial or single-file changes regardless of signal score
- **Parallel subagents:** right for independent per-section review with no cross-section communication needed during EXECUTE
- **Workflow:** right for deterministic pipeline validation, TDD fan-out loops, repeated metric iteration
- **Agent team:** right when 2+ workstreams must communicate mid-execution (e.g. one section's output is required input for another)

---

## III. Test Coverage Plan

Full test plan per blast radius area. Shows all tiers, what each covers, what it does NOT cover,
and explicit resolution options for every gap. Execute-agent runs these gates and records outcomes.

---

**Area: [package/service name — e.g. `packages/api` — new API route]**

| Tier | Scenario | Command / Steps | What it proves | What it does NOT prove |
|---|---|---|---|---|
| Fully-automated | [e.g. Route returns 200 with correct shape] | `[exact command]` exits 0 | [Specific outcome proved] | [Explicit gap] |
| Fully-automated | [e.g. Route returns 401 on missing token] | Same suite, auth-rejection case | [Specific outcome proved] | [Explicit gap] |
| Hybrid | [e.g. Integration with real DB] | `[exact command]` — precondition: [what must be running/set] | [Specific outcome proved] | [Explicit gap] |
| Agent probe | [e.g. Visual or behavioral judgment] | [Step-by-step scenario for the agent] | [What the agent judges] | [What cannot be automated] |
| Known-gap | [e.g. Load behavior under concurrent requests] | — | — | Cannot be tested within this plan's scope |

Gaps and resolution options:

| Gap | Resolution options |
|---|---|
| [Gap 1 description] | A) [Write new test — estimated effort]. B) [Set up infra — what and how]. C) [Accept as known-gap — rationale]. D) [Backlog artifact — what to create]. |
| [Gap 2 description] | A) [Option]. B) [Option]. |

---

**Area: [package/service name — repeat block for every area in blast radius]**

[Same structure as above]

---

**High-risk class areas** (auth, billing, schema migration, public API, container/gateway, secrets)

These areas require hybrid tier minimum. Known-gap is not allowed without an explicitly documented rationale.

| Area | High-risk class | Minimum tier | Gap rationale if known-gap accepted |
|---|---|---|---|
| [e.g. Auth/identity flow] | auth/identity | Hybrid | [If known-gap: must state why hybrid is impossible and what alternative coverage exists] |
| [e.g. Billing credit deduction] | billing/credits | Hybrid | — |

---

**Missing test areas (no coverage possible at any tier within this plan's scope)**

| Area | Why untestable in this plan | Resolution chosen |
|---|---|---|
| [e.g. Production migration path] | Requires prod-like Postgres; outside phase scope | Backlog: [artifact name] |
| [e.g. Token expiry mid-session] | Requires Clerk test tenant with configurable JWT TTL | Backlog: [artifact name] |
| [e.g. Cross-instance isolation] | Requires 2+ live running instances | Deferred to [program/phase name] |

---

## IV. Proposed Plan Updates (for review before V5)

These are the changes validate will apply to the plan file when you accept.
Review them here. If any are wrong, say so before accepting.

| # | What changes | Where in plan | Why |
|---|---|---|---|
| P1 | [e.g. Add route registration step to Section A checklist] | [Section A — Implementation Checklist] | Gap found: route not reachable without this step |
| P2 | [e.g. Correct blast radius: add SkillCard.tsx and useSkillsList.ts] | [Blast Radius section] | Breaking-changes agent found 2 unlisted downstream consumers |
| P3 | [e.g. Clarify db:push = dev/test context; db:deploy = prod migration] | [Section B — Implementation Checklist] | Ambiguity would cause execute-agent to use wrong command in wrong context |
| P4 | [e.g. Add auth probe as explicit test scenario with step-by-step] | [Verification Evidence section] | Test coverage agent found auth probe undocumented |

Execute-agent instructions (concerns that cannot be fixed in plan text):

| # | Instruction | Trigger condition |
|---|---|---|
| E1 | [e.g. Confirm proxy.ts exact path before writing Section A. If path differs: update edit target, do NOT skip. Document corrected path in phase report.] | Section A entry |
| E2 | [e.g. ctx-gateway change requires image rebuild. Use docker:build + container recreate via API lifecycle. Never docker cp.] | Section D entry |

Backlog artifacts to create during durable capture phase:

| Artifact | Location | What it tracks |
|---|---|---|
| [e.g. test-envelope-regression_NOTE_03-06-26.md] | [process/features/development-process/backlog/] | [Envelope regression test against downstream consumers] |
| [e.g. prod-migration-smoke-test_NOTE_03-06-26.md] | [Same] | [CI-runnable db:deploy smoke test against throwaway Postgres] |

---

## V. User Decision (V5 Gate)

> You are approving the proposed plan updates (Section IV) and the test coverage plan (Section III).
> When you accept, validate applies the plan fixes, writes the validate-contract, then hands off to EXECUTE.

**Your options:**

**A — Accept (apply fixes + write contract + proceed to EXECUTE)**
All proposed plan updates (P1–P[N]) applied to the plan file. Execute-agent instructions (E1–E[N]) written to the validate-contract. Test plan written. Backlog artifacts listed for creation during durable capture. Gate: [PASS / CONDITIONAL].

**B — Modify before accepting**
One or more proposed updates need changing before applying. State which and how. Validate revises and re-presents before writing.

**C — Override execution strategy**
Accept the findings and fixes, but use a different strategy than recommended. Name it:
`Sequential` / `Workflow` / `Agent team` / specify team member count and roles.
Validate writes the contract with your chosen strategy.

**D — Re-run specific agents**
One or more dimension or section agents returned suspect findings. Name which to re-run. Menu re-presented after re-run with updated findings.

**E — Escalate to PLAN (BLOCKED)**
A fundamental design problem exists that requires rethinking the approach — not just fixing plan text. [FAILs only, or user-identified architecture issue.] Route back to vc-plan-agent. VALIDATE re-runs from V1 after plan is revised.

---

## Result: Validate Contract written to plan file at V6

After V5 acceptance, this section is appended to the plan file.
It is the complete execute-agent handoff document.

```markdown
## Validate Contract

Status: PASS | CONDITIONAL | BLOCKED
Date: [dd-mm-yy]
Gate: [PASS — no FAILs, all fixes applied] | [CONDITIONAL — [N] concerns resolved: [N] plan fixes, [N] execute-agent instructions, [N] known-gaps accepted] | [BLOCKED — unresolved FAILs: list]

### Parallel strategy
Choice: sequential | parallel-subagents | workflow | vc-team
Signals: [N]/7 — dominant: [signal name]
Agent count: [N] ([breakdown])

### Plan updates applied
- [x] [Description of each fix applied to plan text]
- [x] [...]

### Execute-agent instructions
- [Section/condition]: [Exact instruction for what to do]
- [Section/condition]: [Exact instruction]

### Test gates (run after each section; regression suite after all sections)

**[Area name]**
- [tier]: `[command]` exits 0
  Proves: [what]
  Precondition: [if hybrid]
- [tier]: [scenario steps if agent probe]
- Known-gap: [description] — resolution: [backlog artifact / deferred to / accepted with rationale]

**[Area name]**
- [repeat per area]

**Regression suite (after all sections complete)**
- `pnpm test:local` exits 0
- `pnpm typecheck` exits 0
- `pnpm lint:verified` exits 0

### High-risk pack
Required: yes | no
[If yes: list required artifacts — risk-gate.json, verification.json, etc.]
[If yes: describe what evidence execute-agent must record before phase closeout]

### Backlog artifacts to create during durable capture
- [path/artifact-name.md] — [what it tracks]

### Known gaps on record
- [Gap description] — [resolution rationale and who accepted it]

### Accepted by
[user | session] — [list each accepted concern or known-gap by name]
```

---

## Gate Emit (V7)

After the validate-contract is written to the plan file, emit this block to the orchestrator:

```
Gate: PASS | CONDITIONAL | BLOCKED
Plan path: [path to plan file]
Validate-contract written: yes
Plan updates applied: [N] ([list fix titles])
Execute-agent instructions: [N]
Test gates written: yes
High-risk pack required: yes | no
Backlog artifacts to create: [N]
Next step: ENTER EXECUTE MODE | Return to PLAN (if BLOCKED)
```

---

## Notes for Implementing Agents

**What validate is NOT:**
- A summary or audit report that stays in chat
- A passive verdict on a list of problems
- A gate that just says "accept concerns" and moves on

**What validate IS:**
- A plan improvement pass — fixable concerns become plan text changes
- A test plan generator — every blast radius area gets a full tier assignment with gaps documented
- A handoff document author — the validate-contract tells execute-agent everything it needs without re-reading the conversation

**The validate-contract is complete when execute-agent can answer these questions from it alone:**
1. Which execution strategy and how many agents?
2. Which sections can run in parallel vs. must be sequential?
3. What test command do I run after each section? What tier?
4. What are the preconditions for hybrid tests?
5. What do I do if I encounter concern [G1], [G2], etc.?
6. What evidence do I need to collect for the high-risk pack?
7. What backlog artifacts do I create during durable capture?
8. What does "phase complete" mean — what evidence proves it?
