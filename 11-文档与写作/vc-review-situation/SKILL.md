---
name: vc-review-situation
description: "Use when you need a read-only situation review and handoff summary of current branch state, local/remote refs, worktrees, active project plans, selected-plan hints, and suggested next checks."
license: MIT
argument-hint: "[--json] [--fetch] [--selected-plan <path>] [--cwd <path>]"
trigger_keywords: what's in flight, handoff, worktree status, active plans, next steps
layer: helper
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# Review Situation

> Output style: lead with the bottom line, bullets over prose, one-line TL;DR — `process/development-protocols/communication-standards.md`.

Summarize the current repo state for handoff and resume work.

This is a helper skill only.

- Do use it for read-only branch, worktree, and active-plan summaries.
- Do use it to surface likely selected-plan context when it can be proven or safely hinted.
- Do not use it to choose a plan authoritatively.
- Do not use it to approve execution, resume execution, or mutate repo/process state.

## Mode Selection

`vc-review-situation` operates in two modes. Choose based on the trigger signals below.

### Simple Mode (default)

Run `review-situation-scan.cjs`, return the scan summary.

**Use when:**
- User asks "what's in flight", "what's next", "give me a handoff summary"
- Quick orientation needed at the start of a session
- Orchestrator needs a branch/plan status check before routing

**Output:** Current State, Recent Work, In-Flight Plans, Next Steps, Warnings (standard scan sections).

### Deep Mode

Run the scan **plus** read umbrella plan, latest phase report, and all active-plan handoff sections. Synthesize into a full handoff briefing sufficient for an agent to resume without follow-up questions.

**Trigger conditions (any one):**
- User asks for a full program review: "what are we building", "summarize all active plans", "where are we in the program"
- Session is resuming after a long break or context compaction occurred
- Orchestrator needs a thorough handoff before routing to a phase agent
- Caller explicitly requests deep mode
- **Program Review Mode is active** — Program Review Mode is Deep Mode

**Deep mode steps:**
1. Run `review-situation-scan.cjs` as normal (get branch state + active plan list)
2. If a phase program is active: read the umbrella plan in full — especially `## Current Execution State` and `## Phase Ordering`
3. Read the most recent phase report in full (if it exists)
4. If 3+ phases are completed: read the Forward Preview sections from earlier phase reports
5. For each active plan: read the `## Resume and Execution Handoff` section
6. Synthesize: "You are at Phase N of M. Phase N-1 completed with these outputs. The next phase needs to know: X. These are the open gaps from prior phases: Y."

**Output:** Full handoff briefing — program position, prior-phase outputs, open gaps, next-phase inputs, and any hard-stop conditions.

---

## Core Contract

`vc-review-situation` is advisory.

- Evidence comes from git, worktree metadata, and `process/*` plan inventory.
- Selected-plan awareness is a hint, not a command.
- Next-step recommendations are suggestions, not workflow gates.

If a user needs execution, the repo still requires explicit plan selection and `ENTER EXECUTE MODE`.

## Invocation

Run the local scanner:

```bash
node .claude/skills/vc-review-situation/scripts/review-situation-scan.cjs --json
```

Useful flags:

```bash
node .claude/skills/vc-review-situation/scripts/review-situation-scan.cjs
node .claude/skills/vc-review-situation/scripts/review-situation-scan.cjs --json --max-branches 8 --plan-limit 6
node .claude/skills/vc-review-situation/scripts/review-situation-scan.cjs --selected-plan process/general-plans/active/example_27-05-26/example_PLAN_27-05-26.md
node .claude/skills/vc-review-situation/scripts/review-situation-scan.cjs --since "14 days ago"
node .claude/skills/vc-review-situation/scripts/review-situation-scan.cjs --fetch
```

## Input Sources

The scanner reads from:

- `git status --short --branch`
- `git worktree list --porcelain`
- local and remote branch refs plus sampled recent commits
- `process/general-plans/active/` (plans inside `{slug}_{date}/` task subfolders — scan one level deep)
- `process/features/*/active/` (same depth)
- optional session-state hints if a local session id is present

