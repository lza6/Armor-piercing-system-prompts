---
name: specific-refactor
skill: refactor
description: refactor produces a step-by-step plan to extract auth logic into a separate module
tags: [happy-path]
input: /refactor extract the auth logic into a separate module
state: with-campaign
assert-contains:
  - auth
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user provides a concrete refactoring target on a project with existing context.
The skill must produce a step-by-step refactoring plan rather than making blind edits.

## Expected Behavior

1. Identifies the target: auth logic extraction
2. Produces a numbered plan or steps
3. Mentions the destination module structure
4. Does not make destructive edits without a plan
5. No crash or raw error output
