---
name: soul-grader
description: Use when grading, reviewing, rewriting, or approving a Hermes Agent SOUL.md. Uses the SOUL.md field-guide research artifacts as the only normative source for what makes a good SOUL.md.
version: 1.1.0
author: cobibean
license: MIT
metadata:
  hermes:
    tags: [soul-md, grading, hermes-agent, agent-identity, review, rubric]
    related_skills: [hermes-agent]
---

# SOUL Grader

## Linked references

- `references/fleet-soul-grading-workflow.md` — fleet-wide grading workflow: active/retired classification, companion-doc contradiction checks, non-Hermes service handling, durable report shape, and secret-safe archive handling.
- `references/research-deliverable-and-fleet-remediation.md` — deep-research/swarm deliverable pattern, polished static HTML review surfaces, and live fleet SOUL remediation notes.
- `references/phase-coupled-soul-refresh.md` — refresh pattern for a deployed agent whose durable identity has fallen behind its current phase, including unscored companion-doc placement checks.
- `references/live-soul-upgrade-production-handoff.md` — production handoff pattern for backup, target-score iteration, live profile verification, safe refresh, and behavior smoke testing.

## Overview

Use this skill to grade a Hermes Agent `SOUL.md`, draft a SOUL review, or turn a weak SOUL into a stronger one. The grading standard is intentionally narrow: **use the linked SOUL.md research artifacts as the only normative source for what makes a good SOUL.md.**

Do not import generic prompt-engineering advice, personal taste, web articles, model-provider docs, or vibes into the grade. You may use tools to read the SOUL being graded and to verify Hermes runtime facts, but quality judgments must come from the research artifacts bundled with this skill.

## Public / SSR publication posture

This is an unofficial community skill for Hermes Agent. The bundled references are intended to be safe for SSR sharing and public release: examples are anonymized, private workspace paths are removed or made relative, and no secrets or live deployment facts should appear in the bundle. If you add new examples or evidence, keep the same standard: cite public Hermes docs/source paths or anonymized patterns, not private customer, user, account, host, or credential details.

## Required source files

Before grading, load at least the grading standard reference:

```text
skill_view(name="soul-grader", file_path="references/soul-md-grading-standard.md")
```

Use the other references when you need more detail or citations:

```text
skill_view(name="soul-grader", file_path="references/soul-md-field-guide.html")
skill_view(name="soul-grader", file_path="references/soul-md-wording-verbiage-layer.md")
```

Source hierarchy for grading:

1. `references/soul-md-grading-standard.md` — canonical grader rubric and procedure.
2. `references/soul-md-wording-verbiage-layer.md` — supporting wording examples interpreted through the canonical standard.
3. `references/soul-md-field-guide.html` — historical research background; non-normative where it conflicts with the canonical standard.

If these references conflict, use the higher-ranked source. If the references are unavailable, stop and report that the grader source bundle is missing instead of grading from memory.

## When to use

Use when the user asks to:

- grade, score, audit, review, or approve a `SOUL.md`
- compare two `SOUL.md` files
- rewrite or improve a `SOUL.md`
- upgrade a `SOUL.md` to production handoff quality and iterate until a user-chosen score target is met
- check whether a new agent identity is ready to deploy
- create acceptance criteria for a SOUL file
- diagnose agent identity drift, overreach, genericness, or approval-boundary failures
- decide what belongs in `SOUL.md` versus `CLAUDE.md`, `AGENTS.md`, skills, memory, manifests, or operator guides

Do not use as the sole workflow for:

- installing or configuring Hermes itself — load `hermes-agent`
- full new-agent intake/design — load the project’s new-agent intake or design skill too, if one is available
- authoring a new skill — load `hermes-agent-skill-authoring` too
- editing a user’s actual SOUL file without permission

## Strict source rule

The target `SOUL.md` being graded is evidence, not a standard. Runtime/tool output is evidence about deployment state, not a standard. The bundled references are the standard.

Allowed sources:

- the user-provided SOUL text or file path
- optional adjacent files only for placement, contradiction, or system-level deployability evidence; never score their prose as part of the SOUL grade
- live Hermes/runtime output only for runtime hygiene checks
- this skill’s linked references

Not allowed as grading sources:

