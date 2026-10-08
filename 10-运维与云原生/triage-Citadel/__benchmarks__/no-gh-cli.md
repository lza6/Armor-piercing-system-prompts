---
name: no-gh-cli
skill: triage
description: triage outputs a helpful gh CLI setup message when gh is not found or not authenticated
tags: [fringe, missing-state]
input: /triage
state: clean
assert-contains:
  - repo
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/triage` on a machine where the GitHub CLI (`gh`) is not installed or not
authenticated. The skill must not crash with a raw command-not-found error. It should
output a helpful message about installing or authenticating with `gh`.

## Expected Behavior

1. Detects that `gh` is unavailable or not authenticated
2. Outputs a helpful message mentioning `gh`
3. Suggests an install or auth step
4. Does not crash with a raw shell error
5. No raw error output
