---
name: specific-feature
skill: prd
description: prd generates a structured requirements document for a user authentication system
tags: [happy-path]
input: /prd build a user authentication system with OAuth
state: clean
assert-contains:
  - auth
  - OAuth
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user provides a concrete feature description. The skill must produce a structured
PRD with sections for requirements, constraints, and success criteria.

## Expected Behavior

1. Acknowledges the feature: user authentication with OAuth
2. Produces a PRD with at least: overview, requirements, and success criteria sections
3. Mentions OAuth providers or flow in the requirements
4. Structure is clear enough to hand to an architect or engineer
5. No crash or raw error output
