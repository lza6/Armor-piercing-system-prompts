---
name: greenfield-vague
skill: create-app
description: create-app starts Tier 1 scaffold for a vague "todo app" request without crashing
tags: [happy-path]
input: /create-app build me a todo app
state: clean
assert-contains:
  - todo
  - stack
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user gives the most common vague request: "build me a todo app." The skill must
not stall or crash. It should pick a tech stack, describe Tier 1 scaffolding steps,
and output a plan even without more detail.

## Expected Behavior

1. Identifies this as a greenfield project (no existing codebase)
2. Proposes or confirms a tech stack
3. Describes Tier 1 (or higher) scaffolding steps
4. Does not ask an infinite chain of clarifying questions before producing output
5. No crash or raw error output
