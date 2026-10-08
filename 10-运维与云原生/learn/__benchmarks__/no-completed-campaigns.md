---
name: no-completed-campaigns
skill: learn
description: /learn gives a clear message when no completed campaigns exist
tags: [fringe, missing-state]
input: /learn
state: clean
assert-contains:
  - No completed campaigns
assert-not-contains:
  - ENOENT
  - undefined
  - TypeError
  - Cannot read properties
---

## What This Tests

A user runs `/learn` on a fresh project or before completing any campaigns.
The skill must give a helpful message — not crash trying to read files that
don't exist.

This catches the "glob on nonexistent directory" failure mode.

## Expected Behavior

1. Outputs a clear message like "No completed campaigns found."
2. Suggests what to do next (complete a campaign first, or run /archon)
3. Does not crash with a Node.js file system error
4. Does not write empty knowledge files
