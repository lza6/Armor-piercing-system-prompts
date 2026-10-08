---
name: no-target
skill: refactor
description: refactor asks what to refactor when invoked with no target specified
tags: [fringe, missing-state]
input: /refactor
state: clean
assert-contains:
  - refactor
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/refactor` with no arguments on a clean project. The skill must not
crash or start blindly refactoring random files. It should ask the user what they
want to refactor.

## Expected Behavior

1. Detects no target was specified
2. Asks the user what to refactor (file, module, or pattern)
3. Does not attempt to modify any files without a target
4. No crash or raw error output
