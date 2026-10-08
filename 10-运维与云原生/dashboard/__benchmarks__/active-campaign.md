---
name: active-campaign
skill: dashboard
description: Dashboard shows active campaign name, phase progress, and recent telemetry
tags: [happy-path]
input: /dashboard
state: with-campaign
assert-contains:
  - Citadel Dashboard
  - test-campaign
  - Phase
  - CAMPAIGNS
  - RECENT ACTIVITY
assert-not-contains:
  - ENOENT
  - undefined
  - raw JSON
  - "{\"hook\""
---

## What This Tests

A user mid-campaign who runs the dashboard to check project state.
The dashboard must surface the campaign name and phase without exposing
raw JSON entries from the telemetry files.

## Expected Behavior

1. Campaign "test-campaign" appears with phase progress (e.g., "Phase 2/3")
2. The direction or a truncated version appears
3. Recent activity shows the telemetry entries in human-readable form
4. No raw JSON objects appear in the output
