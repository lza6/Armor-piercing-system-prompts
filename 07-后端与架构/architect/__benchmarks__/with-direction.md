---
name: with-direction
skill: architect
description: Architect produces phases, file structure, and decisions for a REST API spec
tags: [happy-path]
input: /architect build a REST API with auth and posts
state: clean
assert-contains:
  - phase
  - auth
assert-not-contains:
  - ENOENT
  - TypeError
  - SyntaxError
  - undefined
---

## What This Tests

A user provides a concrete architectural direction inline. The skill must generate
a meaningful architectural plan including phases, a proposed file/module structure,
and key decisions (tech stack, auth strategy, data model).

## Expected Behavior

1. Architect acknowledges the direction: REST API with auth and posts
2. Produces at least two phases (e.g., foundation, feature build)
3. Includes a file or module structure section
4. Calls out at least one architectural decision (e.g., JWT vs session auth)
5. No crash or raw error output
