# Review Situation Workflow

`vc-review-situation` is a read-only orientation helper for this repo.

## What It Scans

1. Git truth
   - current branch or detached HEAD
   - dirty tree summary
   - ahead/behind state when available
   - local and remote branch refs from local metadata
   - sampled recent commits per ranked branch
   - local worktrees
2. project plan truth
   - `process/general-plans/active/` (plans inside `{slug}_{date}/` task subfolders — scan one level deep)
   - `process/features/*/active/` (same depth)
   - compatibility shapes discovered from filesystem and ranked tracked refs
3. Optional selected-plan hints
   - explicit CLI path
   - session-state `activePlan` hint
   - exactly one active primary plan candidate

## Plan Shapes

Treat these as compatibility shapes:

- `*_PLAN_*.md`
- `PLAN.md`
- `plan.md`
- `phase-*.md`

Primary plan candidates are:

- `*_PLAN_*.md`
- `PLAN.md`
- `plan.md`

Phase files can still show that work is in flight, but they should not become the selected-plan hint unless explicitly passed in.

Tracked-ref scans may surface active plans from other local or remote refs for awareness only. They must never become selected-plan authority on their own.

## Hint Priority

Selected-plan resolution order:

1. explicit `--selected-plan`
2. local session-state `activePlan` when it resolves to a real file
3. exactly one primary active plan candidate
4. otherwise no selected plan

All non-explicit hints must be labeled advisory.

## Output Policy

- default: stdout text summary
- optional: `--json`
- branch/ref scans are stale-by-default unless `--fetch` is explicit
- no saved report by default

If the user later wants a saved report, that should be a separate explicit follow-up decision.

Preferred text report structure:

1. Current State
2. Recent Work
3. In-Flight Plans
4. Next Steps
5. Warnings

## Non-Goals

- no execute authority
- no plan mutation
- no status frontmatter edits
- no auto-resume
- no automatic remote fetch
- no upstream `plans/**` assumptions

## Fallback Behavior

If the scanner errors, report that directly and fall back to:

```bash
git status --short --branch
git worktree list --porcelain
git for-each-ref --format='%(refname:short) %(committerdate:iso8601) %(objectname:short) %(subject)' refs/heads refs/remotes
find process/general-plans/active process/features -path '*/active/*' -type f | sort
```

Do not present the fallback summary as if the full scanner succeeded.

## Validation

Cross-check `vc-review-situation` output with:

```bash
git status --short --branch
git worktree list --porcelain
git for-each-ref --format='%(refname:short) %(committerdate:iso8601) %(objectname:short) %(subject)' refs/heads refs/remotes
find process/general-plans/active process/features -path '*/active/*' -type f | sort
```

Recommended regression tests:

- explicit vs inferred selected-plan hinting
- multiple primary local plans suppressing inference
- tracked-ref branch visibility without `--fetch`
- bounded tracked plan scanning via `--max-plan-refs`
- detached HEAD and fetch-failure warning paths
