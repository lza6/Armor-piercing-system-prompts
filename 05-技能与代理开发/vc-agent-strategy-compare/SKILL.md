---
name: vc-agent-strategy-compare
description: "Evaluate 4 execution strategies (sequential, parallel-subagents, workflow, agent-team) for a phase or fan-out task. Outputs 7-signal score table, agent count math, cost guards, and strategy recommendation."
argument-hint: "[phase context description or fan-out task description]"
trigger_keywords: execution strategy, parallel agents, strategy comparison, agent count, fan-out recommendation
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.2.0"
---

# vc-agent-strategy-compare

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Evaluate 4 execution strategies for any RIPER-5 phase or fan-out task. Computes the 7-signal score, shows explicit agent count math, applies cost guards, and outputs a ranked strategy recommendation.

All strategies use the existing vc-system agents and skills. The question is only how they are orchestrated.

---

## Model Selection Policy

Every spawned agent — across ALL four strategies (sequential, parallel subagents, dynamic workflow, agent team) — defaults to **sonnet**. Spawn opus ONLY when the agent is carrying out real source-code or build execution (writing/editing code, running builds, applying migrations). Planning, research, analysis, validation, review, and synthesis all run on sonnet.

Concrete rules:

- **Default = sonnet** for any teammate, parallel subagent, or `agent()` call in a workflow. State the model explicitly when spawning.
- **opus ONLY for execution work**: the agent that actually implements code (the vc-execute-agent leg, or a workflow `agent()` step whose job is to modify source / run a build). The RIPER-5 phase that runs on opus is EXECUTE; all other phases (RESEARCH, SPEC, INNOVATE, PLAN, VALIDATE, UPDATE PROCESS) run on sonnet.
- **Agent-team members**: assign sonnet to reviewers, researchers, validators, and planners; assign opus only to the teammate doing real code execution.
- **Dynamic workflows**: pass `model: 'sonnet'` on `agent()` calls by default; pass `model: 'opus'` only on the execution stage that writes code or runs builds.
- **Parallel subagents**: fan-out investigation/review subagents are sonnet; only an implementing subagent is opus.

Rationale: opus is reserved for the one place judgment-under-execution materially changes code quality. Everything upstream is sonnet-cheap without quality loss, and this keeps fan-outs (which multiply agent count) on the cost-efficient tier. This mirrors the live agent frontmatter: `vc-execute-agent`, `vc-fast-mode-agent`, and `vc-quick-fix-agent` are opus; every other vc-agent is sonnet.

---

## Mode Selection

This skill runs in one of two modes. Choose based on context availability and decision stakes.

### Simple Mode (default)

Score the 7 signals from information already present in the conversation context. No additional file scanning.

Use Simple Mode when:
- File count and blast radius are already clear from the current plan or research summary
- Test infrastructure status is already mentioned or irrelevant
- The strategy decision is not gating a phase program kickoff
- No signal is ambiguous given current context

Output format: `Signal 3: +1 (estimated 5-10 files)`

### Deep Mode

Run targeted codebase scans BEFORE scoring signals, then score each signal with concrete evidence.

**Trigger conditions — any one is sufficient:**
- Signal 3 (file count / directions) cannot be accurately determined from context alone
- Signal 5 (test infra maturity) is unknown — no test files have been mentioned yet
- The strategy recommendation will gate a phase program kickoff (high-stakes decision)
- Caller explicitly requests deep mode (e.g. "run deep mode", "scan first")

**Scans to run before scoring:**

Signal 3 scan — count actual touchpoint files:
```bash
find . -path '*/active/*' -name '*.md' | head -5
grep -A 30 "Touchpoints" [plan path] | grep -E "^\s*-"
```

Signal 5 scan — assess test infra maturity:
```bash
# Check if local test script exists
grep -l "test:local" packages/*/package.json apps/*/package.json 2>/dev/null | head -5
# Count test files in blast radius directories
find [blast-radius-dirs] -name '*.test.ts' -o -name '*.spec.ts' 2>/dev/null | wc -l
# Check validate-contract for tier assignments
grep -A 5 "tier:" [plan path] | head -20
```