- generic prompt-engineering heuristics not present in the references
- web search results
- model-provider documentation
- personal preference unless the user explicitly asks for a custom overlay after the source-grounded grade
- unstated assumptions about a deployed agent’s access, host, credentials, or service state

## Grading rubric

Score out of 100 using the reference-defined categories:

| Category | Points | What to evaluate |
|---|---:|---|
| Mission clarity | 15 | Names who/what the agent serves and what outcome matters. |
| Identity + positive role boundaries | 12 | Defines role, ownership, layer, and adjacent responsibilities positively. |
| Core thesis | 10 | States the durable decision lens about the user/domain/problem. |
| Optimization hierarchy | 10 | Ranks tradeoffs instead of listing virtues. |
| Judgment + decision ownership | 10 | Explains tradeoff judgment and preserves consequential operator decisions. |
| Soft preferences | 8 | Separates scoring signals from bans. |
| Relationship + authority posture | 10 | Defines initiative, challenge, correction, and operator authority. |
| Voice + truthfulness posture | 10 | Covers distinctive voice plus candor and evidence orientation. |
| Success orientation | 8 | Defines meaningful user/product outcomes rather than activity. |
| Artifact separation + placement | 5 | Rewards correctly placed guidance; duplication earns no extra credit. |
| Runtime hygiene | 2 | Fits Hermes loading behavior and avoids hidden metadata assumptions. |

System-level automatic blockers:

- secrets, tokens, passwords, API keys, or connection strings in SOUL
- verified false claims of access, deployment state, health, publication, or authority
- verified missing or bypassed controls for spend, publishing, outreach, destructive edits, production mutations, or customer-visible actions
- cross-client data/credential/workspace contamination
- assuming YAML/frontmatter is hidden from Hermes native SOUL when it is visible prompt text
- contradictions with nearby operating files, manifests, or approval policy

These blockers apply to the deployed agent system, not to a missing paragraph in SOUL. Assert them only from direct companion-policy, manifest, runtime, or live-system evidence. If that evidence is absent, report **System blockers: not assessed**. They do not change the numeric SOUL score.

## Procedure

1. **Load the grading reference.** Use `skill_view` for `references/soul-md-grading-standard.md`. Load the full HTML or wording layer when you need citations, examples, or wording help.
2. **Get the SOUL.** If the user provides a path, read it with `read_file`. If they provide inline text, grade that. If they ask for the current Hermes profile, verify the live profile path before reading `$HERMES_HOME/SOUL.md`.
3. **Score the SOUL first.** Score all 11 rows without requiring approval matrices, tool rules, verification checklists, escalation procedures, or detailed DoD in SOUL.
4. **Award placement credit, not duplication credit.** Correctly absent operational detail strengthens artifact separation. Do not add points because the same rule appears in both SOUL and `AGENTS.md` / `CLAUDE.md`.
5. **Apply scope after the rubric.** Classify the agent as personal, business/internal, client/business, public/open-source, meta/operator, multi-agent peer, or temporary/tactical. Use scope as an unscored overlay for follow-up checks.
6. **Check system blockers only with evidence.** Inspect adjacent/system sources when provided. If they are unavailable, mark system blockers not assessed rather than treating SOUL omissions as failures.
7. **Route every suggestion.** Prefix fixes with `[SOUL]`, `[AGENTS/CLAUDE]`, or `[SYSTEM]`. Suggestions may explicitly say “put this in your AGENTS.md and/or CLAUDE.md.”
8. **Write findings as identity drift risks.** Explain how weak identity wording could cause a future session to drift.
9. **Give patches only when useful.** If the user asked for a rewrite, provide a replacement section or full revised SOUL. Otherwise give prioritized fixes.
10. **For “should we make these SOUL updates?” questions, re-read live state before opining.** Fetch/read the current live SOUL and the latest grade/report if available, compare them, and identify which suggested fixes are already present. Recommend a surgical patch, not a full rewrite, unless identity or scope changes require it. Do not apply the change unless the user explicitly asks you to edit.
11. **Cite the bundled sources.** Cite the linked reference section names and, when useful, file names. Do not cite outside sources.

## Output format: full grade

Use this shape by default:

