---
name: existing-codebase
skill: create-app
description: create-app detects an existing project and switches to feature-addition mode
tags: [happy-path]
input: /create-app add dark mode to this app
state: with-campaign
assert-contains:
  - dark mode
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user runs `/create-app` with an intent to add a feature to an existing codebase
(indicated by the with-campaign state). The skill must detect existing files/context
and switch to feature-addition mode rather than generating a greenfield scaffold.

## Expected Behavior

1. Detects an existing project context from the campaign state
2. Enters feature-addition mode (not full greenfield scaffold)
3. References existing files or structure where relevant
4. Produces a plan to add dark mode (CSS variables, theme toggle, etc.)
5. No crash or raw error output
