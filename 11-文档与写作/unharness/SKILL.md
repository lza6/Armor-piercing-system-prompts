---
name: unharness
license: MIT
description: >-
  Safely leave Citadel using the active adoption receipt. Produces a no-write,
  reviewable plan, preserves a portable archive, removes only exact owned
  material, and reports modified or externally registered surfaces as retained
  or unknown. Legacy installs must be imported before exact leave is claimed.
user-invocable: true
auto-trigger: false
trigger_keywords:
  - unharness
  - remove citadel
  - uninstall citadel
  - clean up citadel
  - remove harness
  - uninstall harness
last-updated: 2026-07-30
---

# /unharness - Receipt-owned exit from Citadel

## Orientation

Use this skill when the user wants Citadel removed from a project. Exit is a
two-step, receipt-owned operation. A plan never mutates the project. Apply
requires the exact saved plan and its confirmation token.

Do not infer ownership from familiar paths. Do not delete `.planning/`,
`.citadel/`, runtime settings, hooks, skills, or registrations merely because
they look Citadel-related.

## Protocol

### 1. Locate Citadel and inspect adoption authority

Read `.citadel/plugin-root.txt` when present. Otherwise use the directory
containing this skill. Check for `.citadel/adoption/active.json`; the adoption
planner also checks the private recovery ledger.

### 2. Create a no-write leave plan

Choose a plan path inside the project, such as
`.planning/adoption/leave.plan.json`, and run:

```bash
node {citadelRoot}/scripts/adopt.js leave plan \
  --target {projectRoot} \
  --out {planPath} \
  --json
```

Show the user:

- exact files that will be removed or restored;
- modified or ambiguous footprint entries that will be retained;
- portable archive path;
- runtime registrations whose removal evidence is `unknown`;
- the plan digest and confirmation token.

If the result is `NOT_ADOPTED`, stop. Create a conservative legacy inventory:

```bash
node {citadelRoot}/scripts/adopt.js import plan {citadelRoot} \
  --target {projectRoot} \
  --out {importPlanPath} \
  --json
```

Apply that import only after its own explicit approval. Then create a new leave
plan from the resulting receipt. Never describe legacy cleanup as exact
removal.

### 3. Obtain explicit approval

Leaving is destructive. Ask the user to approve the displayed plan and exact
confirmation token. Do not apply from an unsaved or regenerated plan.

### 4. Apply and verify

After approval:

```bash
node {citadelRoot}/scripts/adopt.js leave apply {planPath} \
  --confirm {confirmationToken} \
  --json

node {citadelRoot}/scripts/adopt.js doctor \
  --target {projectRoot} \
  --json
```

Doctor may report `not_adopted` after a clean leave. Report the leave receipt,
archive path, removed entries, retained conflicts, and every `unknown`
unregistration observation. “Citadel removed” is permitted only when the
receipt proves every required local and external removal.

## Export-only compatibility

When the user explicitly wants a human-readable export without leaving:

```bash
node {citadelRoot}/scripts/unharness.js {projectRoot} --export-only
```

This writes the legacy Markdown archive but does not remove harness material.

## Fringe Cases

- If `.planning/` does not exist, treat it as an empty portable-state set and
  show the setup/import hint; do not create it merely to leave.
- Missing or invalid adoption authority blocks leave. Use `import plan` or
  private-ledger recovery; never guess ownership.
- A changed saved plan, source, target, or pre-image requires a fresh plan.
- An unavailable runtime unregister API stays `unknown`; report the manual
  observation required.
- If the Citadel root cannot be located, stop with the exact missing path.

## Quality gates

- Planning is read-only unless the user explicitly requests `--out`.
- Apply consumes the exact saved plan and revalidates source, target, and
  pre-images.
- Modified owned material is retained with an explicit conflict.
- Shared runtime files restore exact pre-images only while installed bytes
  remain exact.
- Missing receipts, malformed receipts, and unenumerable registrations are
  `unknown` or blocked, never successful.
- The portable archive and private receipt ledger remain available for restore
  and recovery.

## Exit Protocol

Return the leave operation ID and receipt digest, portable archive path, exact
removed/restored counts, retained conflicts, and unknown external removals.
State plainly whether exact exit was proved. Do not claim success from a
completed command when required removal evidence is missing.
