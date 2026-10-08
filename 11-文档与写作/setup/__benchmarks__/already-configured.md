---
name: already-configured
skill: setup
description: setup detects existing harness.json and confirms or skips re-initialization
tags: [fringe]
input: /setup
state: with-campaign
assert-contains:
  - config
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/setup` on a project that already has a campaign (implying prior setup).
The skill must detect the existing configuration and either confirm the current config
or ask before overwriting, rather than blindly re-running setup.

## Expected Behavior

1. Detects existing configuration (harness.json or equivalent)
2. Informs the user that configuration already exists
3. Asks to confirm before overwriting, or reports the current config
4. Does not silently overwrite existing setup
5. No crash or raw error output
