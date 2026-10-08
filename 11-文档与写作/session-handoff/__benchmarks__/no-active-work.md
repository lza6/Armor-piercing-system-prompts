---
name: no-active-work
skill: session-handoff
description: session-handoff produces a minimal handoff or "nothing active" message on a clean project
tags: [fringe, missing-state]
input: /session-handoff
state: clean
assert-contains:
  - HANDOFF
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/session-handoff` on a clean project with no active campaigns, fleet sessions,
or in-progress work. The skill must not crash. It should produce a minimal handoff block
or explain that nothing is currently active.

## Expected Behavior

1. Detects no active campaigns or in-progress work
2. Outputs either a minimal handoff block or a clear "nothing active" message
3. Does not crash with filesystem errors
4. No raw error output