Signal 6 scan — find independent phases for parallelization:
```bash
grep -A 5 "Phase Ordering\|phase.*depend" [umbrella plan path] | head -20
```

Signal 7 scan — check prior parallel execution outcomes:
```bash
find process/features/[feature]/active/ process/features/[feature]/completed/ -name '*_REPORT_*' 2>/dev/null | grep -i "parallel\|execute" | head -5
```

After scanning, score each affected signal with concrete evidence:
`Signal 3: +1 (14 files confirmed via Touchpoints scan: packages/api/src/router/billing.ts, apps/web/src/components/billing/*, ...)`

---

## When To Invoke

Invoke this skill at these five RIPER-5 checkpoints:

1. **RESEARCH** — when 2+ distinct investigation directions are identified before fanning out (Checkpoint 1: Research Fan-Out)
2. **INNOVATE** — after 2-3 approaches are surfaced and before the Decision Summary is locked (Checkpoint 2: Innovate Fan-Out)
3. **PLAN** — when 3+ phase plans will be created (phase program detected; Checkpoint 4: Phase-Program Validation Fan-Out)
4. **VALIDATE** — at V4 (Validate Menu) — always runs as part of the mandatory VALIDATE sequence
5. **UPDATE PROCESS** — when post-execute review involves multiple dimensions simultaneously (Checkpoint 5: Post-Execute Review Fan-Out)

### Phase-END Invocation Rule

Invoke at the END of each RIPER-5 phase to recommend the execution strategy for the NEXT phase. The skill receives the current phase's output as context and produces a strategy recommendation that the orchestrator carries into the next phase handoff.

### Fan-Out-Level Invocation Rule

When any agent is about to spawn multiple parallel subagents for work WITHIN the current phase, invoke `vc-agent-strategy-compare` FIRST to determine the execution method.

- **Input context**: describe the specific fan-out task (not the whole phase). Example: "run feasibility checks on 5 phase plans" or "review 3 independent API surface changes."
- **Output**: Workflow / Parallel-subagents / Agent team / Sequential recommendation for THIS fan-out only.
- This prevents defaulting to parallel subagents for tasks that are better served by a deterministic workflow or a single sequential agent.

### Orchestrator Pre-Spawn Rule

Before any multi-file edit begins, the orchestrator surfaces a strategy recommendation from this skill and waits for user confirmation before spawning `vc-execute-agent`. This is not optional for non-trivial plans (blast radius ≥ 3 files or any high-risk class present).

---

## 7-Signal Scoring Table

Count how many signals are present. Each signal counts as 1. Use the score to select a strategy threshold.

### Seven Signals

| ID | Signal | Present? |
|----|--------|----------|
| S1 | **Multi-package scope** — files touch 3+ workspace packages | [ ] |
| S2 | **Schema/API/auth surface touched** — plan or research identifies changes to DB schema, public API contracts, or auth/identity flows | [ ] |
| S3 | **3+ viable directions** — research or innovate surfaced 3+ meaningfully different approaches or investigation areas | [ ] |
| S4 | **Phase program classification** — the work was classified as a phase program (3+ phases) | [ ] |
| S5 | **User requests depth** — the user explicitly asks for depth ("go deep", "explore all options", "compare thoroughly") | [ ] |
| S6 | **High-risk class in plan** — plan's Blast Radius or Public Contracts section names auth/identity, billing/credits, schema/migration, public API, container/proxy/gateway, or secrets/trust-boundary | [ ] |
| S7 | **5+ files in blast radius** — plan's Blast Radius section lists 5 or more distinct files | [ ] |

**Total score**: [0–7]

### Threshold Table

| Score | Label | Recommended strategy |
|-------|-------|---------------------|
| 0–1 | LOW | Sequential — one vc-agent at a time. Do not mention fan-out. |
| 2–3 | MEDIUM | Parallel subagents — spawn one vc-agent per direction; orchestrator merges outputs. |
| 4+ | HIGH | Workflow or Agent team — workflow for deterministic step-by-step pipelines (automated, predictable sequence); **agent team** (named teammates + shared task list: TeamCreate + TaskCreate/TaskUpdate + Agent with team_name/name + SendMessage, tracked by TaskList — **NOT** parallel subagents) when specialists must share what they find while still working (real-time coordination). Parallel subagents are fire-and-forget and cannot talk to each other; agent-team members can send messages mid-run. |

