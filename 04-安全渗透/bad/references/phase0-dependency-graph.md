# Phase 0: Dependency Graph — Detailed Reference

Read this file during Phase 0 steps 4 and 6.

---

## Step 4: Update GitHub PR/Issue Status and Reconcile sprint-status.yaml

GitHub PR merge status is the **authoritative source of truth** for whether a story is `done`. This step always runs, even on resume.

### PR Status Lookup

Search by branch name:
```bash
gh pr list --search "story-{number}" --state all --json number,title,state,mergedAt
```

### GitHub Issue Number Lookup

Resolve in this order:
1. Check the epic file and `sprint-status.yaml` for an explicit issue reference.
2. If not found, search by the BMad issue title prefix `"Story {number}:"`:
   ```bash
   gh issue list --search "Story 3.1:" --json number,title,state
   ```
   Pick the best match by comparing titles.
3. If still not found, leave the Issue column blank.

### Reconcile sprint-status.yaml from PR Status

After updating PR statuses, sync `_bmad-output/implementation-artifacts/sprint-status.yaml` at the **repo root** to match. For every story whose PR status is now `merged`, set its sprint-status entry to `done` (regardless of what the file currently says).

This repair step handles cases where sprint-status.yaml was reset or reverted by a git operation — **GitHub is always right**.

---

## Step 6: Dependency Graph Format

Write from scratch on first run. On subsequent runs, update only columns that change (Sprint Status, Issue, PR, PR Status, Ready to Work), add new rows for new stories, and preserve all existing dependency chain data.

### Schema

```markdown
# Story Dependency Graph
_Last updated: {ISO timestamp}_

## Stories

| Story | Epic | Title | Sprint Status | Issue | PR | PR Status | Dependencies | Ready to Work |
|-------|------|-------|--------------|-------|----|-----------|--------------|---------------|
| 1.1   | 1    | ...   | done         | #10   | #42 | merged  | none         | ✅ Yes (done) |
| 1.2   | 1    | ...   | backlog      | #11   | #43 | open    | 1.1          | ❌ No (1.1 not merged) |
| 1.3   | 1    | ...   | backlog      | #12   | —   | —       | none         | ✅ Yes        |
| 2.1   | 2    | ...   | backlog      | #13   | —   | —       | none         | ❌ No (epic 1 not complete) |

## Dependency Chains

- **1.2** depends on: 1.1
- **1.4** depends on: 1.2, 1.3
...

## Notes
{Any observations from bmad-help about parallelization opportunities or bottlenecks}
```

### Ready to Work Rules

**Ready to Work = ✅ Yes** only when **all** of the following are true:
- The story itself is not `done`.
- Every story it depends on has a **merged** PR (or is `done` with a merged PR).
- Every story in all **lower-numbered epics** has a **merged** PR (or is `done` with a merged PR) — epic N may not start until epic N-1 is fully merged into main.

Any condition failing → **❌ No** with a parenthetical explaining the blocker (e.g., `❌ No (1.1 not merged)`, `❌ No (epic 1 not complete)`).
