# SPEC Contract — Section-by-Section Writing Rules

This reference is the full writing contract for a product-discovery SPEC. The SKILL.md body stays short; the detailed rules live here.

A SPEC is a **readable product doc** written for **user review**. Lead with the user's perspective; keep the language plain. A non-technical stakeholder should read it and understand: what changes for the user, how we will know it worked, and what we are deliberately not doing.

**Audience first.** If a sentence only makes sense to an engineer, rewrite it or move it to `## Background / Research Findings`. No library names, no file paths, no schema decisions, no code — those belong to INNOVATE/PLAN/EXECUTE, downstream.

---

## Canonical Section Order

Every SPEC file has these sections, in this exact order:

1. `## Summary`
2. `## User Stories / Jobs To Be Done`
3. `## What The User Wants (Behavioral Outcomes)`
4. `## Flow / State Diagram`
5. `## Acceptance Criteria (Testable Outcomes)`
6. `## Out Of Scope`
7. `## Constraints`
8. `## Open Questions`
9. `## Background / Research Findings`

---

## Section Rules

### `## Summary`

One paragraph, plain language — the "elevator pitch" of what changes for the user and why it matters. A non-engineer must be able to read this and understand the point.

### `## User Stories / Jobs To Be Done`

The heart of the SPEC — everything else supports it. Use one of:

- User story: `As a [user], I want [X], so that [Y]`
- JTBD: `When [situation], I want to [motivation], so I can [outcome]`

One entry per distinct user goal. Capture the user's perspective, not the system's.

### `## What The User Wants (Behavioral Outcomes)`

Plain-language description of what the experience does, observed from the outside. Observable outcomes only — what the user sees and can do. No file paths, no database operations, no API calls, no library names.

### `## Flow / State Diagram`

At least one ASCII flow or state diagram of the user journey. Show the happy path; annotate the important branches and error states. A diagram communicates intent to a reviewer faster than paragraphs. Example shape:

```
[user starts] --> [does X] --> {valid?} --yes--> [sees result]
                                   |
                                  no
                                   v
                            [sees error, retries]
```

### `## Acceptance Criteria (Testable Outcomes)`

How the user (and we) will know it's done. Each criterion is a concrete, observable outcome stated so a reviewer can say "yes, that's what I want." The outcome itself reads as an observable statement, **not** a test command. Independently verifiable. No criterion references a file path or implementation detail.

- Minimum: one criterion per user-story area.
- Maximum: 20 criteria per SPEC.

Every acceptance criterion MUST carry two secondary annotations, placed under the outcome (not in the headline):

- `proven by:` — the named test scenario/gate that verifies this criterion (requirement→test link).
- `strategy:` — one of `Fully-Automated` / `Hybrid` / `Agent-Probe`.

Example:

```
- AC1: When the user submits an empty form, they see an inline validation message and the form does not submit.
  proven by: form-validation-empty-submit scenario
  strategy: Fully-Automated
```

Hard rules for acceptance criteria:

- Every criterion MUST name the test scenario that proves it and its strategy. A criterion with no `proven by:` scenario is **incomplete**.
- **Every criterion MUST be provable by comprehensive tests, with a fully-automated E2E/integration gate wherever the behavior is automatable.** `Agent-Probe` or `Known-Gap` stand only as the explicitly-justified residual where automation is genuinely impossible (no automatable AND no hybrid/agent-probe coverage possible).
- Vacuous-green (zero automated gates on developed behavior) is **BANNED** as a terminal state.
- Scenario enumeration MUST be **grounded in the test-context-discovery performed in RESEARCH** — the full `process/context/tests/all-tests.md` router plus its downstream chain, with scenarios grouped by the 3 strategies. Scenarios come from that enumeration, **not invented** in the SPEC.

This requirement→test discipline applies to **all developed behavior** across every surface; INNOVATE, PLAN, VALIDATE, and UPDATE PROCESS all inherit the fully-automated E2E/integration gate wherever the behavior is automatable.

### `## Out Of Scope`

At least one item required. List what this explicitly does NOT do — even things that seem related. This is the product out-of-scope boundary, stated for the reviewer to protect both the user and the builder from scope creep.

### `## Constraints`

Hard limits the solution must respect: user-stated requirements, system/process rules, and known technical boundaries surfaced by research.

### `## Open Questions`

Unresolved intent items, each with an owner (user, research, or next-phase). Must be resolved (or deferred to backlog under /goal) before PLAN begins.

- If any remain open at finalize in an **interactive** session: emit `SPEC_INTENT_BLOCKED` and stop.
- Under **/goal**: record each as a backlog note and continue — a true /goal run pauses for nothing.

### `## Background / Research Findings`

Pull the key facts from RESEARCH that shaped the SPEC, plus the user's brainstorm input captured verbatim where it clarifies intent. Do not paste the full research output — extract only what shaped the requirements. This sits at the bottom: supporting context, not the headline.

---

## SPEC File Location

- Filename: `{slug}_SPEC_{dd-mm-yy}.md`
- Location: same task folder as the active plan — `active/{slug}_{date}/`
- Phase program: the umbrella task folder contains both `{program-slug}_SPEC_{date}.md` and `{program-slug}_PLAN_{date}.md`

---

## Frozen Document Rules

Once INNOVATE/PLAN begins, the SPEC is locked:

- Never edited during INNOVATE, PLAN, VALIDATE, or EXECUTE.
- If a scope gap is found later: note it in the phase report under `## SPEC Gaps` and add a backlog item. Do not emit any amendment signal.
- UPDATE PROCESS scores SPEC achievement: each acceptance criterion is evaluated after execution; any unmet criterion becomes a backlog note.

---

## Inner Loop Skips SPEC

In a phase program, the SPEC is written once during the outer loop and governs every inner phase. The inner loop (`R → I → P → PVL → E → EVL → UP`) never writes a per-phase SPEC. Inner-loop scope gaps go to the phase report `## SPEC Gaps` heading (already named in the update-process convention), then a backlog note — never a new SPEC file or amendment signal.