```md
# SOUL.md grade: [agent/name]

Verdict: [Excellent / Operational / Scaffold / Needs rewrite]
Score: [N]/100
System deployability: [Approved / Blocked / Not assessed]

## Score table

| Category | Points | Score | Notes |
|---|---:|---:|---|
| Mission clarity | 15 |  |  |
...

## Scope overlay

Scope: [personal/business-internal/client-business/public/meta/multi-agent/tactical]
[Unscored scope-specific observations]

## System-level blockers

- [None verified / Not assessed / blocker with exact system evidence]

## Top identity drift risks

1. [Risk] — [where the SOUL permits drift]
2. [Risk] — [where the SOUL permits drift]
3. [Risk] — [where the SOUL permits drift]

## What is strong

- [Concrete strengths tied to source criteria]

## What to fix first

1. [SOUL] [Highest leverage identity fix]
2. [AGENTS/CLAUDE] [Operating-policy placement suggestion, if useful]
3. [SYSTEM] [System-control suggestion, if evidence warrants it]

## Suggested wording

```md
[patch/replacement sections, if requested or obviously helpful]
```

## Source basis

- `references/soul-md-grading-standard.md`: [sections used]
- `references/soul-md-field-guide.html`: [sections used]
- `references/soul-md-wording-verbiage-layer.md`: [sections used]
```

## Output format: quick grade

For quick review requests:

```md
Score: [N]/100 — [verdict]
System deployability: [Approved / Blocked / Not assessed]

Biggest issue: [one sentence]
Best thing: [one sentence]
Fix next:
1. ...
2. ...
3. ...
```

## Attachment / HTML delivery pattern

When the user asks to “send me the SOUL,” “put it in an HTML file,” or otherwise wants a reviewable artifact rather than only chat text:

1. Fetch or read the exact target `SOUL.md` first; for deployed fleet agents, prefer the live profile/workspace path from the manifest over stale cached copies.
2. Include both the raw SOUL and the grade in the delivered artifact unless the user explicitly asks for only one.
3. Produce a self-contained HTML file for reading when requested: embedded CSS, verdict cards, score table, system-level blockers, top drift risks, suggested wording, source basis, and raw SOUL in a readable `<pre>` block.
4. Also save a plain Markdown grade report and raw `SOUL.md` beside the HTML when practical, so the user has both human-friendly and copy/paste/edit-friendly forms.
5. Keep the artifact secret-safe: do not include raw credentials from manifests or companion docs; cite credential locations only if needed.
6. Use canonical roster/manifest spelling for the agent name in the report, while noting any user spelling variant only if it could cause confusion.

## Verdict bands

- **90–100 Excellent** — production-grade identity; keep it reviewed as scope changes.
- **75–89 Operational** — usable; patch missing layers before high-risk autonomy.
- **60–74 Scaffold** — serviceable draft; needs sharper identity, judgment, boundaries, or success orientation.
- **0–59 Needs rewrite** — rewrite from mission and identity upward.
- **System blocked** — a verified system-level blocker remains unresolved; this is separate from the SOUL score.

## Wording standards

Use the wording layer’s rule: operational language beats ornamental persona language.

Prefer:

- `You are [name], [user/client]’s [specific layer/domain] agent.`
- `You own [specific layer] and turn [input] into [outcome]; [operator/peer] owns [adjacent decisions].`
- `Own clear day-to-day decisions; preserve [operator]'s authority over [consequential decisions].`
- `Evidence outranks confidence; detailed claim thresholds belong in AGENTS.md and/or CLAUDE.md.`
- `Public-facing output must [brand/audience rule], not [private voice leak].`

Cut or rewrite:

- `helpful assistant`, `friendly and professional`, `be proactive`, `use best practices`
- `never hallucinate` without evidence thresholds
- mission statements that name vibes instead of outcomes
- tone adjectives without context behavior
- long command/runbook dumps
- exact ports/processes/current state
- secrets or credential values

## Deep research / swarm deliverables

When the user asks for a broad SOUL.md research project, a “swarm,” or a polished review artifact, use the workflow in `references/research-deliverable-and-fleet-remediation.md`:

1. Split research into independent lanes: Hermes semantics, rubric/wording quality, fleet/examples, and deliverable/design.
2. Consolidate lane results into durable class-level guidance rather than a one-session narrative.
3. Produce a self-contained static HTML review surface when the user asks for something to view over Tailscale: embedded CSS, navigation, verdict cards, score tables, before/after examples, and source-basis notes.
4. Keep durable source copies in the corpus (usually `docs/research/`) and convenience/download copies in the active profile cache when needed.
5. Do not claim a Tailscale/local review URL works until the server or `tailscale serve` path has been verified live.

