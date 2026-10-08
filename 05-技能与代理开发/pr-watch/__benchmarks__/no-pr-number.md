---
name: no-pr-number
skill: pr-watch
description: /pr-watch without a PR number shows usage instructions
tags: [fringe, wrong-input]
input: /pr-watch
state: clean
assert-contains:
  - pr-watch
assert-not-contains:
  - TypeError
  - undefined
  - Cannot read properties
  - ENOENT
---

## What This Tests

A user types `/pr-watch` without a PR number. This is a common mistake.
The skill should show usage instructions or ask for the PR number — not
crash trying to process an undefined PR number.

## Expected Behavior

1. Detects that no PR number was provided
2. Shows usage instructions (e.g., "/pr-watch {number}")
3. Optionally: lists open PRs using `gh pr list` so user can choose
4. Does not crash or produce an error dump
