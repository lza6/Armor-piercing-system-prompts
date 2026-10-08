---
name: component-from-exemplar
skill: scaffold
description: Scaffold generates a new component that matches the project's existing conventions
tags: [happy-path]
input: /scaffold new component SettingsPanel
state: with-source
assert-contains:
  - SettingsPanel
  - conventions
assert-not-contains:
  - ENOENT
  - TypeError
  - undefined
---

## What This Tests

The core happy-path flow: scaffold finds 2+ existing component exemplars, analyzes
their conventions (naming, imports, exports, structure), generates a new SettingsPanel
component matching those conventions exactly, and wires it into the project.

## Expected Behavior

1. Scaffold identifies the target type (component) and name (SettingsPanel)
2. Finds 2-3 existing component files as exemplars
3. Outputs a 3-5 line convention analysis
4. Generates the component file with no placeholder comments or TODO stubs
5. Wires the new file into the project's barrel exports or route registration
6. Runs typecheck and reports PASS
7. Outputs "SCAFFOLD COMPLETE" with the created files listed
