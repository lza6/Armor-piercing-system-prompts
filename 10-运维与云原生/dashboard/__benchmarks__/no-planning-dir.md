---
name: no-planning-dir
skill: dashboard
description: Dashboard renders with empty state when .planning/ does not exist
tags: [fringe, missing-state]
input: /dashboard
state: clean
assert-contains:
  - Citadel Dashboard
  - CAMPAIGNS
  - FLEET
  - PENDING
  - HEALTH
  - /do setup
assert-not-contains:
  - ENOENT
  - SyntaxError
  - TypeError
  - undefined
  - Cannot read
---

## What This Tests

A user who has just installed the harness but has not run `/do setup` yet.
The dashboard must render completely with empty/zero state and guide them
toward initialization — not crash or show raw Node.js error messages.

This is the most common first-run experience. Getting it wrong means the
user's first interaction with Citadel is a broken error dump.

## Expected Behavior

1. Dashboard renders the full template (all sections present)
2. All counts show 0 or "(none active)"
3. A visible hint to run `/do setup` appears
4. No raw file system errors leak into the output
