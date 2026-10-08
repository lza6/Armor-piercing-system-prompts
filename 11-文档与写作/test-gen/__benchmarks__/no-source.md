---
name: no-source
skill: test-gen
description: test-gen asks what to test when invoked with no target on a clean project
tags: [fringe, missing-state]
input: /test-gen
state: clean
assert-contains:
  - test
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/test-gen` with no arguments on a clean project. The skill must not
crash or generate empty test files. It should ask the user what source file or
module they want to test.

## Expected Behavior

1. Detects no source target was specified
2. Asks the user what to generate tests for
3. Does not create empty test files
4. No crash or raw error output
