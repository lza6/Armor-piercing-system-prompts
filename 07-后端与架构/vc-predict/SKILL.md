---
name: vc-predict
description: "5 expert personas debate proposed changes before implementation. Catches architectural, security, performance, and UX issues early. Use before major features or risky changes."
argument-hint: "<feature description or change proposal> [--files <glob>]"
trigger_keywords: risks, predict issues, architectural review
layer: helper
metadata:
  author: claudekit
  attribution: "Multi-persona prediction pattern adapted from autoresearch by Udit Goenka (MIT)"
  license: MIT
  version: "1.0.0"
---

# vc-predict — Multi-Persona Pre-Analysis

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Five expert personas independently analyze a proposed change, then debate conflicts to produce a consensus verdict before a single line of code is written.

## When to Use

- Before implementing a major or high-risk feature
- Before a significant refactor or architecture change
- Evaluating competing technical approaches
- Stress-testing assumptions in a proposed design

## When NOT to Use

- Trivial or low-risk changes (use `debugger` for bugs, `generate-plan` / `plan-agent` for already-decided tasks)
- Already-approved work with no open design questions
- Pure dependency upgrades with no API changes

---

## Mode Selection

`vc-predict` runs in **Simple** or **Deep** mode. Choose based on the conditions below.

| | Simple | Deep |
|---|---|---|
| Context source | Approach description already in context | Approach description + historical research subagent |
| Subagent spawned | No | Yes — reads git log, prior reports, test failure history |
| Persona debate quality | Reasons from first principles | "We tried this 3 months ago and hit X" |
| When to use | Contained feature, clear scope, no prior attempts | See trigger conditions below |

### Deep Mode — Trigger Conditions (any one is sufficient)

- The approach involves a pattern previously attempted in this codebase (git history may show prior attempts)
- The approach touches a surface with known failure history: auth, billing, container lifecycle, streaming, or WebSocket reconnect
- Caller explicitly requests deep mode (`--deep` flag or "use deep predict")
- The plan is **COMPLEX** shape and this is the pre-checklist predict call

### Simple Mode — Trigger Conditions (default)

- Approach is a contained feature with clear scope
- No prior attempts at this surface area are likely
- Plan is SIMPLE shape and the design is not controversial

---

## Deep Mode — Research Subagent Protocol

Before the 5-persona debate, spawn a research subagent that performs the following steps in order:

1. **Git history scan** — run `git log --oneline --all -- [relevant files]` for each file or directory the approach touches. Flag any commits within the last 6 months that suggest a prior attempt, revert, or known fix.
2. **Prior phase reports** — search task folders under `process/features/{feature}/active/` and `process/features/{feature}/completed/` for `_REPORT_` files mentioning this surface (pattern: `active/{slug}_{date}/{slug}_REPORT_{date}.md`). Read any that contain keywords from the approach (e.g. "streaming", "SSE", "auth", "billing", "container lifecycle"). Legacy sibling `reports/` dir may also exist — scan it too if present.
3. **Test failure history** — grep test output files and CI logs for `FAIL` or `Error` patterns on relevant module names. Note recurring failures.
4. **Return a Historical Context block** containing:
   - What was tried before (commit refs, dates, brief description)
   - What failed and why
   - Current state of the surface (is it stable, under active churn, recently refactored?)
   - Known landmines (specific lines, patterns, or edge cases that broke before)

The 5 personas then receive this Historical Context block before their independent analysis phase. Each persona incorporates it when relevant.

### Output Quality Difference

**Simple predict** (no research):
> "Senior dev: this streaming approach could cause a memory leak in the SSE proxy."

**Deep predict** (with historical research):
> "Senior dev: we tried this exact streaming approach in commit `a3f921` (2026-03-15) — it caused a memory leak in the SSE proxy because the Bun response body was never released on client disconnect. The current proposal has the same pattern in `packages/api/src/routes/gateway-proxy.ts` line 84. The fix at the time was adding an `AbortController` listener; verify that is still present or re-apply."

