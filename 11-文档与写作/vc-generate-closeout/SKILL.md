---
name: vc-generate-closeout
description: "Generate the post-EXECUTE closeout packet for a plan or phase. Includes archive-readiness classification, drift signal scoring, commit checkpoint recommendation, and move-on next-state recommendation."
argument-hint: "[selected plan path or phase name]"
trigger_keywords: closeout, phase closeout, archive readiness, drift scoring, move-on
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-generate-closeout

> Output style: closeout packet leads with the verdict/recommendation, tables for drift/readiness, one-line TL;DR — `process/development-protocols/communication-standards.md`.

Generate the post-EXECUTE closeout packet for a completed plan or phase. Produces a structured summary with archive-readiness classification, drift signal scoring, commit checkpoint recommendation, and the single best next valid state.

## When To Invoke

Invoke this skill at the end of:

- Any non-trivial EXECUTE completion (the execute-agent's own closeout block)
- ENTER UPDATE PROCESS MODE flows before archiving a plan
- fast-mode-agent session end, after implementation and verification are complete
- Phase program closeout between phases (after validate + regression checkpoint, before inter-phase UPDATE PROCESS)

Do not invoke for trivial single-file fixes where the plan file is not involved.

## Mode Selection

Choose one mode before generating the closeout packet.

### Simple Mode (default)

Use when:
- The execute session just ended and all context is fresh in the conversation.
- The plan was a single-phase, straightforward implementation.
- No context compaction occurred during the session.

Behavior: write the 8-item closeout packet directly from the plan file and conversation context. No additional git or file scanning is required.

### Deep Mode

Trigger conditions — use Deep Mode if **any one** of the following applies:

- The session was resumed after a context compaction (conversation context may be incomplete).
- The execute session was long — many sub-steps, multiple checklist sections, or multiple commits.
- The caller explicitly requests deep mode.
- The phase is part of a multi-phase program and accurate phase-completion status is required for the umbrella plan.

Behavior: gather evidence before writing the closeout packet. Run these steps in order:

1. `git diff HEAD~1 --name-only` — confirm which files were actually changed. Cross-reference against the plan's blast radius.
2. `git log --oneline -5` — confirm commit messages match what was planned. Flag any commits that touch files outside the plan's scope.
3. Read the phase report if one was already written — confirm its claims match actual execution output.
4. Read the validate-contract test gates — confirm each gate's final status: green, known-gap, or skipped. Do not rely on memory for gate outcomes.
5. Read the plan Implementation Checklist — for every checked item, confirm a matching file appears in the git diff. Flag any checked item with no corresponding change.

After gathering evidence, write the closeout packet with explicit source citations for every material claim.

**Quality difference between modes:**

- Simple: "Section 2 completed — billing router updated" (from memory)
- Deep: "Section 2 completed — `git diff HEAD~1` confirms `packages/api/src/router/billing.ts` modified (+47/-12 lines); test gate `pnpm test:billing` confirmed green in execute log"

When in doubt, prefer Deep Mode. A false-confident Simple closeout is worse than a slightly slower Deep one.

## Closeout Packet Schema

Every closeout packet must include these 9 items. Present them in order.

1. **Selected plan path**
   - The exact file path of the plan being closed out (e.g. `process/features/foo/active/foo-phase-01_PLAN_03-06-26.md`). Never leave this implicit.

2. **Closeout classification** (one of three states — see §Closeout Classification States)

3. **What was finished**
   - A concrete list of what was actually implemented or changed. Not a restatement of the plan checklist — what was done in practice.

4. **What was verified vs still unverified**
   - What tests or evidence exist that confirm the work.
   - What still requires manual verification, user confirmation, or future test coverage.

4b. **Validate-contract compliance**
   - Was VALIDATE run for this plan?
   - Is a `## Validate Contract` section present in the plan file?
   - If VALIDATE was skipped, state the documented skip reason.
   - A plan cannot be classified `Ready for UPDATE PROCESS archival` without a present validate-contract or a documented skip reason.

5. **Cleanup done vs still needed**
   - What context docs, reports, or process artifacts were already updated.
   - What remains: open TODOs, uncommitted changes, missing reports, stale references, or plan debt.

6. **Single best next valid state** (one of the allowed states from §Move-On Semantics)
   - Name the exact next action or plan path. Never end with a generic "move to next task."

7. **Commit-checkpoint recommendation** (see §Commit Checkpoint Classification)
   - Whether to invoke `vc-git-manager` before UPDATE PROCESS, or whether the remaining changes are process-only and the commit belongs after UPDATE PROCESS.

8. **Regression status** (phase programs only)
   - Which previously verified surfaces were checked for regression against this phase's blast radius.
   - Whether all passed, or whether fixes were applied before re-verification.
   - If regression checking was skipped (e.g., first phase with no prior verified surfaces), state why explicitly.

9. **SPEC achievement**
   - For each acceptance criterion in the locked `*_SPEC_*.md`, score **met** or **unmet**.
   - Each unmet criterion → a backlog NOTE (the SPEC is frozen; gaps go to backlog only).
   - If there is no SPEC for this plan (e.g. trivial or phase-program inner loop governed by the umbrella SPEC), state that explicitly.

## Closeout Classification States

Exactly three states are allowed. Choose one and state it verbatim.

- **Ready for UPDATE PROCESS archival**
  - The selected plan path still matches the implemented work.
  - Required verification evidence exists.
  - No material deviations remain unresolved.
  - The user has confirmed or approved cleanup.
  - validate-contract is present in the plan file, or VALIDATE was explicitly skipped with a documented reason.

- **Keep in active/testing**
  - Implementation is substantially complete.
  - But testing, manual verification, or explicit user confirmation is still pending.
  - Do not archive until those are resolved.

- **Needs PLAN/UPDATE PROCESS reconciliation**
  - Material deviations from the selected plan were required during execution.
  - Context or process updates are needed before the plan can be archived.
  - The work should route through UPDATE PROCESS or back to PLAN first.

## Drift Signal Scoring

After building the closeout packet, score the UPDATE PROCESS urgency by counting signals.

Signal sources (5 sources, max score 6):

- (a) Files touched during the EXECUTE phase: **+1** if ≥1 file, **+1 more** if ≥10 files (max 2 from this source)
- (b1) Any `.claude/`, `.codex/`, or agent harness file (agent `.md`/`.toml`, SKILL.md, settings, hooks) changed: **+1**
- (b2) Any `README.md`, `AGENTS.md`, `CLAUDE.md`, or `process/development-protocols/` file changed: **+1**
- (c) Session involved 3 or more memory-worthy observations (new patterns discovered, deviations documented, architectural decisions made): **+1**
- (d) Feature-folder structural change (new `{slug}_{date}/` task folder created, backlog NOTE written, or task folder archived/moved): **+1**
- (e) Validate-contract deviation (execution diverged from the validate-contract or the plan's declared blast radius): **+1**

Thresholds and required wording:

The exact phrase for each band must appear word-for-word in the closeout output (do not rephrase — these are machine-matched strings):

- **LOW (0–1 signals):** (few files changed, nothing critical) include `"UPDATE PROCESS available if you want."`
- **MEDIUM (2–3 signals):** (notable scope — worth capturing learnings) include `"Recommend UPDATE PROCESS -- significant changes detected."`
- **HIGH (4+ signals):** (harness, agents, or protocol docs were edited — durable capture is urgent) include `"Strongly recommend UPDATE PROCESS -- harness/protocol files touched."`

Always include the exact threshold phrase verbatim in the closeout output. Do not summarize or rephrase the wording.

## Move-On Semantics

"Move on" does not include an automatic transition into UPDATE PROCESS. It still must not silently archive work or widen scope.

The orchestrator should:

1. Finish the closeout packet.
2. Recommend the next valid state explicitly.
3. Name the exact next plan or phase when one clear successor exists.
4. Avoid reopening broad research when the next step is already known from the current program structure.

Allowed next-state examples (use these exact forms when applicable):

- If the selected plan is verified and the next phase is explicit:
  `ENTER UPDATE PROCESS MODE, then continue with process/features/.../next-phase_PLAN_...md`
- If the selected plan is verified and implementation changes are still uncommitted:
  `Invoke vc-git-manager for a logical execution commit, then ENTER UPDATE PROCESS MODE for plan/context reconciliation`
- If the selected plan is code-complete but still testing:
  `Keep the plan active and continue validation on the same selected plan`
- If the selected plan exposed follow-up work outside its boundary:
  `ENTER UPDATE PROCESS MODE to capture the split and route the follow-up into its own plan`

## Archive-Readiness Semantics

Do not treat every successful code change as immediately archive-ready.

Use these three states (see also §Closeout Classification States for the exact criteria):

- **Ready to archive**
  - The selected plan path still matches the implemented work.
  - Required verification evidence exists.
  - No material deviations remain unresolved.
  - The user has confirmed or approved cleanup.
  - validate-contract is present in the plan file, or VALIDATE was explicitly skipped with a documented reason.

- **Keep in active / testing**
  - Implementation is substantially complete but testing, manual verification, or explicit user confirmation is still pending.

- **Needs reconciliation before archival**
  - Material deviations from the selected plan were required.
  - Context/process updates are needed before the plan can be archived.
  - The work should route through UPDATE PROCESS or back to PLAN first.

For non-trivial work, prefer routing archive decisions through UPDATE PROCESS so context updates, lessons learned, and selected-plan archival happen together.

## Phase Program Closeout Shape

After each executed phase in a phase program, produce a short closeout packet with these 7 items:

1. **Selected phase plan path** — the exact file path of the completed phase plan.
2. **Phase status** — one of:
   - `✅ VERIFIED`
   - `Keep in active/testing`
   - `🚧 BLOCKED`
   - `Needs reconciliation`
3. **What green actually proves** — a precise statement of what the passing gates confirm, and what they do not cover.
4. **Regression status** — surfaces checked, results, any fixes applied. Format each entry as:
   ```
   Regression: [surface] — [PASS | FIXED | BLOCKED]
   Command: [exact command or manual step]
   Result: [1-line outcome]
   ```
5. **What remains outside this phase** — scope explicitly deferred, planned follow-up work, or items that would require a new phase or feature folder.
6. **Whether UPDATE PROCESS is the next required step** — inter-phase UPDATE PROCESS is mandatory between phases, not optional. Phase outputs must survive compaction.
7. **The exact next phase or follow-up plan if known** — name the path explicitly. If the next phase is already in the umbrella plan, name it. Do not make the user infer it from folder state.

This is how a phase program moves on without losing durable state or requiring the user to reconstruct context from a long transcript.

## Commit Checkpoint Classification

For validated phase work, classify the commit checkpoint explicitly using one of two forms:

- **Execution commit recommended before UPDATE PROCESS**
  - Implementation or test changes from the selected phase are well-tested and ready for a logical code/test commit.
  - Later UPDATE PROCESS edits are expected to touch `process/`, `.claude/`, `.codex/`, or `AGENTS.md` separately.
  - Recommend invoking `vc-git-manager` before routing to UPDATE PROCESS.

- **Process commit belongs after UPDATE PROCESS**
  - The remaining changes are primarily plan, report, context, or harness-governance artifacts.
  - Splitting execution and process commits will keep the history easier to review and resume.
  - Do not invoke `vc-git-manager` before UPDATE PROCESS; route to UPDATE PROCESS first, then commit.

When both execution and process changes are present, always recommend the execution commit first, then UPDATE PROCESS, then the process commit.

## Output Format

Present the closeout packet as a structured Markdown block with each numbered item as a heading or bold label. End with the drift signal score and required threshold phrase, and the single best next valid state as a clear final recommendation.

Per **task-folder artefact colocation**, when the closeout packet is persisted as a file, write it INTO the plan's own task folder (`process/features/{feature}/active/{slug}_{date}/` or `process/general-plans/active/{slug}_{date}/`) as `{slug}_REPORT_{date}.md` — never into a sibling `reports/` dir or any ad-hoc location. On completion the whole folder moves as a unit.

Example shape:

```
**Closeout Packet**

1. Selected plan path: `process/features/foo/active/foo_03-06-26/foo_PLAN_03-06-26.md`
2. Closeout classification: Ready for UPDATE PROCESS archival
3. What was finished: [...]
4. Verified: [...] | Unverified: [...]
4b. Validate-contract: present (inline in plan, PASS)
5. Cleanup done: [...] | Still needed: [...]
6. Next valid state: ENTER UPDATE PROCESS MODE
7. Commit checkpoint: Execution commit recommended before UPDATE PROCESS — invoke vc-git-manager first
8. Regression status: (phase programs only) [...]

Drift score: HIGH (3 signals: 12 files touched, .claude/ skill added, 3 memory-worthy observations)
Strongly recommend UPDATE PROCESS -- harness/protocol files touched.
```
