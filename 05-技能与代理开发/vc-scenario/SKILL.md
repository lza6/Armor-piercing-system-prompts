---
name: vc-scenario
description: "Generate comprehensive edge cases and test scenarios by decomposing features across 12 dimensions. Use before implementation or testing to catch issues early."
argument-hint: "<file path or feature description>"
trigger_keywords: edge cases, test scenarios, what could go wrong
layer: helper
metadata:
  author: claudekit
  attribution: "Scenario exploration pattern adapted from autoresearch by Udit Goenka (MIT)"
  license: MIT
  version: "1.0.0"
---

# vc-scenario — Edge Case & Scenario Explorer

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Decompose any feature or code path across 12 dimensions to surface edge cases, risks, and test targets before implementation begins.

## Mode Selection

Choose a mode before generating scenarios. Default is Simple unless a trigger condition applies.

### Simple Mode (default)

Generates edge cases from the plan description, checklist item, or approach text provided in the prompt. No subagent spawned.

**Use when:**
- The checklist item is self-contained and clearly described
- Blast radius is narrow (1–2 files, single package)
- No auth, billing, schema, or external API surface is touched
- Speed matters and hypothetical coverage is sufficient

### Deep Mode

Spawns a research subagent to read the actual source before generating scenarios. Scenarios reference real variable names, real function signatures, and real failure modes visible in the code rather than hypothetical ones.

**Trigger conditions (any one):**
- Checklist item modifies an auth, billing, schema, or external API surface
- Blast radius spans 3+ files or 2+ packages
- The plan marks the item as `HIGH_RISK`
- Caller explicitly requests deep mode

**Deep mode subagent steps:**
1. Reads the actual source files being modified (from plan Touchpoints)
2. Locates and reads existing test files for those files (via grep for import paths or describe blocks)
3. Reads any Public Contracts affected (from plan's Public Contracts section)
4. Returns: real function signatures, actual data shapes, existing test coverage gaps, real failure modes visible in the code

The orchestrator then generates scenarios using the research output.

### Output quality difference

| Mode | Example scenario |
|------|-----------------|
| Simple | "What if the input is null?" |
| Deep | "What if `creditBalance.available` is 0 but `creditBalance.pending` is positive — does `deductCredits(amount)` check `available`-only or `available + pending`?" |

Simple mode surfaces generic edge cases quickly. Deep mode surfaces scenarios that are only discoverable by reading the actual implementation.

---

## When to Use

- Before implementing complex or stateful features
- Before writing tests (generates test targets)
- Risk assessment during planning or code review
- API design review — surface contract edge cases early

## When NOT to Use

- Trivial single-line changes or cosmetic UI tweaks
- Already well-tested, stable code with no recent modifications
- Pure configuration changes with no logic paths

---

## 12 Decomposition Dimensions

Not all 12 apply to every feature. Identify relevant dimensions first, then generate scenarios only for those.

| # | Dimension | What to Look For |
|---|-----------|------------------|
| 1 | **User Types** | admin, guest, banned, new user, power user, bot/scraper |
| 2 | **Input Extremes** | empty, null, max length, unicode, special chars, SQL/script injection |
| 3 | **Timing** | concurrent access, race conditions, timeout, slow network, retry storms |
| 4 | **Scale** | 0 items, 1 item, 1M items, pagination boundary, cursor wrap |
| 5 | **State Transitions** | first use, mid-flow abort, resume after crash, partial completion |
| 6 | **Environment** | mobile/low-end CPU, no JS, screen reader, proxy/VPN, different timezone/locale |
| 7 | **Error Cascades** | DB down, API timeout, disk full, OOM, network partition, partial write |
| 8 | **Authorization** | expired token, wrong role, shared/public link, CORS, CSRF, privilege escalation |
| 9 | **Data Integrity** | duplicate entries, orphan references, encoding mismatch, concurrent schema migration |
| 10 | **Integration** | webhook replay, API version mismatch, third-party outage, contract drift |
| 11 | **Compliance** | GDPR deletion request, audit logging gap, data retention, accidental PII exposure |
| 12 | **Business Logic** | edge pricing (zero/negative), coupon stacking, refund after partial delivery, free tier limits |

---

## Workflow

**Step 0 — Select mode** using the Mode Selection rules above.

**Simple mode:**

1. **Parse** feature description or checklist item from the prompt
2. **Filter dimensions** — mark which of the 12 apply; skip irrelevant ones explicitly
3. **Generate 3–5 scenarios** per relevant dimension
4. **Categorize severity** — Critical / High / Medium / Low
5. **Output** as structured table (see format below)
6. **Summarize** total scenario count by severity

**Deep mode:**

1. **Spawn research subagent** — pass Touchpoints, Public Contracts, and checklist item text
2. **Subagent returns** real function signatures, data shapes, coverage gaps, visible failure modes
3. **Filter dimensions** using research output to remove inapplicable ones
4. **Generate 3–5 scenarios** per relevant dimension, referencing actual variable/function names
5. **Categorize severity** — Critical / High / Medium / Low
6. **Output** as structured table annotated with source evidence (file + line where relevant)
7. **Summarize** total scenario count by severity

### Severity Criteria

| Level | Meaning |
|-------|---------|
| **Critical** | Data loss, security breach, auth bypass, silent corruption |
| **High** | Feature broken for a subset of users, data inconsistency |
| **Medium** | Degraded UX, recoverable error not surfaced to user |
| **Low** | Minor visual glitch, non-blocking warning |

---

## Output Format

```
## Scenario Report: [target]

Dimensions analyzed: [list]
Dimensions skipped: [list + reason]

| # | Dimension | Scenario | Severity | Expected Behavior |
|---|-----------|----------|----------|-------------------|
| 1 | Input Extremes | Empty string for required name field | High | Return 400 with field error |
| 2 | Authorization | Expired JWT accessing protected route | Critical | Redirect to login, invalidate session |
| 3 | Timing | Two users submit same form simultaneously | High | Idempotency key or conflict error |

### Summary
- Critical: N
- High: N
- Medium: N
- Low: N
- Total: N scenarios across X dimensions
```

---

## Integration with Other Skills

| Next Step | Skill | How |
|-----------|-------|-----|
| Generate test cases from scenarios | `vc-test` | Pass scenario table as input context |
| Inform implementation plan risks | `generate-plan` / `plan-agent` | Paste Critical/High rows into risk assessment |
| Deep persona debate on top risks | `vc-predict` | Feed Critical scenarios as the change proposal |

---

## Example Invocations

```
# Simple mode (default — self-contained, narrow blast radius)
/vc-scenario src/api/payment.ts
/vc-scenario "User registration with OAuth providers"
/vc-scenario src/middleware/auth.ts

# Deep mode (auto-triggered: billing surface, 3+ files)
/vc-scenario "Deduct credits on model usage — touches CreditBalance, CreditTransaction, usage-sync.ts"

# Deep mode (explicit request)
/vc-scenario --deep "Add multi-tenancy to the database layer"
```
