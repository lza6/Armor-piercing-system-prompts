---
name: with-error
skill: systematic-debugging
description: systematic-debugging performs 4-phase analysis for a TypeError in auth middleware
tags: [happy-path]
input: /systematic-debugging TypeError: cannot read properties of undefined in auth middleware
state: clean
assert-contains:
  - auth
assert-not-contains:
  - ENOENT
  - SyntaxError
---

## What This Tests

A user provides a concrete error message. The skill must apply its structured
debugging methodology — at minimum identifying phases (reproduce, isolate, hypothesize,
verify) — and generate at least one hypothesis about the root cause.

## Expected Behavior

1. Parses the error: TypeError in auth middleware, undefined property access
2. Applies a structured debugging approach (phases or steps)
3. Generates at least one hypothesis about the root cause
4. Suggests a concrete next investigation step
5. No crash or raw error output