---

## The 5 Personas

| Persona | Focus | Core Questions |
|---------|-------|----------------|
| **Architect** | System design, scalability, coupling | Does this fit the architecture? Will it scale? What new coupling does it introduce? |
| **Security** | Attack surface, data protection, auth | What can be abused? Where is data exposed? Are auth boundaries respected? |
| **Performance** | Latency, memory, queries, bundle size | What is the latency impact? N+1 queries? Memory leaks? Bundle bloat? |
| **UX** | User experience, accessibility, error states | Is this intuitive? What does the error state look like? Accessible on mobile? |
| **Devil's Advocate** | Hidden assumptions, simpler alternatives | Why not do nothing? What is the simplest alternative? Which assumption could be wrong? |

---

## Debate Protocol

1. **Read** the proposed change/feature description from the argument
2. **Read relevant code** if file paths are provided (grep for affected areas)
3. **Each persona analyzes independently** — do not let personas influence each other during this phase
4. **Identify agreements** — points where all (or 4+) personas align
5. **Identify conflicts** — points where personas meaningfully disagree
6. **Weigh tradeoffs** — for each conflict, evaluate which concern has higher impact
7. **Produce verdict** — GO / CAUTION / STOP with actionable recommendations

---

## Output Format

```
## Prediction Report: [proposal title]

## Verdict: GO | CAUTION | STOP

### Agreements (all personas align)
- [Point 1 — what they all agree on]
- [Point 2]

### Conflicts & Resolutions

| Topic | Architect | Security | Performance | UX | Devil's Advocate | Resolution |
|-------|-----------|----------|-------------|-----|-----------------|------------|
| [Issue] | [View] | [View] | [View] | [View] | [View] | [Recommendation] |

### Risk Summary

| Risk | Severity | Mitigation |
|------|----------|------------|
| [Risk description] | Critical/High/Medium/Low | [Concrete action] |

### Recommendations
1. [Action item — rationale]
2. [Action item — rationale]
3. [Action item — rationale]
```

---

## Verdict Levels

| Verdict | Meaning |
|---------|---------|
| **GO** | All personas aligned, no critical risks, proceed with confidence |
| **CAUTION** | Concerns exist but are manageable — mitigations identified, proceed carefully |
| **STOP** | Critical unresolved issue found — needs redesign or more information before proceeding |

### STOP Triggers (any one is sufficient)
- Security persona identifies auth bypass or data exposure with no viable mitigation
- Architect identifies fundamental design incompatibility requiring significant rework
- Performance persona identifies unacceptable latency or query explosion with no workaround
- Devil's Advocate exposes a false assumption that invalidates the entire approach

---

## Integration with Other Skills

| Workflow Step | Skill | How |
|---------------|-------|-----|
| Deepen risk scenarios | `vc-scenario` | Feed Risk Summary rows as feature description |
| Create implementation plan | `generate-plan` / `plan-agent` | Attach Recommendations as constraints to the canonical planning path |
| High-risk feature implementation | `execute-agent` | Reference CAUTION/STOP items as acceptance gates |

---

## Example Invocations

### Simple Mode (default)
```
/vc-predict "Add WebSocket support for real-time notifications"
/vc-predict "Migrate authentication from JWT to session cookies"
/vc-predict "Add multi-tenancy to the database layer"
/vc-predict "Replace REST API with GraphQL" --files src/api/**/*.ts
```

### Deep Mode
```
/vc-predict "Rework SSE streaming for chat responses" --deep
/vc-predict "Change container lifecycle on instance stop" --deep --files packages/api/src/infra/**/*.ts
/vc-predict "Refactor billing credit deduction" --deep
```

Deep mode is also auto-triggered (no flag needed) when the plan is COMPLEX shape or the approach touches auth, billing, container lifecycle, or streaming surfaces.
