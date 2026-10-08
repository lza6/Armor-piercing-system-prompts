---
name: list-empty
skill: schedule
description: /schedule list gives a clear message when no schedules are active
tags: [fringe, missing-state]
input: /schedule list
state: clean
assert-contains:
  - no scheduled
assert-not-contains:
  - Error
  - undefined
  - TypeError
---

## What This Tests

A user runs `/schedule list` when they haven't created any schedules yet.
The skill must output the "no schedules" message from the spec — not an
empty output, not a crash from CronList returning nothing.

## Expected Behavior

1. Calls CronList (or handles gracefully if it returns empty)
2. Outputs a clear "No active schedules" message
3. Suggests how to create one: `/schedule add`
4. No errors or empty output
