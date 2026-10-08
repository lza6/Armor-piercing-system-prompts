---
name: vc-generate-phase-program
description: "Generate kickoff artifacts for a multi-phase program: umbrella plan, Program Goal Charter, session-goal block, per-phase plan stubs, and the 7-step per-phase inner loop reference."
argument-hint: "[program name and goal description]"
trigger_keywords: phase program, umbrella plan, session goal block, kickoff template, multi-phase
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-generate-phase-program

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Generate kickoff artifacts for a multi-phase RIPER-5 program: umbrella plan, Program Goal Charter,
session-goal block, per-phase plan stubs, and the 7-step per-phase inner loop reference
(`R → I → P → PVL → E → EVL → UP`, which SKIPS SPEC).

The 7 steps are: Research → Innovate → Plan-Supplement → PVL (plan-validate loop — the gate that confirms the plan is ready before EXECUTE runs) → Execute → EVL (execute-validate loop — the confirmation run after EXECUTE, re-running gate tests independently) → Update-Process.

Source of truth: `process/development-protocols/phase-programs.md`

---

## When To Invoke

Invoke this skill at PLAN phase when phase-program initiation is detected (3 or more dependent
phases), or at ORCHESTRATOR when the incoming request involves 3 or more dependent phases.

Signal patterns that trigger this skill:

- the work naturally breaks into 3 or more dependent phases
- each phase needs its own validation gate before the next phase starts
- the work spans multiple packages, services, or runtime surfaces
- the user wants high-confidence progress with durable checkpoints
- repeated research is needed because new facts will emerge during execution

Do not invoke for a simple one-session feature or a small bug fix. Use the normal RIPER flow instead.

---

## Kickoff Procedure

Before creating any plan files, follow these steps in order:

**Step 1 — invoke `vc-agent-strategy-compare`.**
For programs with 3 or more phases, output will always recommend parallel-subagents (one per phase
plan) or dynamic workflow. Plans are created in the recommended parallel mode, not sequentially.

**Per-phase strategy invocation (mandatory):** For each individual phase during scaffold, invoke `vc-agent-strategy-compare` with that phase's scope before writing that phase's plan stub — not once at the program level. Each phase may have a different recommended execution strategy.

**Step 1a — Read template files.**
Before creating any plan artifacts, execute `Read` on both template files:
- `.claude/skills/vc-generate-phase-program/templates/umbrella-plan-template.md`
- `.claude/skills/vc-generate-phase-program/templates/phase-stub-template.md`

Use the template structure as the basis for all generated artifacts. Substitute `{placeholder}` values with program-specific content. Do not reconstruct structure from memory.

**Step 2 — research the full problem space.**
Read `process/context/all-context.md`. Run `find process/context/ -type f` and
`find process/development-protocols/ -type f`. Load domain-specific context files relevant to the
program. Understand the full problem space before proposing any structure.

**Step 3 — emit a kickoff recommendation (stop for approval before creating files).**
Present the recommendation using the format in "Kickoff Recommendation Format" below. Do not create
plan artifacts yet. Stop and wait for user approval.

**Step 4 — after approval, create the required artifacts:**
- feature folder under `process/features/{feature}/` with subdirs `active/`, `completed/`, `backlog/` (no `reports/` or `references/` — new repos omit these deprecated sibling dirs)
- ONE program task folder in `active/`: `active/{program-slug}_{date}/` holding the umbrella `_PLAN_`, every phase `_PLAN_`, every phase `_REPORT_`, the phase registry, and `_REF_` files — all FLAT (no per-phase subfolders)
- the umbrella/orchestration plan: `active/{program-slug}_{date}/{program-slug}-umbrella_PLAN_{date}.md` — the filename MUST carry the literal `umbrella` token (enforced by `validate-plan-artifact.mjs` and `validate-umbrella-artifact.mjs` for any plan whose frontmatter declares `phase: umbrella`)
- one direct phase plan per phase, FLAT in the same program folder: `active/{program-slug}_{date}/phase-NN-{slug}_PLAN_{date}.md`

Per **task-folder artefact colocation**, the umbrella plan, every phase plan, the phase registry, and all reports/references each live FLAT INSIDE the ONE program task folder; never write program artefacts to the deprecated sibling `reports/` or `references/` dirs, and never create per-phase subfolders. The whole program folder moves as a unit on completion.

**Step 5 — emit the compressed session-goal block in chat.**
Once the umbrella plan and its Program Goal Charter exist, emit the session-goal block directly in
chat (not to a file). See "Kickoff Recommendation Format" step 5 for the exact shape and the 4000-
character limit rule.

