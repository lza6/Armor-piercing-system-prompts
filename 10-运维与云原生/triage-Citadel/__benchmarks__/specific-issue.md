---
name: specific-issue
skill: triage
description: triage produces investigation steps for a described auth failure without crashing
tags: [happy-path]
input: /triage investigate why auth is failing
state: clean
assert-contains:
  - auth
  - GitHub
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user describes a problem to triage rather than passing a GitHub issue number. The
skill must produce investigation steps or analysis without requiring a GitHub issue
to be present — a graceful degradation to local analysis mode.

## Expected Behavior

1. Accepts the free-form problem description: auth is failing
2. Produces analysis steps or a triage plan
3. Does not require a GitHub issue number to proceed
4. Includes at least one concrete investigation step
5. No crash or raw error output
