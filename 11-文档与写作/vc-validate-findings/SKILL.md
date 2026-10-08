---
name: vc-validate-findings
description: "Use when running VALIDATE V2-V3 fan-out. Two-layer investigation (4 dimension agents + per-section feasibility agents) synthesized into PASS/CONDITIONAL/BLOCKED net gate. Strategy-agnostic."
argument-hint: "[plan file path]"
trigger_keywords: validate findings, layer 1 dimensions, layer 2 feasibility, net gate, validate fan-out
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-validate-findings

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Defines the role specs, prompts, and output schemas for the two-layer VALIDATE fan-out (a parallel check across 4 dimensions + per-section feasibility probes). Produces a net gate verdict (PASS / CONDITIONAL / BLOCKED) and all the inputs needed to write the validate-contract — the written checklist that gates EXECUTE.

---

## References

- `references/example-validate-output.md` — full V4 menu + validate-contract template; calibrate output format against this file

---

## When To Invoke

- VALIDATE V2–V3: invoked by `vc-validate-agent` after V1 pre-check passes
- Post-execute review in UPDATE PROCESS when re-validation of a plan is needed

---

## Strategy Boundary

This skill is **STRATEGY-AGNOSTIC**. It defines the role specs, prompts, and output schemas for each dimension/section agent. It does NOT determine how those agents are executed (Workflow vs parallel Agent tool vs vc-team).

The execution method is determined by `vc-agent-strategy-compare`, which must be invoked BEFORE `vc-validate-findings` is called. `vc-validate-findings` outputs agent role definitions; the caller executes them using the recommended strategy.

---

## Mode Selection

Choose before spawning any agents. Pass the selected mode to all Layer 1 and Layer 2 agents in their prompt context.

### Simple Mode (default)

Runs the Layer 1 + Layer 2 fan-out using context available in the current conversation. Validation agents derive context from the plan file and what was passed in the prompt.

**Appropriate when:**
- Plan is self-contained (all context needed is in the plan text)
- Context is fresh in the current conversation window
- Blast radius is clear and confined to a single domain
- No container/infra/worker lifecycle surfaces are touched
- Fewer than 5 blast-radius packages

### Deep Mode

Trigger when **any one** of the following is true:
- Plan blast radius touches container, infra, or worker lifecycle
- Plan has 5+ blast-radius packages
- Plan is a phase in a multi-phase program (prior phase outputs must be loaded)
- Caller explicitly requests deep mode

**How Deep Mode works — run this context-loading step BEFORE spawning any Layer 1 or Layer 2 agents:**

1. Read `process/context/all-context.md` routing table → identify which context groups are relevant to the blast radius
2. Load relevant context group `all-*.md` files (e.g. `container/all-container.md`, `infra/all-infra.md`, `tests/all-tests.md`, `skills/all-skills.md`) — only the groups that apply
3. If phase program: read the latest 1–2 phase reports from inside task folders (`process/features/{feature}/active/{slug}_{date}/{slug}_REPORT_{date}.md`) or legacy `process/features/{feature}/reports/` (read-only legacy path)
4. Read existing test files in the blast radius (gives the Layer 1 test-coverage agent real data, not guesses)
5. Package all loaded material as a **Context Bundle** and pass it to every Layer 1 and Layer 2 agent