---

## Goal Block Requirements

Every phase program umbrella plan must include a `## Stable Program Goal` section containing a copy-pasteable /goal block. Requirements:

- **Hard limit:** ≤ 4000 characters (the /goal command rejects longer blocks). Verify char count before writing.
- **Required sections (in order):** TARGET / PER-PHASE LOOP / HARD STOPS / SAFETY / TEST GATES / VALIDATE CONTRACT / START
- **PER-PHASE LOOP must state:**
  - Loop steps (7-step inner loop `R → I → P → PVL → E → EVL → UP`, SKIPS SPEC): 1 RESEARCH → 2 INNOVATE → 3 PLAN-SUPPLEMENT → 4 PVL → 5 EXECUTE → 6 EVL → 7 UPDATE-PROCESS
  - PVL never skipped rule
  - Placeholder contract = blocked rule
  - Every subagent first action: vc-context-discovery + vc-plan-discovery (once available)
  - Every phase-END: invoke vc-agent-strategy-compare
  - Correct test tier names: automated / hybrid / agent-probe
- **TEST GATES** must list all 5 validator commands with full paths
- **START** must name the current phase and loop step explicitly
- When updating the goal block after phases complete, re-verify char count before writing — compress if needed, never truncate required sections

---

## Kickoff Template

Use this template as the starting prompt when handing off to a program-capable agent or when
opening a new long-running session. Replace all placeholders with real program content.

```text
Build [PROJECT OR PROGRAM NAME] end-to-end using the repo's phase-program workflow.

Goal:
- [state the real end goal in one or two sentences]

Scope:
- Start by reading `process/context/all-context.md`
- Use `process/development-protocols/phase-programs.md`
- Treat this as a large multi-phase program, not a normal single-plan task
- First do the necessary research to understand the whole problem space
- First recommend:
  - whether this should be a standard complex plan or a phase program
  - the proposed feature folder
  - the umbrella/orchestration plan shape
  - the proposed phase sequence
  - the recommended immediate next action
- Present that recommendation clearly and stop for approval
- Only after approval, create or confirm:
  - one feature folder
  - one umbrella/orchestration plan
  - one direct phase plan per phase
- Make the phase boundaries explicit
- Define what each phase green check proves
- Separate foundation proof from later expansion if they are different scopes

Execution rule:
- Do not execute the whole program at once
- For each phase, follow the required 7-step inner loop `R → I → P → PVL → E → EVL → UP` (SKIPS SPEC):
  research -> innovate -> plan-supplement -> PVL (validate-contract) -> execute -> EVL (validate + regression) -> update-process (capture + commit + move-on)
- Re-research at the start of every phase before implementation
- After validation, run regression checks against previously verified surfaces that overlap with this phase's blast radius
- Commit execution changes via vc-git-manager before moving to the next phase
- Run inter-phase UPDATE PROCESS to archive the completed phase and capture learnings
- Do not mark a phase `✅ VERIFIED` without both phase evidence and regression evidence
- If blocked, document the blocker, safest next action, and update later phase plans/reports so the work survives compaction

Deliverables:
- initial recommendation on plan shape, sequencing, and next actions
- feature folder under `process/features/{feature}/`
- umbrella/orchestration plan
- phase plans
- durable reports and references as phases execute
- context updates when durable operational knowledge changes

Working instruction:
- Proceed phase by phase
- Do not stop at analysis if the selected phase is approved and unblocked
- Do not silently widen scope across phases
- Keep status honest and keep future work split cleanly
```

---

## Practical Operator Kickoff

Shorter version for day-to-day reuse when the full template is overkill:

```text
Build [NAME] as a phase program per process/development-protocols/phase-programs.md.

Goal: [1-2 sentences]

First: recommend structure (feature folder, phases, immediate next action). Stop for approval.
Then: advance one phase at a time using the 7-step inner loop `R → I → P → PVL → E → EVL → UP` (SKIPS SPEC): research -> innovate -> plan-supplement -> PVL -> execute -> EVL (validate + regression) -> update-process (capture + commit + move-on).
```

---

## Template Files

Two copy-pasteable template files exist for generating structurally correct artifacts:

| Template | Path | Use when |
|---|---|---|
| Umbrella plan | `.claude/skills/vc-generate-phase-program/templates/umbrella-plan-template.md` | Creating a new umbrella/orchestration plan |
| Phase stub | `.claude/skills/vc-generate-phase-program/templates/phase-stub-template.md` | Creating a new per-phase plan stub |

