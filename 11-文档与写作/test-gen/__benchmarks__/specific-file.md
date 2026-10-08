---
name: specific-file
skill: test-gen
description: test-gen generates test structure and assertions for an auth middleware file
tags: [happy-path]
input: /test-gen write tests for the auth middleware
state: with-campaign
assert-contains:
  - auth
  - test
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user asks for tests for a specific module on a project with existing context.
The skill must generate a test file with meaningful test cases and assertions,
not a blank or placeholder file.

## Expected Behavior

1. Identifies the target: auth middleware
2. Generates test cases covering at least one happy path and one error path
3. Includes assertion statements (expect, assert, or equivalent)
4. Structure is valid for the detected test framework (Jest, Vitest, etc.)
5. No crash or raw error output
