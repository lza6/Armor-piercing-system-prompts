---
name: specific-file
skill: review
description: review applies the 5-pass review structure to a named campaign file
tags: [happy-path]
input: /review the campaign file
state: with-campaign
assert-contains:
  - pass
  - campaign
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user asks for a review of a specific file with a project in an active campaign state.
The skill must locate the file, apply its multi-pass review structure, and report findings.

## Expected Behavior

1. Locates the campaign file in the project state
2. Applies the review in multiple passes (at minimum 2 passes referenced)
3. Reports at least one finding or confirms the file is clean
4. Structures output clearly (not a wall of unformatted text)
5. No crash or raw error output
