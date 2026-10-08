---
name: vc-autoresearch
description: "Loop: find gaps → fix → repeat until agents find no gaps or a metric goal is hit. Shared loop primitive for PVL, EVL, and standalone quality runs."
argument-hint: "[domain] [corpus path(s)] [verify: command] [max_iterations: N]"
trigger_keywords: autoresearch, harden spec, fix all errors, improve coverage, iterative improvement, gap loop
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# vc-autoresearch

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Reusable loop primitive. Runs: find gaps → write report → fix → check → repeat.

Used directly for spec/doc/UX hardening. Wired into PVL (plan-validate-fix loop — the fix cycle between writing a plan and approving EXECUTE) and EVL (execute-validate-fix loop — the confirmation run after EXECUTE) as the shared bookkeeping layer.

---

## When To Invoke

- **Standalone:** user says "harden this spec", "fix all lint errors", "improve test coverage"
- **PVL:** the ORCHESTRATOR invokes it when vc-validate-agent returns a first-pass CONDITIONAL/BLOCKED verdict (validate-fix loops are needed)
- **EVL:** the ORCHESTRATOR invokes it at the EVL confirmation run — unconditionally after every EXECUTE DONE, before UPDATE PROCESS

Do NOT invoke during RESEARCH or INNOVATE phases.

## Who Runs This (Loop Driver)

The **ORCHESTRATOR is the loop driver**. It executes every bookkeeping step itself:

- **Step 0 setup** — creates the task folder and the `results.tsv` tracking file.
- **Cycle counter + per-cycle iteration report** — writes one report file per loop iteration.
- **TSV row** — appends a row to `results.tsv` after each cycle.
- **Plateau/cap/regression checks** — stops the loop when no progress is made for 3 cycles (HALT_PLATEAU — no improvement after 3 tries), when the hard 10-cycle limit is hit (HALT_CAP), or when a test that was passing now fails (HALT_REGRESSION).

Subagents (vc-validate-agent, vc-tester, vc-plan-agent, vc-execute-agent) are fire-and-forget: they emit a verdict and terminate. They cannot invoke this skill on the orchestrator's behalf, cannot loop themselves, and cannot spawn each other.

If no one runs Step 0, the loop never exists and verdicts silently become "proceed" — that failure mode is exactly what this section forbids.

Per-verdict routing tables: `process/development-protocols/orchestration.md` §PVL/EVL Loop Routing.

---

## Subcommands

| Subcommand | Does | Stops when |
|---|---|---|
| `vc-autoresearch` (core) | find gaps → fix → repeat | agents find no gaps OR metric goal hit |
| `vc-autoresearch:probe` | 8 personas interrogate the corpus until saturation | no new constraints for 3 rounds |
| `vc-autoresearch:reason` | adversarial debate with blind judges until convergence | judges converge or iteration cap |
| `vc-autoresearch:evals` | analyze TSV results — trends, plateaus, recommendations | N/A (analysis only) |

Not ported (already covered by existing vc-system skills): debug → `vc-debugger`, security → `vc-security`, scenario → `vc-scenario`, predict → `vc-predict`.

---

## Parameters

| Parameter | Required | Default | Notes |
|---|---|---|---|
| `domain:` | yes | — | `spec` / `tests` / `ux` / `docs` / `plan` / `errors` |
| `corpus:` | yes | — | file glob(s) or path list to investigate |
| `verify:` | no | — | shell command that outputs a number; required for "hit the metric goal" mode |
| `target:` | no | `0` | the number `verify:` must reach (lower-is-better assumed; use `target_direction: higher` to flip) |
| `guard:` | no | — | safety shell command that must pass after every fix batch |
| `frozen_files:` | no | — | glob pattern(s); any file matching is excluded from the fix corpus and must never be modified by a fix agent |
| `max_iterations:` | no | per domain | hard cap on loop cycles |
| `severity_escalation_at:` | no | `7` | after this many iterations, stop fixing CONCERN findings (move to backlog) |
| `consecutive_all_clear:` | no | `2` | how many consecutive zero-gap iterations before SUCCESS |
| `research_agents:` | no | per domain | number of parallel research agents |
| `fix_agents:` | no | per domain | number of parallel fix agents |
| `feature:` | no | inferred | feature folder name for report output paths |
| `task_slug:` | no | auto | task folder slug; auto-generated as `autoresearch-{domain}-{YYMMDD}` |
| `auto_run:` | no | prompt | `true` = no pauses; `false` = confirm before each fix batch; under `/goal` always `true` |