**Auto-skip rule**: single-file or trivial changes always use sequential regardless of score. Do not mention other strategies for trivial changes.

**Fit note**: the right strategy is the one that fits the work — not the highest tier. Sequential is correct for trivial changes. Workflow is correct for deterministic pipelines and full RIPER-5 automation. Agent team is correct only when 2+ specialists need to coordinate mid-execution.

---

## Strategy Options Table

ALL 4 strategies must always be evaluated and presented. Never omit one.

| Strategy | How it works in the vc system | Agent count math | Cost guard | Best fit |
|----------|-------------------------------|------------------|------------|----------|
| **Sequential** | One vc-agent at a time in strict RIPER-5 order: orchestrator spawns vc-research-agent → waits → spawns vc-innovate-agent → waits → etc. Each agent gets the previous agent's output. Single context window per phase. | 1 agent per phase (6 total for full RIPER-5) | None | Trivial/single-file changes; iterative `/goal` phase-program execution where steps are known but parallelism adds no benefit |
| **Parallel subagents** | Orchestrator spawns multiple vc-agents simultaneously via the `Agent` tool, each investigating one independent direction. Each agent loads its own context, invokes its own skills (vc-scout, vc-docs-seeker, vc-sequential-thinking), and returns a result. Orchestrator merges. | 4 (Layer 1 dimensions) + N (one per direction) + 3 (optional validation fan-out) = 7–15 typical | >30: show breakdown before proceeding; >100: ask explicit confirmation | 5+ independent directions (e.g. 5 separate codebase areas, 5 phase plans to validate simultaneously) with no mid-task communication needed between agents |
| **Workflow** | Full RIPER-5 pipeline as a deterministic `Workflow` script. Each phase is a `phase()` + `agent()` call. Supports `pipeline()` for per-item fan-out, `parallel()` barriers when all-results are needed before proceeding, and loop-until-dry patterns. Can run the full RESEARCH→VALIDATE→EXECUTE→TEST sequence automatically with built-in gates. | P (phase steps) × A (agents per step) × I (iterations) = P × A × I; up to 1000 agents, 16 concurrent | >30: show breakdown; >100: ask confirmation | Full RIPER-5 automation, TDD fan-out loops, metric iteration, large sweeps (lint/test/migrate); unknown item count upfront; quality > cost |
| **Agent team** | Claude Code's built-in team feature: `TeamCreate` provisions named specialist teammates. Each teammate gets a `TaskCreate` assignment scoped to their specialty (e.g. one runs vc-research-agent, another runs vc-validate-agent, another runs vc-execute-agent). `SendMessage` enables mid-execution coordination. `TaskList` tracks all in-flight work. | M (members) × R (rounds) = M × R; keep M ≤ 6, R ≤ 3 unless scope demands it; typically 6–18 total | >6 members: show each member's role and ask explicit confirmation | 2+ workstreams that must share findings mid-execution (e.g. a security reviewer feeds blockers to the implementer before the implementer finishes); named specialist roles known upfront; adversarial challenge tasks |

### Cost Guard Rules

- **>30 agents total**: show the full breakdown (strategy × count math) so the user can judge before proceeding.
- **>100 agents total**: show breakdown AND ask for explicit confirmation before spawning.
- **>6 team members** (vc-team only): show each member's role and ask for explicit confirmation.

**Note**: V5 "Accept" in the VALIDATE sequence satisfies both the cost-guard confirmation and the plan-approval confirmation in one gate. The cost guard is surfaced at V4 (Validate Menu) so the user sees it before deciding at V5. No separate yes/no prompt is required for the cost guard alone.

---

## Recommendation Output

After computing the score and filling the table, output a single recommendation block. Use plain English labels alongside technical ones so readers unfamiliar with the system can follow along.

