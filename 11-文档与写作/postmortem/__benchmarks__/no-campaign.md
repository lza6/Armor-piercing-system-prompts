---
name: no-campaign
skill: postmortem
description: postmortem outputs a helpful message when no completed campaigns exist
tags: [fringe, missing-state]
input: /postmortem
state: clean
assert-contains:
  - campaign
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/postmortem` on a clean project with no completed campaigns. The skill
must not crash. It should output a message explaining there are no completed campaigns
to analyze, or guide the user toward running a campaign first.

## Expected Behavior

1. Detects no completed campaigns exist
2. Outputs a helpful message ("no completed campaigns found" or similar)
3. Does not produce a blank output or raw error
4. No crash or raw error output