---

## Canonical Domain Defaults

Full configs in `process/development-protocols/vc-autoresearch-spec.md` §Canonical domain configs.

| Domain | Research agents | Fix agents | Max iterations | Escalation at | Guard |
|---|---|---|---|---|---|
| `spec` | 2 | 3 | 15 | 7 | none |
| `tests` | 2 | 2 | 20 | — | `pnpm test` |
| `ux` | 2 | 2 | 10 | 5 | `pnpm typecheck` |
| `docs` | 1 | 2 | 8 | — | node validator script |
| `plan` | 1 | 1 | 3 | — | none |
| `errors` | 1 | 2 | 20 | — | none |
| `harness` | 2 | 2 | 10 | — | `pnpm test:runtime-harness:unit` |

_* harness full config: .claude/skills/vc-autoresearch/domains/harness.md_

---

## Loop Execution

### Step 0 — Setup

1. Parse parameters, apply domain defaults for any missing values
2. If `auto_run:` not set and NOT under `/goal`: prompt once — "Auto-run (no pauses) or confirm before each fix batch?" Choice is sticky for the full loop.
3. Create task folder: `process/features/{feature}/active/{task_slug}_{dd-mm-yy}/`
4. Initialize TSV at `{task_folder}/results.tsv` with header row and baseline row (iteration 0, gaps_found: TBD, loop_status: baseline)

### Step 1 — Research

Spawn `research_agents:` parallel agents. Each agent:
- Reads the corpus files assigned to it
- Investigates its thread list (cross-file consistency, missing cases, contradictions, undefined behaviors, etc.)
- Returns a structured gap list: **SEVERITY:** FAIL | CONCERN | OBSERVATION per finding

Collect all findings. Count: `gaps_found`, `fail_count`, `concern_count`.

Apply severity floor: if `iteration > severity_escalation_at`, discard CONCERN findings (add to backlog section of report — do not fix).

### Step 2 — Convergence check

**"Until agents find no gaps"** (no `verify:` param):
- If all agents returned zero findings above the severity floor: increment `consecutive_all_clear` counter
- If counter >= `consecutive_all_clear:` → **SUCCESS**
- Else: reset counter, continue to Step 3

**"Hit the metric goal"** (`verify:` param set):
- Run verify command, parse numeric output
- If output reaches `target:` → **SUCCESS**
- Else: continue to Step 3

### Step 3 — Termination check (non-success)

Check in priority order:

1. **PLATEAU** — `gaps_found` unchanged or increased for 3 consecutive iterations → **HALT_PLATEAU**
2. **CAP** — `iteration >= max_iterations` → **HALT_CAP**
3. **REGRESSION** — more than 2 new gaps in areas that were gap-free last iteration → **HALT_REGRESSION** (always pauses for user, even under `/goal`)

If none triggered: continue to Step 4.

### Step 4 — Write iteration report

Write a NEW per-iteration report file:
`{task_folder}/{task_slug}-iteration-{NNN}_REPORT_{dd-mm-yy}.md` — `{NNN}` is the
zero-padded 3-digit iteration number (`001`, `002`, … `042`), so files sort correctly and
every iteration is uniquely named no matter how many run.

ONE FILE PER ITERATION — hard rule. NEVER append iterations to a single rolling file
(no `ITERATION-NOTES.md`, no shared `{task_slug}_REPORT_*.md` updated in place). The
rolling cross-iteration view is `results.tsv`, nothing else.