```
Score: [N]/7 — signals: [list present signal IDs]
Recommended strategy: [Sequential (one agent at a time) | Parallel subagents (independent work, no coordination) | Workflow (automated pipeline) | Agent team (specialists coordinating live)]
Agent count: [explicit math per row above]
Model: [sonnet for all spawned agents; opus ONLY for the code-execution leg — see §Model Selection Policy]
Cost guard: [triggered / not triggered]
Rationale: [one sentence on dominant signal and why this strategy fits]
```

### Strategy-by-Fit Rules

Use these rules to select and justify the recommendation — not the threshold table alone:

- **Sequential**: right for trivial or single-file changes; right for `/goal` phase-program execution where each phase clearly depends on the previous. Hard limit: single context window per phase. Each vc-agent (vc-research-agent, vc-plan-agent, etc.) runs one at a time; orchestrator hands off between them.
- **Parallel subagents**: right when there are 5+ independent items in different file domains with no mid-task communication needed. Each spawned agent uses the same vc-system agents and skills but works on a scoped slice. Orchestrator must stay clean (parent context must not accumulate all output). Time savings: 50–70%. Cost: approximately N× linear token multiplier.
- **Workflow**: right for deterministic pipelines (full RIPER-5 automation, TDD loop, metric iteration, quality gate sequences), tasks too big for a single context window, or when the item count is unknown upfront. The workflow script calls vc-system agents as `agent()` calls in `pipeline()` or `parallel()` steps. Quality > cost priority. Up to 1000 agents, 16 concurrent. Total tokens roughly the same as sequential — most cost-effective for large volume.
- **Agent team**: right when 2+ specialist workstreams must share findings mid-execution (not just consume the same input), when named roles are known upfront (e.g. security reviewer + implementer + tester coordinating live), or when an adversarial challenge pattern is required. Uses Claude Code `TeamCreate` + `TaskCreate` + `SendMessage`. Each teammate runs its own vc-agent and skills. NOT for simple fan-out where agents work independently — use parallel subagents for that. **Mechanism:** a team shares a TaskList and uses SendMessage to coordinate mid-run; parallel subagents have NO inter-agent channel and CANNOT coordinate — so any task that needs mid-execution coordination (e.g. blast-radius non-overlap across phase plans) MUST be agent-team, never parallel subagents.

---

## Phase Program Rule

When a plan describes a program with 3+ phases (phase program classification, signal S4 present):

- **Sequential is NEVER valid** for plan creation fan-out OR validate fan-out across the phases.
- **3+ phase-plan CREATION default: AGENT TEAM.** Phase plans share files (CLAUDE.md, agent .md files) and MUST coordinate blast-radius non-overlap + dependency declarations — only an agent team can do this (TeamCreate + shared TaskList + SendMessage). Fire-and-forget subagents cannot communicate, so they cannot keep blast radii disjoint and are the WRONG strategy for plan creation. (Outer-PVL VALIDATE fan-out across already-written phase plans, where each validator reads one finished plan with no cross-talk, MAY use independent read-only subagents — see the reconciliation note below.)
- **Reconciliation (CREATION vs read-only VALIDATE fan-out):** validating N already-written plans needs no inter-agent talk, so bare parallel subagents are valid there; plan CREATION needs cross-talk, so it is agent-team. Note `orchestration.md` even describes Outer PVL as an *agent-team* — prefer agent-team for both and reserve bare parallel subagents only for truly independent read-only fan-out.
- **If phases have complex interdependencies requiring mid-draft communication** (e.g., phase 2 plan depends on design decisions surfaced during phase 1 planning): use agent team instead. Assign one teammate per phase, use `SendMessage` for cross-phase coordination.
- **If the total phase count is unknown upfront** (e.g., the program scope is discovered incrementally): use workflow so the pipeline can expand without re-spawning the orchestrator. Each `agent()` call in the workflow runs the appropriate vc-agent for that phase.

---

## Strategy Reference

### 1 — Sequential (one vc-agent at a time)

The orchestrator runs one vc-agent, waits for it to complete, then routes to the next.

