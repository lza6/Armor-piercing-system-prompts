# {Program Name} — Program SPEC

> **Program-level (umbrella) SPEC.** Written ONCE during the outer loop of a phase program. It governs every inner phase. The inner loop never writes its own SPEC — this document is the requirements doc for all phases.
>
> **Location:** the umbrella task folder, named `{program-slug}_SPEC_{date}.md`, alongside `{program-slug}_PLAN_{date}.md`.
>
> Written for **user review**: plain language, no file paths, no library names, no schema or code.

---

## Summary

<!-- One plain-language paragraph a non-engineer can read: what this program builds and why it matters to the user. -->

## User Stories / Jobs To Be Done

<!-- The heart of the doc. One entry per distinct user goal.
- As a [user], I want [X], so that [Y].
- When [situation], I want to [motivation], so I can [outcome].
-->

## What The User Wants (Behavioral Outcomes)

<!-- Observable outcomes only — what the user sees and can do across the whole program. No implementation. -->

## Flow / State Diagram

<!-- At least one ASCII diagram of the user journey or state changes. Show the happy path and important branches. -->

```
[start] --> [...] --> [outcome]
```

## Acceptance Criteria (Testable Outcomes)

<!--
Each criterion is an observable outcome a reviewer recognizes as "what I want."
Each carries `proven by:` (named test scenario/gate) + `strategy:` (Fully-Automated / Hybrid / Agent-Probe).
Every criterion must be provable by comprehensive tests, with a fully-automated E2E/integration gate
wherever the behavior is automatable. Agent-Probe / Known-Gap only as explicitly-justified residual.
Vacuous-green is BANNED. Scenarios grounded in RESEARCH test-context-discovery, not invented here.
Maximum 20 criteria.
-->

- AC1: <observable outcome>
  proven by: <named test scenario/gate>
  strategy: Fully-Automated

## Out Of Scope

<!-- At least one item. What this program explicitly does NOT do, even things that seem related. -->

## Constraints

<!-- User-stated requirements, system/process rules, hard technical limits from research. -->

## Open Questions

<!-- Each with an owner (user / research / next-phase). Must be empty/"None" (or backlogged under /goal) before PLAN begins. -->

- None

## Background / Research Findings

<!-- Key facts from RESEARCH that shaped these requirements + user brainstorm input. Supporting context, kept at the bottom. -->

---

## SPEC Gaps (inner-loop reference)

> **Convention note — not part of the umbrella SPEC body.** This program SPEC is frozen once INNOVATE/PLAN begins. When an inner-loop phase discovers a scope gap versus this umbrella SPEC, it does **not** edit this file. Instead the phase report records the gap under its own `## SPEC Gaps` heading (the convention named in the update-process protocol) plus a backlog note, and the loop continues. UPDATE PROCESS scores SPEC achievement per acceptance criterion at closeout; any unmet criterion becomes a backlog note.