Append a row to `{task_folder}/results.tsv`.

If `auto_run: false`: surface gap summary, wait for user confirmation before fixing.

### Step 5 — Fix

Spawn `fix_agents:` parallel agents. Each agent:
- Receives its assigned gap IDs and file targets
- Applies fixes
- Reports: APPLIED (fixed in this iteration) or BACKLOG (deferred, with reason)

### Step 6 — Safety check

If `guard:` is set: run guard command.
- **Passes** → loop back to Step 1
- **Fails** → revert fix batch, log regression flag, retry this iteration. If regression budget exceeded (> 2 flags in one iteration) → HALT_REGRESSION

### Step 7 — Loop

Increment iteration counter. Go to Step 1.

---

## Termination Output

On any terminal state, write final iteration report with `loop_status:` set, then emit:

```
AUTORESEARCH COMPLETE
Domain:          {domain}
Iterations run:  N
Terminal state:  SUCCESS | HALT_PLATEAU (no progress after 3 cycles) | HALT_CAP (10-cycle hard limit) | HALT_REGRESSION (passing test now fails) | HALT_SEVERITY (critical gap found)
Gaps remaining:  N (FAIL: N, CONCERN: N)
Files updated:   [list]
Report:          {task_folder}/{task_slug}-iteration-{NNN}_REPORT_{dd-mm-yy}.md  (final iteration)
TSV:             {task_folder}/results.tsv
```

---

## PVL Wiring

When vc-autoresearch is the bookkeeper for a PVL (plan-validate-fix loop):

- `domain: plan`, `corpus:` = the plan .md file
- **Research step** = vc-validate-agent runs V1–V3 gates; autoresearch does NOT run these itself. The validate step itself fans out in parallel via `vc-validate-findings` (Layer 1 dimension agents + Layer 2 feasibility agents) — that parallelism is owned by vc-validate-agent.
- **Gap signal** = FAIL / CONDITIONAL / CONCERN from validate-agent
- **Fix step (parallel)** = when the gap set spans independent plan sections, the orchestrator spawns **multiple parallel plan-fix agents**, one per independent gap group, partitioned so no two agents edit the same plan region. `fix_agents:` defaults to the count of independent gap groups (cap to the `plan`-domain default unless raised). Each fixer is scoped to its assigned gap IDs only. When gaps are interdependent or touch one section, fall back to a single plan-fix agent.
- **Convergence** = validate-agent returns PASS → SUCCESS
- **Cap** = 10 plan-validate-fix loops
- **Per-cycle report** = every PVL cycle writes its own report file
  `{task_folder}/{plan-slug}-pvl-iteration-{NNN}_REPORT_{dd-mm-yy}.md` (zero-padded `{NNN}`)
  capturing: gaps found (with severity), fixes applied vs backlogged, and the validate verdict
  for that cycle. Never a single rolling notes file.

**Boundary:**
- vc-autoresearch owns: iteration counter, plateau detection, per-cycle iteration report, regression flag, parallel-fix partitioning (which gap groups go to which fixer) — **all executed by the ORCHESTRATOR at each cycle boundary; no agent runs these implicitly**
- vc-validate-agent owns: V1–V7 gate sequence + its own Layer-1/Layer-2 fan-out, SUPPLEMENT REQUEST format, validate-contract write, known-gap exclusion — it emits its verdict and terminates; the orchestrator re-spawns it from V1 after each `SUPPLEMENT_APPLIED`

---

## EVL Wiring

When vc-autoresearch is the bookkeeper for an EVL (execute-validate-fix loop):

