---
name: vague-idea
skill: prd
description: prd asks clarifying questions when the idea is too vague to write requirements
tags: [fringe]
input: /prd I want to build an app
state: clean
assert-contains:
  - app
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
  - Cannot read
---

## What This Tests

A user provides the most vague possible input: "I want to build an app." The skill
must not crash or output a useless empty PRD. It should ask clarifying questions
about the app's purpose, target users, and key features before proceeding.

## Expected Behavior

1. Recognizes the input is too vague to write concrete requirements
2. Asks at least one clarifying question (purpose, users, features, or platform)
3. Does not generate a full PRD with placeholder content
4. No crash or raw error output
