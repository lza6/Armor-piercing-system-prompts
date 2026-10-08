---
name: check
description: Invoke after any implementation task completes or before merging. Reviews the diff, auto-fixes safe issues, runs specialist security and architecture reviewers on large diffs. Not for exploring ideas or debugging.
metadata:
  version: "3.8.0"
---

# Check: Review Before You Ship

Prefix your first line with 🥷 inline, not as its own paragraph.


Read the diff, find the problems, fix what can be fixed safely, ask about the rest. Done means verification ran in this session and passed.

## Get the Diff

Get the full diff between the current branch and the base branch. If unclear, ask. If already on the base branch, ask which commits to review.

## Scope

Measure the diff and classify depth:

| Depth | Criteria | Reviewers |
|-------|----------|-----------|
| **Quick** | Under 100 lines, 1-5 files | Base review only |
| **Standard** | 100-500 lines, or 6-10 files | Base + conditional specialists |
| **Deep** | 500+ lines, 10+ files, or touches auth/payments/data mutation | Base + all specialists + adversarial pass |

State the depth before proceeding.

## Did We Build What Was Asked?

Before reading code, check scope drift: do the diff and the stated goal match? Label: **on target** / **drift** / **incomplete**.

Drift signals (any one is enough to label drift):
- A changed file has no connection to the stated goal
- The diff includes pure refactoring (renames, formatting, restructuring) when the goal was a bug fix or feature
- A new dependency appears that the goal did not mention
- Code unrelated to the goal was deleted or commented out
- A new abstraction or helper was introduced that is not required by the goal

## Hard Stops (fix before merging)

- **Destructive auto-execution**: any task marked "safe" or "auto-run" that modifies user-visible state (history files, config, preferences, installed software) must require explicit confirmation.
- **Release artifacts missing**: verify every artifact listed in the release template exists as a local file and has been uploaded before declaring done.
- **Unknown identifiers in diff**: any function, variable, or type introduced in the diff that does not exist in the codebase is a hard stop. Grep before writing or approving any reference: `grep -r "name" .` -- no results outside the diff = does not exist.
- **Injection and validation**: SQL, command, path injection at system entry points. Credentials hardcoded or logged.
- **Dependency changes**: unexpected additions or version bumps in package.json, Cargo.toml, go.mod, requirements.txt. Flag any new dependency not obviously required by the diff.

## Specialist Review (Standard and Deep only)

Load `references/persona-catalog.md` to determine which specialists activate. Launch all activated specialists in parallel via the environment's agent or sub-agent facility when available, passing the full diff. If no parallel reviewer facility exists, run the specialist passes sequentially in the same session.

Merge findings: when two specialists flag the same code location, keep the higher severity and note cross-reviewer agreement. Findings on different code locations are never duplicates even if they share a theme.

## Autofix Routing

| Class | Definition | Action |
|-------|------------|--------|
| `safe_auto` | Unambiguous, risk-free: typos, missing imports, style inconsistencies | Apply immediately |
| `gated_auto` | Likely correct but changes behavior: null checks, error handling additions | Batch into one user confirmation block |
| `manual` | Requires judgment: architecture, behavior changes, security tradeoffs | Present in sign-off |
| `advisory` | Informational only | Note in sign-off |

Apply all `safe_auto` fixes first. Batch all `gated_auto` into one confirmation block. Never ask separately about each one.

## Adversarial Pass (Deep only)

"If I were trying to break this system through this specific diff, what would I exploit?" Four angles (see `references/persona-catalog.md`): assumption violation, composition failures, cascade construction, abuse cases. Suppress findings below 0.60 confidence.

## GitHub Operations

Use `gh` CLI for all GitHub interactions, not MCP or raw API. Confirm CI passes before merging.

## Verification

Run `bash "${CLAUDE_SKILL_DIR:-$HOME/.agents/skills/check}/scripts/run-tests.sh"` or the project's known verification command. Paste the full output.

If the script exits non-zero or prints `(no test command detected)`: halt. Do not claim done. Ask the user for the verification command before proceeding. If the user also cannot provide one, document this explicitly in the sign-off as `verification: none -- no command available` and flag it as a structural gap, not a pass.

For bug fixes: a regression test that fails on the old code must exist before the fix is done.

If any of these phrases appear in your reasoning, stop and run verification first:
- "should work now" / "probably correct" / "seems to be working" / "trivial change"

## Gotchas

| What happened | Rule |
|---------------|------|
| Commented on #249 when discussing #255 | Run `gh issue view N` to confirm title before acting |
| PR comment sounded like a report | 1-2 sentences, natural, like a colleague. Not structured, not AI-sounding. |
| PR comment used bullet points | Write as paragraphs; thank the contributor first |
| article.en.md inside _posts_en/ doubled the suffix | Check naming convention of existing files in the target directory first |
| Released with no artifacts attached | Verify every artifact exists locally and is uploaded |
| "Trivial one-line fix" broke things | If the urge to skip verification arises, run tests anyway |
| Deployed without env vars set | Run `vercel env ls` before deploying; diff against local keys |
| Push failed from auth mismatch | Run `git remote -v` before the first push in a new project |

## Sign-off

```
files changed:    N (+X -Y)
scope:            on target / drift: [what]
review depth:     quick / standard / deep
hard stops:       N found, N fixed, N deferred
specialists:      [security, architecture] or none
new tests:        N
verification:     [command] -> pass / fail
```