- `domain: tests`, `verify:` = validate-contract gate commands (read from contract — never invented)
- **Research step** = a SPAWNED vc-tester agent runs the validate-contract fully-automated gate commands (vc-tester may run independent gate groups in parallel). The orchestrator NEVER runs gate commands in its own shell — a gate result not produced by a vc-tester spawn does not count as an EVL confirmation, even if green.
- **Gap signal** = failing gate (non-zero exit)
- **Fix step (parallel)** = when multiple independent gates fail across non-overlapping file groups, the orchestrator spawns **multiple parallel execute-fix agents** (vc-execute-agent in supplement mode), one per failing gate / file group, partitioned so no two agents edit the same file. Each fixer is scoped to exactly its failing gate — no scope expansion. When failing gates share files or a single root cause, fall back to a single execute-fix agent. The fix is ALWAYS a vc-execute-agent spawn — the orchestrator never edits source files itself, no matter how small the fix.
- **Convergence** = all validate-contract fully-automated gates pass → SUCCESS
- **Cap** = 10 execute-validate-fix loops
- **Per-cycle report** = every EVL cycle writes its own report file
  `{task_folder}/{plan-slug}-evl-iteration-{NNN}_REPORT_{dd-mm-yy}.md` (zero-padded `{NNN}`)
  capturing: which gates ran, which failed (with trimmed failure output), fixes applied, and
  the re-run result for that cycle. Never a single rolling notes file.

**Boundary:**
- vc-autoresearch owns: iteration counter, plateau detection, TSV log, per-cycle iteration report, HANDOFF SUMMARY trigger, parallel-fix partitioning (which failing gate goes to which fixer) — **all executed by the ORCHESTRATOR at each cycle boundary; no agent runs these implicitly**
- vc-tester owns: which gate commands to run, HANDOFF SUMMARY format, agent-probe re-invocation — its confirmation run is UNCONDITIONAL after every EXECUTE DONE (execute-agent's internal iterate-until-green loop never substitutes for it); it reports failing gates and terminates; the orchestrator runs the fix cycle and re-spawns it

---

## Iteration Report Format

See `process/development-protocols/vc-autoresearch-spec.md` §Iteration report for full frontmatter schema and body sections.

Gap entry format:
```
### GAP-I{N}-{ID} — {short title}
- **SEVERITY:** FAIL | CONCERN | OBSERVATION
- **LOCATION:** {file} §{section}
- **GAP:** {what is missing or wrong}
- **RESOLUTION:** {what was changed}
- **STATUS:** APPLIED | BACKLOG
```

Each iteration report is written inside the active task folder as
`{slug}-iteration-{NNN}_REPORT_{dd-mm-yy}.md` with `{NNN}` zero-padded to 3 digits
(task-folder artefact colocation — never a sibling `reports/` dir). One file per
iteration; PVL/EVL cycles use the `{plan-slug}-pvl-iteration-{NNN}` /
`{plan-slug}-evl-iteration-{NNN}` slug variants.

---

## TSV Log Format

Header row:
```
iteration	timestamp	gaps_found	fail_count	concern_count	applied	saturation_status	loop_status	notes
```

- Baseline row = iteration 0, before any fixes, `loop_status: baseline`
- One row appended after each iteration's fix batch completes
- `saturation_status`: ACTIVE | PLATEAU | SATURATED
- `loop_status`: CONTINUE | HALTED_SUCCESS | HALTED_PLATEAU | HALTED_CAP | HALTED_REGRESSION

Run `vc-autoresearch:evals {task_folder}/results.tsv` to analyze trends and get a plateau/recommendation report.

---

## Hook Notes

- `iteration-context.cjs` — injects last 3 TSV rows at UserPromptSubmit; must scan `process/features/*/active/*/results.tsv` (not project-root `autoresearch/` paths)
- `session-init.cjs` — active plan summary at SessionStart; adapted to scan `process/general-plans/active/` and `process/features/*/active/`
- `stop-notify.cjs` — terminal notification at SessionEnd; no changes needed
- Do NOT register `scout-block.cjs` or `privacy-block.cjs` from the autoresearch reference repo — vc-system settings.json has superior versions; registering them would cause conflicts
