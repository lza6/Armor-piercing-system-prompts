---
name: no-server
skill: qa
description: qa reports that no dev server is running rather than hanging or crashing
tags: [fringe, missing-state]
input: /qa test the dashboard
state: clean
timeout: 240000
assert-contains:
  - server
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/qa test the dashboard` but no dev server is running. The skill must
detect that the server is unavailable and produce a helpful message, not hang waiting
for a connection or crash.

## Expected Behavior

1. Detects no running dev server (or times out gracefully)
2. Outputs a message mentioning the server situation
3. Suggests starting the dev server before running QA
4. Does not hang indefinitely waiting for a server response
5. No crash or raw error output