**Mandatory Read steps:** Before writing any umbrella plan or phase stub, execute:
1. `Read(".claude/skills/vc-generate-phase-program/templates/umbrella-plan-template.md")`
2. `Read(".claude/skills/vc-generate-phase-program/templates/phase-stub-template.md")`

The template files are the source of truth for required sections and placeholder wording. Do not reconstruct structure from memory.

---

## Kickoff Recommendation Format

Before creating any plan files for a new large program, present a short recommendation with these
five items:

**1. Program fit**
- should this be `standard complex` or a `phase program`
- why

**2. Recommended structure**
- feature folder name
- umbrella plan name
- proposed phase list in order

**3. Recommended immediate next action**
- what should happen now
- what should wait until later

**4. Approval checkpoint**
- ask whether to proceed with creating the plan artifacts

**5. Compressed session-goal block (printed in chat)**
Once the umbrella plan and its Program Goal Charter exist, emit a compressed, copy-pasteable
"session goal" block directly in chat (do NOT write it to a file). This is the launch packet a user
pastes to start an unattended, long-running session. Keep it to roughly 8-12 lines with this shape:

```text
SESSION GOAL: [PROGRAM NAME]
Charter + umbrella plan: process/features/{feature}/active/{program-slug}-umbrella_{date}/{program-slug}-umbrella_PLAN_{date}.md
Autonomy: Run autonomously under this persistent goal. Execute phases on your own
recommendation via the 7-step inner loop `R → I → P → PVL → E → EVL → UP` in phase-programs.md
(the inner loop SKIPS SPEC); report conflicts, errors, and learnings in the phase report (the
report is the communication channel, not a question). Only pause for outward-facing /
irreversible / costful / destructive actions (see feedback_autonomous_phase_execution.md).
Hard stop conditions / safety constraints:
- [hard safety constraint 1 from the charter]
- [hard safety constraint 2 from the charter]
Next phase: process/features/{feature}/active/{program-slug}_{date}/phase-NN-{slug}_PLAN_{date}.md
Validate contract: [path to validate-contract or "inline in plan"]
Execute start: [fully-auto commands] | [e2e spec] | [probe scenario] | high-risk pack: [yes/no]
```

The block must name the charter/umbrella plan path, state the autonomy rule (citing
`feedback_autonomous_phase_execution.md`: execute phases on own recommendation under a persistent
goal; only pause for outward-facing/irreversible/costful actions), and list the charter's hard
stop-conditions/safety constraints verbatim.

**Hard rule:** the session-goal block MUST be under 4000 characters total — it is pasted into a
persistent `/goal` whose ceiling is ~4000 chars. If the program's safety constraints and
definition-of-done won't compress under 4000 chars, summarize and reference the charter's plan path
for the full detail rather than inlining everything.

---

## Autonomous Session-Goal Variant

This is an explicit opt-in variant. It does NOT weaken the default supervised loop; it only applies
when the user sets a persistent autonomous session-goal (e.g. a standing `/goal`).

When the user sets a persistent autonomous session-goal:

- the per-phase Execution Approval Checkpoint (the PVL step — step 4 of the 7-step inner loop) is treated as STANDING-GRANTED.
  The agent does not pause for approval between phases.
- the agent self-decides, executes, and reports learnings after each phase. On failure it diagnoses,
  writes a new plan/fix, and continues.
- the safety boundary REPLACES the approval gate: never take irreversible or costful actions
  (deploys, live/costful provider gates, billing, destructive schema/data ops). These are DEFERRED
  AND REPORTED — never executed and never paused-on.
- every step must stay rollback-able: commit each phase before the next, keep process/plan commits
  separate from execution commits, and prefer disposable targets.
- all other loop steps still apply unchanged: re-research at phase entry, validate, regression check,
  durable capture, commit, inter-phase UPDATE PROCESS, and honest phase status.

**Shared-runtime 2-tier policy:** direct interaction with the shared E2E container — including
rebuilding the image and recreating/restarting it via the project's dedicated managed script — is
normal, autonomous test work. It is NOT forbidden. Only irrecoverable persistent-state loss,
prod-state mutation, production image push/deploy, and live/costful gates are deferred-and-reported.
See your project's container/test context docs for the exact sanctioned commands. Do not re-list
those commands here.