**Quality difference in practice:**
- Simple: Layer 1 infra agent infers container port assignments from plan description text
- Deep: Layer 1 infra agent reads the container context group (process/context/container/ or the repo's equivalent routing entrypoint) and knows the real port table (gateway:3000, file-server:3001, app-dev:3002, ctx-gateway:3099, browser-bridge:9377, MITM:9090/9091) — can catch real port conflicts the plan missed

---

## Layer 1 — Four Always-On Dimension Agents

Always run all four, regardless of complexity score. These run in parallel.

| Dimension | Focus | Context to attach |
|---|---|---|
| Infra/setup fit | Does this work with container/worker/proxy architecture? Are target file paths, port numbers, and runtime surfaces correct? | `process/context/all-context.md` routing → container and infra groups (follow routing table for local paths) |
| Test coverage | Is the verification strategy realistic given the test infra? Which tiers apply (fully-automated / hybrid / agent-probe)? | `process/context/tests/all-tests.md` |
| Breaking changes | Identify API contracts, schemas, auth flows, or public contract changes. Are downstream consumers listed and safe? | Plan's Public Contracts and Blast Radius sections |
| Security surface | Quick STRIDE/OWASP scan. Does the plan touch auth, billing, data, secrets, or trust boundaries? | `vc-security` skill context |

**Note:** the security surface dimension INVOKES the `vc-security` skill — do not absorb `vc-security`'s logic here.

### Per-Agent Output Format

```
Dimension: [name]
Status: PASS | CONCERN | FAIL
Findings:
- [finding 1]
- [finding 2]
Confidence: HIGH | MEDIUM | LOW
Notes: [optional context]
```

---

## Layer 2 — Per-Section Feasibility Agents

One agent per plan section or phase. These run in parallel with each other (and may overlap with Layer 1 in time, but Layer 2 results are presented after Layer 1 results are collected).

Each Layer 2 agent must answer four questions — not just "are edit targets findable?":

1. **Mechanical feasibility** — Are the edit target strings present and uniquely matchable in the named files? Can the described create/write steps be executed without collision?
2. **Plan gaps** — What is this section missing that it should include? Are there adjacent files or behaviors that should be updated but are not listed?
3. **Conflicts** — Does anything in this section contradict current file state, other plan sections, or repo conventions?
4. **Risk** — What is the single highest-risk edit in this section and how should the execute-agent sequence or mitigate it?

### Per-Agent Output Format

```
Section: [section name or phase number]
Status: PASS | CONCERN | FAIL
Mechanical feasibility: [verdict + evidence]
Gaps found: [list or "none"]
Conflicts found: [list or "none"]
Highest-risk edit + mitigation: [description]
```

**Warning:** A Layer 2 agent that only confirms edit targets are findable without assessing gaps and conflicts is incomplete and must be re-run with the full four-question prompt.

### [V2-PROBE] Feasibility Probe Emission Rule

When answering the 4 questions above, if a plan section depends on an **untested runtime/system behavior** (network/protocol/runtime/third-party response shape) that cannot be verified by reading source files, the Layer 2 agent MUST:

1. Emit `VC-FEASIBILITY-PROBE-NEEDED: [hypothesis] — cost-class: [class]` and halt.
2. NOT produce the standard per-agent output format.
3. NOT attempt to reason about the untested behavior as if it were mechanical.

**Mechanical checks (NO probe):** edit targets findable by Grep, file exists, schema field present, export names matchable, port in the container table, config key present in env.ts.

Examples of mechanical checks (NO probe): "does `src/env.ts` export a `<SERVICE>_JWT_SECRET` field?" → read the file; "does the container table list port 3000 for the gateway service?" → read the table.

**Probe candidates (emit + halt):** any behavior that requires a running system, live network call, or in-container exec to verify.

Examples of probe candidates: "does the gateway forward the X-Custom-Routing header to the upstream provider at runtime?", "does the container proxy honor the `allow-list` config field when injecting platform keys?", "does the OpenRouter API return `pricing` as a string or a number?".

**Note:** For each CONCERN found, INVOKE `vc-scenario`. For high-risk flagged concerns, INVOKE `vc-predict`.

---

## V3 Synthesis Rules

The vc-validate-agent (or orchestrator) synthesizes all Layer 1 and Layer 2 outputs:

1. Collect all dimension and section verdicts.
2. List all FAILs prominently — any FAIL from any agent must be surfaced.
3. List all CONCERNs grouped by dimension.
4. Note contradictions between agents explicitly; do not resolve them silently.
5. Compute the net gate status:
   - Any FAIL → gate is BLOCKED (unless user explicitly converts to CONDITIONAL)
   - One or more CONCERNs, no FAILs → gate is CONDITIONAL (if user accepts) or re-runs
   - No FAILs, no CONCERNs → gate is PASS
6. Select test gates per the Test Tier Waterfall section.
7. Confirm or update the parallel strategy recommendation based on synthesized findings.

**Note:** Invoke `vc-sequential-thinking` at this synthesis step for contradiction ranking.

---

## Net Gate Derivation Output Format

Present this exact table format after synthesis:

### Layer 1 dimensions

| Layer 1 dimensions | Status |
|---|---|
| Infra fit | PASS / CONCERN / FAIL |
| Test coverage | PASS / CONCERN / FAIL |
| Breaking changes | PASS / CONCERN / FAIL |
| Security surface | PASS / CONCERN / FAIL |

### Layer 2 sections

| Layer 2 sections | Status |
|---|---|
| Section A — [name] | PASS / CONCERN / FAIL |
| Section B — [name] | PASS / CONCERN / FAIL |
| Section N — [name] | PASS / CONCERN / FAIL |

**Totals: [N] FAILs / [N] CONCERNs / [N] PASSes**

**→ Net Gate: [PASS / CONDITIONAL / BLOCKED]**

Decision rules:
- **PASS:** 0 FAILs, 0 CONCERNs. All plan fixes applied. Proceed to EXECUTE.
- **CONDITIONAL:** 0 FAILs, [N] CONCERNs. [N] fixed in plan, [N] as execute-agent instructions, [N] as known-gaps. Proceed to EXECUTE with gaps on record.
- **BLOCKED:** [N] unresolved FAILs. [List each.] Return to PLAN — do not route to EXECUTE until each FAIL is resolved or explicitly converted to CONDITIONAL by user.

---

## Findings Output Format

Use this table format for each dimension's findings (Section I of the V4 menu):

| Finding | Severity | Proposed fix |
|---|---|---|
| [Finding description] | CONCERN / FAIL | [Proposed fix — apply to plan, execute-agent instruction, or backlog artifact] |
| [Finding description] | ✅ PASS | — |

Show PASS findings as `✅ PASS` with `—` in the proposed fix column.

---

## Section IV Output Format

### Proposed Plan Updates

Show the summary below to the user before they approve EXECUTE (before gate V5). These changes are applied to the plan file when the user accepts.

| # | What changes | Where in plan | Why |
|---|---|---|---|
| P1 | [e.g. Add route registration step to Section A checklist] | [Section A — Implementation Checklist] | [Gap found: route not reachable without this step] |
| P2 | [e.g. Correct blast radius: add downstream consumers] | [Blast Radius section] | [Breaking-changes agent found unlisted consumers] |
| PN | [...] | [...] | [...] |

### Execute-Agent Instructions

Concerns that cannot be fixed in plan text — written to the validate-contract for execute-agent to follow:

| # | Instruction | Trigger condition |
|---|---|---|
| E1 | [e.g. Confirm exact file path before writing Section A. If path differs: update edit target, do NOT skip. Document corrected path in phase report.] | Section A entry |
| E2 | [e.g. Container change requires image rebuild. Use docker:build + container recreate via API lifecycle. Never docker cp.] | Section D entry |
| EN | [...] | [...] |

### Backlog Artifacts

| Artifact | Location | What it tracks |
|---|---|---|
| [e.g. test-envelope-regression_NOTE_03-06-26.md] | [process/features/development-process/backlog/] | [Envelope regression test against downstream consumers] |
| [...] | [...] | [...] |

---

## /goal Block Output

After the validate-contract is written to the plan file (V6), the skill stores the /goal block.

**Single plan:** derive the /goal block from plan content (SESSION GOAL, charter, autonomy rules, hard stops, next phase, contract summary, execute start command). Write it to the plan file under a new `## Autonomous Goal Block` section.

**Multi-phase program:** the /goal already exists in the umbrella `## Stable Program Goal`. Do NOT rewrite it — only verify it is current and points to the umbrella path.

The /goal block must include:
- SESSION GOAL
- Autonomy rules
- Hard stops
- Next phase
- Contract summary
- Execute start command

**Multi-phase program rule:** If operating within a phase program (umbrella plan exists), emit the /goal block update automatically without asking — do not prompt the user.

**Single-plan rule:** Store the /goal block during V6. Whether it is printed for copy-paste is decided by the validate-agent's single V5 gate option (`Accept` vs `Accept + print /goal`). This skill does not open an extra prompt or separate user round-trip.

/goal must be fully copy-pastable (plain text block, no special formatting, under 4000 chars).

**V5 during /goal autonomous execution:** agent self-decides (CONDITIONAL → proceed, BLOCKED → backlog + proceed to next non-blocked phase). Skip any print ask — user is not present.
