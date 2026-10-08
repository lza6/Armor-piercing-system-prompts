---
name: mid-campaign
skill: session-handoff
description: session-handoff emits a full handoff block with campaign state and next steps
tags: [happy-path]
input: /session-handoff
state: with-campaign
assert-contains:
  - campaign
  - next
  - HANDOFF
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user runs `/session-handoff` while a campaign is active. The skill must produce
a structured handoff block containing campaign state, current phase, and recommended
next steps for the incoming session.

## Expected Behavior

1. Detects the active campaign from project state
2. Outputs a HANDOFF block (the three-dash format)
3. Includes current campaign context or phase
4. Includes at least one "next steps" item
5. No crash or raw error output