| Tier | Autonomous? | Examples |
|---|---|---|
| GREEN — fully autonomous (the default) | yes | direct shared-container interaction — exec, read logs, send messages, edit/write files, push secrets, probe health; run any documented script in your project's test context docs; rebuild the image and recreate/restart the shared container via the project's managed script to apply changes (named volume preserved); create/rebuild/recreate/remove E2E-owned disposable targets freely; reversible stop/start parking of the shared container, restored and verified leave-as-found |
| RED — defer-and-report (never autonomous) | no | wipe/delete the shared container's named volume (irrecoverable data loss) or otherwise destroy persistent state without recovery; mutate production DB/storage/streaming state; push production images or deploy; run live/costful/provider-backed gates without per-lane approval |

---

## The Required Per-Phase Loop

The canonical per-phase loop is the **7-step inner loop** `R → I → P → PVL → E → EVL → UP` (it
SKIPS SPEC — SPEC runs once in the outer program loop). The detailed orchestration steps below are
the prose EXPANSION of those 7 steps: RESEARCH (1) → INNOVATE (decide approach) → PLAN-SUPPLEMENT →
PVL = the execution approval checkpoint / validate-contract (2) → EXECUTE (3) → EVL = validate +
regression + regression-found workflow (4–6) → UPDATE-PROCESS = durable capture + commit +
inter-phase UPDATE PROCESS + move-on (7–10). The numbering below is orchestration detail, not a
separate loop.

For every phase, run this loop:

1. **Research subagent**
   - reread the selected phase plan
   - reread the latest relevant reports, references, and context docs
   - inspect codebase drift since the plan was written
   - supplement the phase plan or create a research report if new facts matter
   - Also always read `process/context/all-context.md` and run `find process/context/ -type f` to
     see all available context files. Note: protocol files needed for this phase
     (orchestration.md, plan-lifecycle.md, phase-programs.md) live in
     `process/development-protocols/` and must be explicitly read — they are not discoverable via
     the context router.

2. **Execution approval checkpoint**
   - summarize what changed since planning
   - identify risks, scope adjustments, and exact gates
   - get user approval before substantial implementation
   - For phase programs with a standing /goal, VALIDATE runs here and is treated as
     STANDING-APPROVED. The orchestrator does not pause for validate approval between phases; the
     validate-contract is written and execution proceeds. STANDING-APPROVED means the approval pause
     is skipped — not the VALIDATE run itself. The validate-contract must still be written for each
     phase; a phase cannot reach VERIFIED without it.

3. **Execute subagent**
   - implement only the selected phase scope
   - stop if the work no longer matches the approved plan
   - **Per-section test-gate loop:** After completing each checklist section, immediately run the validate-contract test gates for that section — do not batch all test gates to the end. Fix failures inline before moving to the next section.
   - **Test-failure escalation ladder:**
     1. If the failing test is in this phase's blast radius → fix inline, re-run until green, then continue.
     2. If the fix is out of scope (would change a different module) → create a follow-up phase plan inside a new task folder under `process/features/{feature}/active/{new-slug}_{date}/`, document the gap, and continue.
     3. If there is no fix path → classify as backlog artifact, add to known-gaps in the phase report, and continue without blocking.

4. **Validate subagent**
   - run the exact phase gates
   - inspect artifacts, logs, DB state, screenshots, traces, or runtime evidence as required
   - decide whether the phase is genuinely green, blocked, or only partially proven
   - run all applicable test tiers per the validate-contract: fully automated (fix failures and
     re-run until green), hybrid (record outcome, fix if in blast radius), agent probe (record
     judgment). Phase cannot advance to VERIFIED until all in-blast-radius tiers are green.

5. **Regression checkpoint**
   - run the narrowest representative check against previously verified surfaces that overlap with
     this phase's blast radius
   - see "Regression Checkpoint Standard" below for scope selection and evidence format
   - if all regression checks pass, proceed to durable capture
   - if any regression is found, follow "Regression-Found Workflow" before advancing

6. **Regression-found workflow** (conditional)
   - only enters this step when step 5 finds a regression
   - classify, fix or route, revalidate, then return to step 5
   - see "Regression-Found Workflow" below for the full decision tree

7. **Durable capture**
   - update the phase report with commands, outcomes, deviations, and blockers
   - include regression check results (pass or fix-and-revalidate) in the report
   - update context docs if durable operational knowledge changed
   - update later phase plans if the new learning changes future work
   - if execution reveals a concrete missing downstream lane, create the new follow-up phase plan or
     backlog artifact instead of leaving the follow-up only in chat
   - keep the parent or umbrella plan in sync when follow-up routing or phase sequencing changes