It does not scan upstream `plans/**`, and it never treats a selected-plan hint as execute authority.

## Output Shape

The default output is a human-readable report with these sections:

- Current State
- Recent Work
- In-Flight Plans
- Next Steps
- Warnings

`--json` returns the same information as structured data.

If the scanner fails, say that explicitly and fall back to the minimal read-only commands:

```bash
git status --short --branch
git worktree list --porcelain
git for-each-ref --format='%(refname:short) %(committerdate:iso8601) %(objectname:short) %(subject)' refs/heads refs/remotes
find process/general-plans/active process/features -path '*/active/*' -type f | sort
```

## Safety Rules

1. Read-only only. No branch switching, plan edits, or fetch unless `--fetch` is explicit.
2. Treat selected-plan inference as tentative unless it came from an explicit `--selected-plan` argument.
3. Remote branch data is stale-by-default unless `--fetch` is explicit.
4. Use this for session handoff and quick repo orientation, not for workflow control.
5. If the scan cannot prove something, emit a warning instead of guessing.

Good trigger phrases:

- `what's in flight`
- `give me a handoff summary`
- `what active plans do we have`
- `show branch and worktree status`
- `what should I look at next`

Load `references/review-situation-workflow.md` when you need the project's decision tree or hint-priority rules.

## Program Review Mode

> **Program Review Mode = Deep Mode.** All trigger conditions and synthesis steps from the [Mode Selection — Deep Mode](#deep-mode) section apply here. The output format below is the required presentation layer for Deep Mode when the orchestrator or user needs a full visual summary before execution.

**Trigger:** User asks for plan review, program summary, "what are we building", "summarize all active plans"

**Goal:** Read ALL active plans for a program and produce a comprehensive visual summary for user feedback before execution.

### Required output elements

- Macro goal + specific phase goals
- All important decisions made (DECISION / WHY / REJECTED format)
- Expected behaviors being hardened (before/after tables)
- Phase sequence with dependencies (ASCII box diagram using ─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼ ► → characters — NEVER mermaid)
- Per-phase summary: purpose, key items, exit gate, dependencies, risks
- Validate contract status
- Gaps / out-of-scope items
- End with: "Anything to modify before execution?"

### Output format rules

- Use ASCII box diagrams (─ │ ┌ ┐ └ ┘ ├ ┤ ┬ ┴ ┼ ► → characters) for phase sequence diagrams and dependency flows — NEVER mermaid (mermaid does not render in terminal)
- Use ASCII/markdown tables for before/after behavior comparisons
- Use numbered decision logs (DECISION / WHY / REJECTED)
- Never omit important details — output is for user feedback before execution
- End with: "Anything to modify before execution?"

### Process steps

1. Run standard vc-review-situation check (branch, worktree, active plans list)
2. Identify target program (user-specified feature folder or all active plans)
3. Read ALL plan files in program fully (umbrella + all phase plans)
4. Extract: macro goal, per-phase goals, decisions, behavior changes, validate contract status
5. Produce visual summary using ASCII diagrams + markdown tables
6. Prompt for feedback

## Artifact Review Mode

Read and compare process artifacts (plan file + validate-contract, or plan vs git diff) and produce a concise inline summary.

- **Plan + validate-contract comparison:** Read the plan file and its embedded validate-contract section; render a side-by-side summary of what was planned vs what the contract locked in, flagging gaps or conflicts.
- **Plan vs diff view:** Read the specified plan file and run `git diff [ref]` to show what changed; map changed files against the plan's blast-radius, highlighting covered vs uncovered areas.
- **File viewer:** Read any process artifact (plan, report, validate-contract) and emit a clean inline summary with key decisions, open items, and exit gates — useful before passing context to an execute subagent.
- **Output format:** ASCII tables and prose only — no Mermaid, no HTML, no server. Output is terminal-friendly and safe to paste into a subagent prompt.
- **Trigger phrases:** `review the plan`, `show plan vs contract`, `what did we plan vs what changed`, `summarize this plan file`, `diff plan against code`
