---
name: active-campaign-resume
skill: archon
description: Archon resumes an active campaign from its Continuation State
tags: [happy-path]
input: /archon
state: with-campaign
assert-contains:
  - Phase
  - campaign
assert-not-contains:
  - ENOENT
  - TypeError
  - undefined
  - Cannot read
---

## What This Tests

A user returns to a project mid-campaign. Archon must detect the active campaign file,
read its Continuation State, and resume from the correct phase without asking the user
to re-explain the work. This is the core value of the multi-session persistence model.

## Expected Behavior

1. Archon reads `.planning/campaigns/` and finds a file with `Status: active`
2. Outputs a "Resuming" message that includes the campaign name and current phase
3. Reads the Continuation State and picks up from where it left off
4. Does not restart the campaign from Phase 1
5. Produces a HANDOFF at the end of the session