8. **Commit checkpoint**
   - if the phase produced implementation changes, recommend `vc-git-manager` for a logical
     execution commit before continuing
   - keep process/plan/context artifact commits separate from execution commits
   - do not defer the commit to a later phase — stale worktrees make regression checking unreliable

9. **Inter-phase UPDATE PROCESS**
   - route through UPDATE PROCESS to archive the completed phase plan and capture learnings
   - update context docs, reports, and downstream phase plans as needed
   - this step is mandatory between phases, not optional — phase outputs must survive compaction

10. **Move-on recommendation**
    - name the exact next valid state after the phase closeout
    - if the next phase is already known, name the exact next phase plan path
    - if the current phase is not really green, keep the work on the same phase instead of
      pretending to advance

This loop is mandatory. Do not jump straight from phase plan to implementation without a fresh
research pass on large programs.

---

## Regression Checkpoint Standard

After validating the current phase's own gates (step 4), check that previously verified work still
holds.

**Scope selection:**

- identify previously verified surfaces that overlap with this phase's blast radius
- run the narrowest representative check for each overlapping surface
- if the phase touches shared infrastructure (DB, container, proxy, auth), include at least one
  check from each earlier phase that depends on that infrastructure
- if no earlier phases are verified yet, skip this step

**Evidence format:**

Record regression results in the phase report as:

```
Regression: [surface] — [PASS | FIXED | BLOCKED]
Command: [exact command or manual step]
Result: [1-line outcome]
```

**What counts as a representative check:**

- a single test command that exercises the core path of the earlier phase
- a manual verification step that confirms the earlier phase's key artifact still works
- do not re-run the full validation suite of every earlier phase — pick the narrowest check that
  would catch breakage

---

## Regression-Found Workflow

When a regression is detected in step 5:

**Classify the regression:**

| Type | Definition | Example |
|---|---|---|
| product breakage | previously working product behavior is broken | API endpoint returns 500, container fails to start |
| test breakage | previously passing test now fails | Vitest suite red, Playwright spec timeout |
| harness drift | process/agent/skill artifacts are inconsistent | context doc references a deleted file |
| stale command drift | a previously recorded command no longer works | pnpm script renamed, env var removed |

**Decision tree:**

1. **Fix in place** when the regression is small, the fix is obvious, and it does not widen the
   current phase scope. Fix, revalidate both the regression surface and the current phase gates,
   then continue.
2. **Revalidate only** when the regression is a false alarm (e.g., flaky test, transient infra).
   Record the finding and move on.
3. **Route as BLOCKED** when the regression is real but fixing it would materially widen scope.
   Create a follow-up artifact (backlog plan or blocker note in the phase report), mark the current
   phase `BLOCKED`, and stop.

Never paper over a regression. Always classify it and record it in the phase report, even if the
fix is trivial.

---

## Program Goal Charter

Every phase program must carry a **Program Goal Charter** as part of its umbrella orchestration
plan. Generate it automatically when building the umbrella plan, fill in only program-specific
content, and keep it tight.

Required charter structure (placeholders are program-specific only):

```text
# [PROGRAM NAME] — Program Goal Charter

North star:
- [one sentence stating the real end goal]

Definition of done:
- [the concrete capabilities an unattended agent must be able to perform when the program is complete]

What "verified" means (program level):
- [the exact bar for promoting work to VERIFIED for this program — gate surface, evidence, coverage]
- validate-contract gates must be recorded alongside phase gates and regression evidence for a phase to reach VERIFIED. A phase without a validate-contract (or documented skip reason) cannot be marked VERIFIED.

Scope tiers → phase mapping:
- Tier 1 [name] → Phases [n, n, ...]
- Tier 2 [name] → Phases [n, n, ...]
- Tier 3 [name] → Phases [n, n, ...]
- This program retires Tiers [1-N].

Explicitly out of scope (deferred tier):
- [the tier and items intentionally not addressed by this program]

Hard safety constraints (non-negotiable, per phase):
- [program-specific irreversible/destructive boundaries, e.g. "never mutate prod X"]
```

Important: execution discipline (the required 7-step inner loop `R → I → P → PVL → E → EVL → UP`,
re-research at phase entry, and honest phase status) is already governed by
`process/development-protocols/phase-programs.md`.
Do NOT re-paste that prose into the charter — the charter is program-specific intent and safety
only, not workflow rules.

A blank template plus a filled-in reference example live at
`.claude/skills/vc-generate-phase-program/references/program-goal-charter-template.md`.
