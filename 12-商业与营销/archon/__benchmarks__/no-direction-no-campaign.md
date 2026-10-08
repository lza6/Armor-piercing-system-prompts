---
name: no-direction-no-campaign
skill: archon
description: Archon runs health diagnostic when invoked with no direction and no active campaign
tags: [fringe, missing-state]
input: /archon
state: clean
assert-contains:
  - health
assert-not-contains:
  - ENOENT
  - TypeError
  - Cannot read
  - undefined
  - crashed
---

## What This Tests

A user runs `/archon` with no arguments and no campaign files present. Archon must
recognize the undirected mode and produce a health diagnostic rather than crashing
or hanging. This is the most common first-invocation experience for new users.

## Expected Behavior

1. Archon detects no active campaigns and no direction given
2. Runs the Health Diagnostic protocol
3. Checks `.planning/intake/` for pending items (may be empty or absent)
4. Outputs a human-readable summary with suggestions like "No active work. Give me a direction or run `/do status`."
5. No stack traces or raw error messages appear
