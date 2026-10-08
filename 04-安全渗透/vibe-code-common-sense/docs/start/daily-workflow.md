# Daily Workflow

Use this loop after the project rails exist.

## Before Work

1. Sync the default branch.
2. Read `docs/PRODUCT_TIMELINE.md`.
3. Pick the first unchecked item.
4. Create a small branch.

## During Work

1. Build only that item.
2. Update docs if scope or behavior changes.
3. Add tests or verification notes.
4. Avoid unrelated cleanup.

## After Opening The PR

1. Add the PR number to the matching timeline item.
2. Change `[ ]` to `[x]`.
3. Push the tracker update.
4. Wait for checks or run local verification.
5. Report the next unchecked item.

## After Merge

Tell the agent:

```text
Merged. Sync main, read docs/PRODUCT_TIMELINE.md, and build the next PR.
Keep the tracker updated.
```
