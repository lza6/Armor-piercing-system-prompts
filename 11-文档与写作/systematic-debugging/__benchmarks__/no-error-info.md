---
name: no-error-info
skill: systematic-debugging
description: systematic-debugging asks for an error description when invoked with no input
tags: [fringe, missing-state]
input: /systematic-debugging
state: clean
assert-contains:
  - error
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/systematic-debugging` with no error description. The skill must not
crash or start a debugging session with no target. It should ask the user to describe
the error they are investigating.

## Expected Behavior

1. Detects no error information was provided
2. Asks the user to describe the error or symptom
3. Does not begin hypothesis generation with no information
4. No crash or raw error output
