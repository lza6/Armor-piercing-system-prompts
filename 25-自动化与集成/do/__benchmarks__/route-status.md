---
name: route-status
skill: do
description: "what's happening" routes to dashboard, not a confused response
tags: [fringe, routing, wrong-wording]
input: what's happening
state: clean
assert-contains:
  - Citadel Dashboard
assert-not-contains:
  - I don't understand
  - I'm not sure
  - unclear
  - ENOENT
---

## What This Tests

Users don't read docs. They say natural things like "what's happening" or
"show me what's going on" and expect a dashboard. The `/do` router must
catch this phrase via Tier 0 pattern matching.

This also validates the user's assumption correction principle: the user
doesn't know the exact command but their intent is clear.

## Expected Behavior

1. Tier 0 pattern matches "what's happening" → routes to /dashboard
2. Dashboard renders with the standard format
3. No routing explanation needed (Tier 0 = silent, just do it)
