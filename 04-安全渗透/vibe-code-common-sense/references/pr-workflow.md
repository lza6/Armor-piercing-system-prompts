# PR Workflow

Use this workflow for AI-assisted project builds.

## Before Work

1. Sync the default branch.
2. Read `docs/PRODUCT_TIMELINE.md`.
3. Confirm the first unchecked item.
4. Create a focused branch.

## During Work

1. Implement only the selected item.
2. Add or update tests when the change is executable.
3. Update docs when behavior, scope, or decisions change.
4. Keep unrelated cleanup out of the PR.

## After Opening The PR

Only perform these steps when the user asked for a PR workflow and the repository
has a configured remote.

1. Add the PR number to the matching timeline item.
2. Mark that item `[x]`.
3. Push the tracker commit.
4. Run or wait for verification.
5. Report the next unchecked item.

## Completion Evidence

A ready PR should have:

- passing local checks or documented manual verification;
- passing CI when available;
- a clean working tree;
- no unexpected files;
- a timeline update with the PR number.
