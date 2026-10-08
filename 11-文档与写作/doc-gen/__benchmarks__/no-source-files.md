---
name: no-source-files
skill: doc-gen
description: doc-gen outputs a helpful message instead of crashing when no source files are found
tags: [fringe, missing-state]
input: /doc-gen
state: clean
assert-contains:
  - doc
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user runs `/doc-gen` on a clean project with no source files. The skill must not
crash or output a raw filesystem error. It should output a helpful message explaining
that no source files were found or ask the user what to document.

## Expected Behavior

1. Detects no source files in the project
2. Outputs a helpful message (asks what to document, or explains the situation)
3. Does not crash with ENOENT or similar filesystem errors
4. No raw error output