## Live SOUL remediation after grading

When a grading session turns into live profile edits, keep the remediation class-level and evidence-safe:

1. Re-read the relevant roster/manifest and verify the live profile/workspace identity paths before editing. A profile `SOUL.md` may be a real file, symlink, or stale unrostered stub.
2. Back up every target identity file before edits, including both profile and workspace copies when they are intentionally duplicated.
3. Move volatile workflow, startup, provider, memory-service, and runbook detail out of `SOUL.md` into `AGENTS.md`, skills, manifests, or ops docs. Leave `SOUL.md` as the compact identity/authority layer.
4. If a workspace `AGENTS.md` exists but the profile lacks one, wire the profile `AGENTS.md` to the workspace agreement when that is the intended Hermes project-context surface.
5. For profile stubs/anomalies, archive rather than delete when there is any state/history worth preserving, and verify no gateway/service expects the profile before calling it removed.
6. For surgical post-grade patches on a remote/deployed agent, prefer this proof chain before reporting done: fetch/read the live `SOUL.md`; create a timestamped backup outside the committed workspace if possible; apply only the approved identity/authority wording; run a targeted secret scan on the edited file; commit the workspace identity change if the workspace is git-backed; run a fresh/new-session exact-marker smoke that exercises the new rule; verify affected services remain healthy when the edit was on a live profile; then record a concise manifest/ops note with backup path, commit, smoke session/marker, and no secret values.
7. Do not over-edit a strong SOUL just because a grade found small gaps. If the grade says “approved with fixes,” keep the patch to those behavior-changing gaps; avoid turning SOUL into a runbook, manifest, or service-health log.
8. If Hermes command-safety approval times out or blocks a destructive remote batch, stop and wait for the user. Do not retry, rephrase, split, or route around the blocked operation; resume only after the user explicitly re-approves the same scoped action.

For a production handoff, also use `references/live-soul-upgrade-production-handoff.md`. Use a user-supplied target score, or ask for one when absent; iterate the SOUL score until that target is met or the user accepts the current score. Report system blockers separately rather than forcing operating controls into SOUL. For a stale phase-coupled identity, use `references/phase-coupled-soul-refresh.md` and keep companion policy changes outside the numeric SOUL grade.

## Common pitfalls

1. **Grading from vibes.** Use the bundled references, not generic prompt advice.
2. **Rewarding length.** A long SOUL can still be weak if it lacks mission, identity, positive boundaries, judgment, and a truthfulness posture.
3. **Punishing missing runbooks.** SOUL should not contain every command. Missing commands may belong in `CLAUDE.md`, `AGENTS.md`, skills, or manifests.
4. **Applying scope before the rubric.** Score identity first. Use scope afterward to recommend checks in `AGENTS.md` / `CLAUDE.md` or system controls.
5. **Treating YAML as hidden.** Hermes native SOUL loading injects SOUL content as prompt text; frontmatter is visible unless an adapter strips/uses it before Hermes sees it.
6. **Forgetting session cache.** A corrected SOUL may not affect an existing Hermes session until a new session/restart/compression rebuild.
7. **Over-rewriting personality.** Keep distinctive voice only when it serves mission and does not leak into public/client contexts improperly.
8. **Pulling system policy into SOUL.** Keep the identity-level truth posture in SOUL and exact claim thresholds in the companion operating agreement.

## Verification checklist

- [ ] Loaded `references/soul-md-grading-standard.md` before grading.
- [ ] Read the target SOUL text/file actually being graded.
- [ ] Scored all 11 rubric categories.
- [ ] Awarded placement credit rather than duplication credit.
- [ ] Classified scope after scoring and applied it as an unscored overlay.
- [ ] Checked system blockers only from direct system evidence, or marked them not assessed.
- [ ] Separated SOUL problems from CLAUDE/AGENTS/skill/manifest placement problems and labeled suggestion destinations.
- [ ] Cited only bundled reference artifacts as normative sources.
- [ ] If a rewrite was provided, it preserves real agent facts and does not invent access, authority, credentials, or deployment state.
