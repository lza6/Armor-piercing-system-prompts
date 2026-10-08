---
name: with-completed
skill: postmortem
description: postmortem produces a retrospective for a completed campaign including what worked and what broke
tags: [happy-path]
input: /postmortem auth-overhaul
state: with-completed-campaign
assert-contains:
  - auth-overhaul
  - recommendations
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user runs `/postmortem auth-overhaul` against a project with a completed campaign.
The skill must produce a structured retrospective mentioning the campaign name and
covering what worked and what broke.

## Expected Behavior

1. Locates the completed campaign named "auth-overhaul"
2. Outputs the campaign name in the retrospective
3. Includes a "what worked" section
4. Includes a "what broke" or "issues encountered" section
5. No crash or raw error output
