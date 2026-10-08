---
name: empty-intake
skill: autopilot
description: Autopilot exits cleanly when .planning/intake/ is empty or absent
tags: [fringe, missing-state]
input: /autopilot
state: clean
assert-contains:
  - nothing
  - intake
assert-not-contains:
  - ENOENT
  - TypeError
  - Cannot read
  - undefined
  - crashed
---

## What This Tests

A user runs `/autopilot` but has not yet dropped any files into `.planning/intake/`.
Autopilot must recognize the empty state, communicate clearly that there is nothing
to process, and exit without crashing or producing a confusing error.

## Expected Behavior

1. Autopilot scans `.planning/intake/` (which is empty or does not exist)
2. Outputs a clear message indicating nothing was found to process
3. Optionally hints at how to add intake items or run `/do setup`
4. Exits with a clean HANDOFF showing 0 items processed
5. No raw filesystem errors appear in the output
