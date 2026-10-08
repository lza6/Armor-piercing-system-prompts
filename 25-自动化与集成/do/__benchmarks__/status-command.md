---
name: status-command
skill: do
description: /do status routes directly to /dashboard
tags: [happy-path]
input: /do status
state: with-campaign
assert-contains:
  - dashboard
  - CAMPAIGNS
assert-not-contains:
  - ENOENT
  - TypeError
  - undefined
---

## What This Tests

A user runs `/do status` to check harness state. This is a Tier 0 pattern match
that must route immediately to `/dashboard` without going through complexity
classification. The dashboard output must appear with campaign information.

## Expected Behavior

1. Tier 0 pattern match catches "status" immediately
2. Routes to `/dashboard` and invokes it
3. Dashboard output includes CAMPAIGNS section and active campaign name
4. No routing overhead or classification delay
5. Full dashboard output is relayed to the user
