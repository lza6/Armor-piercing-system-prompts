<!--
name: "Agent Prompt: /code-review inline gap sweep phase"
description: "Adds a final same-context sweep for defects missed by an inline code review when subagents are unavailable"
ccVersion: "2.1.213"
variables:
  - "SWEEP_FOCUS"
-->

## Phase 3 — Sweep for gaps

Take one more pass yourself (same context, no subagent) as a fresh reviewer
who has the deduplicated list. Re-read the diff and enclosing functions
looking ONLY for defects not already listed: ${SWEEP_FOCUS}
