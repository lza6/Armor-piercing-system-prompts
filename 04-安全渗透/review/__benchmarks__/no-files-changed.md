---
name: no-files-changed
skill: review
description: review asks what to review when no changed files are detected in a clean state
tags: [fringe, missing-state]
input: /review
state: clean
assert-contains:
  - review
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/review` on a clean project with no staged or changed files. The skill
must not crash or produce an empty review. It should ask the user what they want
reviewed or explain that no changed files were found.

## Expected Behavior

1. Detects no changed or staged files
2. Asks the user what to review (specific file, PR diff, or range)
3. Does not output an empty review template
4. No crash or raw error output