```
orchestrator → vc-research-agent → (result) → vc-innovate-agent → (result) → vc-plan-agent → ...
```

- Each agent invokes its own skills (vc-scout, vc-docs-seeker, vc-sequential-thinking, etc.) independently.
- Simplest option. No fan-out overhead. Single context window per phase.
- Hard limit: cannot parallelize. If a phase has 5 independent sections, all 5 run back-to-back.
- Best for: standard single-feature RIPER-5 runs, `/goal` phase-program execution, trivial fixes delegated directly to vc-execute-agent.

### 2 — Parallel Subagents (one agent per direction)

The orchestrator spawns multiple vc-agents simultaneously via the `Agent` tool. Each agent goes in one direction independently, invokes its own skills, and returns a result. The orchestrator merges all results before proceeding.

```
orchestrator ──► vc-research-agent (direction A)
             ──► vc-research-agent (direction B)   (all run simultaneously)
             ──► vc-research-agent (direction C)
             └── merge all results → route to vc-innovate-agent
```

- Parent context must stay clean — pass only scoped input to each agent; collect structured output.
- Time savings: 50–70% vs sequential for the fan-out step. Cost: approximately N× token multiplier.
- NOT appropriate when one agent's findings should influence another agent mid-run — use agent team for that.
- Best for: parallel RESEARCH across 5+ distinct codebase areas; parallel VALIDATE across 5+ phase plans; parallel dimension checks in PLAN.

### 3 — Dynamic Workflow (full RIPER-5 pipeline)

A deterministic `Workflow` script orchestrates the vc-system agents as programmatic steps. Each `agent()` call in the script corresponds to a vc-agent running its phase. `pipeline()` handles per-item fan-out without a barrier. `parallel()` handles all-results-needed barriers.

```
workflow script
  phase('Research')   → agent('run vc-research-agent for feature X', {agentType: 'vc-research-agent'})
  phase('Plan')       → pipeline(items, item => agent('write phase plan for ' + item))
  phase('Validate')   → parallel(plans.map(p => () => agent('validate plan ' + p)))
  phase('Execute')    → agent('run vc-execute-agent for plan path', {agentType: 'vc-execute-agent'})
```

- Supports loop-until-dry, budget-aware iteration, and unknown item counts.
- Each `agent()` call runs the appropriate vc-system agent and its skills.
- Up to 1000 agents / 16 concurrent. Total tokens roughly equal to sequential — work is distributed.
- Best for: full RIPER-5 automation across many items, TDD fan-out loops, lint/test sweeps, phase-program pipelines where count is unknown upfront.

### 4 — Agent Team (multiple specialist teammates)

Claude Code's built-in `TeamCreate` feature provisions named teammates. Each teammate is assigned a specialized role via `TaskCreate`. Teammates communicate via `SendMessage`. `TaskList` tracks in-flight work.

```
TeamCreate("security-review") → teammate A: runs vc-validate-agent (security dimension)
TeamCreate("implementation")  → teammate B: runs vc-execute-agent (implements)
TeamCreate("test-coverage")   → teammate C: runs vc-tester (writes/runs tests)
SendMessage: A → B (security findings mid-implementation, not after)
```

- Each teammate runs its own vc-agent (vc-research-agent, vc-plan-agent, vc-execute-agent, vc-tester, vc-debugger, etc.) and invokes the relevant skills for their specialty.
- Named roles known upfront. Mid-execution coordination via `SendMessage`.
- Highest cost — all teammates are active simultaneously.
- NOT appropriate for simple fan-out where agents work independently — use parallel subagents for that.
- Best for: security reviewer + implementer + tester coordinating live; adversarial plan challenge (one agent proposes, another attacks); multi-specialist investigation where findings from specialist A must reach specialist B before B finalizes.

---

## Source References

- `process/development-protocols/orchestration.md` — Two-Tier Fan-Out Escalation, Parallel Fan-Out Checkpoints
- `process/development-protocols/orchestration.md` §VALIDATE Gate — skip conditions, gate verdicts, BLOCKED escalation path
