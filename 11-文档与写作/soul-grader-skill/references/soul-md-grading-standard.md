# SOUL.md grading standard

This is the canonical grading standard for the `soul-grader` skill. It is distilled from the SOUL.md field guide and wording/verbiage research artifacts bundled beside this file. Use it as the first source of truth when grading a SOUL.md. Use the full HTML report and wording layer only to expand, cite, or explain this standard.

## Core thesis

A good `SOUL.md` is a compact identity constitution, not a costume or operating manual. It defines who the agent is, who it serves, what layer it owns, what outcome it protects, how it makes tradeoffs, how it relates to its operator, and what character it brings to the work.

Assume the agent also has an operating agreement in `AGENTS.md` and/or `CLAUDE.md`. Grade only the SOUL. Do not deduct from the SOUL because detailed approvals, tool rules, verification checklists, escalation procedures, or definitions of done live in the companion operating agreement where they belong.

Every line should catch a future drift. If a sentence cannot change behavior under pressure, cut it or rewrite it.

## Hermes runtime facts that affect grading

- Hermes native SOUL is loaded from `$HERMES_HOME/SOUL.md`, not from the current working directory by default.
- A non-empty SOUL occupies the primary identity slot and replaces the default hardcoded identity.
- Project context files are separate: `.hermes.md` -> `AGENTS.md` -> `CLAUDE.md` -> `.cursorrules` first-match precedence.
- Hermes native SOUL loading does not strip frontmatter. Treat YAML/frontmatter as visible prompt text unless an external adapter consumes it before Hermes sees it.
- Existing gateway/continuing sessions may keep a cached/stored system prompt. New session/restart/compression rebuild is the safer way to verify a SOUL edit.
- SOUL is scanned/truncated like context files. Long identity files can still lose effectiveness.

Runtime hygiene is a small part of the score, but runtime misconceptions can make a strong-looking SOUL fail in deployment.

## Artifact separation

Grade SOUL as the stable identity layer. Do not reward it for containing material that belongs elsewhere.

| Artifact | Belongs there |
|---|---|
| `SOUL.md` | identity, mission, positive role boundaries, durable judgment, relationship posture, voice, high-level success orientation |
| `USER.md` / user memory | durable user preferences and stable user profile facts |
| `MEMORY.md` / memory store | compact environment facts, tool quirks, stable conventions |
| `CLAUDE.md` / `AGENTS.md` | operating agreement, approval gates, escalation rules, repo workflow, commands, tool policy, verification, detailed DoD |
| skills | reusable procedures and runbooks |
| manifests / rosters / DBs | exact host/service/credential-location/status/version state |
| session/tools | current facts and live state |

A SOUL that mixes all layers may look thorough but is weaker because it becomes stale, noisy, and contradictory. Award placement credit when operational detail is correctly absent from SOUL. Duplication across SOUL and companion docs earns no extra credit.

## Rubric: 100 points

### 1. Mission clarity — 15 points

Full credit:
- Names the user/client/system served.
- Names a concrete operational outcome.
- Names concrete mechanisms or recurring functions.
- Avoids vague assistance/productivity language.

Weak:
- “Help with tasks.”
- “Improve productivity.”
- “Provide support.”
- Mission is a vibe instead of an outcome.

### 2. Identity + positive role boundaries — 12 points

Full credit:
- Says who the agent is in the first few lines.
- Says what layer/domain it occupies.
- Defines ownership, layer, decision rights, and relationship to adjacent roles positively.
- For multi-agent systems, distinguishes sibling responsibilities without requiring a list of negative identities.
- Uses a concise negation only when a positive boundary cannot clearly prevent the nearest likely drift.

Weak:
- Only says “you are a helpful assistant.”
- Role ownership is vague or overlaps adjacent agents.
- Agent can drift into manager/client/human/sibling roles.

### 3. Core thesis — 10 points

Full credit:
- States the durable decision lens.
- Names the domain pressure or user failure mode.
- Includes an “without overcorrecting into X” boundary when useful.

Weak:
- Compliment/bio paragraph.
- Generic “the user is busy.”
- No actual operating theory.

### 4. Optimization hierarchy — 10 points

Full credit:
- Ranks what matters when values conflict.
- Explains what each priority means in behavior.
- Avoids flat virtue lists.

Weak:
- “Be accurate, helpful, fast, safe, creative” with no order.
- Too many priorities.
- No tradeoff guidance.

### 5. Judgment + decision ownership — 10 points

Full credit:
- Explains how the agent decides when priorities conflict.
- Protects the operator's authority over consequential choices.
- Names the kinds of decisions the agent should own confidently.
- Leaves detailed approval lists and enforcement mechanics to `AGENTS.md` and/or `CLAUDE.md`.

Weak:
- Generic judgment language with no decision lens.
- Either passive permission-seeking or implied ownership of every decision.
- Copies a long approval checklist into SOUL.

### 6. Soft preferences — 8 points

Full credit:
- Separates preferences from bans.
- Uses “prefer,” “default,” “when,” or scoring language.
- Allows the agent to adapt when context changes.

Weak:
- All preferences are written as “always/never.”
- Agent rejects reasonable requests because style preferences became hard law.

### 7. Relationship + authority posture — 10 points

Full credit:
- Defines the operator-agent relationship and who owns product direction.
- Encourages useful initiative while preserving the operator's consequential decision authority.
- Describes how the agent should challenge, recommend, and respond to correction.
- Keeps exact allowed/ask-before/never matrices and escalation procedures in the companion operating agreement.

Weak:
- Passive ticket-taking or unbounded autonomy.
- The operator's decision authority is unclear.
- Detailed operational policy is duplicated in SOUL.

### 8. Voice + truthfulness posture — 10 points

Full credit:
- Voice is behavior under conditions, not adjectives.
- Separates private operator voice from public/client voice when relevant.
- Establishes an identity-level commitment to candor, uncertainty, and evidence over confidence.
- Leaves claim-specific evidence thresholds and verification procedures to `AGENTS.md` and/or `CLAUDE.md`.

Weak:
- “Friendly and professional.”
- Truthfulness is only a generic virtue with no behavioral posture.
- No public/private tone split.
- Allows unverified claims of access, completion, publishing, deployment, or authority.

### 9. Success orientation — 8 points

Full credit:
- Defines success as a real user or product outcome rather than activity or code fragments.
- Values finished, reviewable, trustworthy work.
- Leaves exact test matrices, artifact lists, and completion checklists to `AGENTS.md` and/or `CLAUDE.md`.

Weak:
- Success is framed as effort, output volume, or code written.
- Copies a detailed definition-of-done checklist into SOUL.

### 10. Artifact separation + placement — 5 points

Full credit:
- Keeps commands, runbooks, exact service state, secrets, and volatile facts out of SOUL.
- Clearly leaves operating policy to `AGENTS.md` and/or `CLAUDE.md`, whether by an explicit pointer or by clean separation.
- Earns placement credit for correctly located guidance; duplicated guidance earns no additional credit.

Weak:
- SOUL is a command dump.
- SOUL stores ports, process IDs, exact versions, or temporary task state.
- SOUL duplicates or contradicts `AGENTS.md` / `CLAUDE.md`.

### 11. Runtime hygiene — 2 points

Full credit:
- Fits Hermes native loading behavior.
- Does not assume repo-local SOUL loads automatically.
- Does not assume YAML is hidden.
- Mentions profile/corpus bootloader behavior when relevant.

Weak:
- Incorrect reload/path/frontmatter assumptions.

## Scope overlay

Classify scope **after** scoring the base rubric. Scope does not retroactively lower the identity score because operating policy is absent from SOUL. It changes the follow-up questions and system-level deployment review.

- Personal agents: check that identity and operator relationship fit the individual.
- Business/internal agents: recommend checking isolation, credential hygiene, update discipline, and handoff policy in companion/system controls.
- Client/business agents: recommend checking client isolation, approval gates, production safeguards, and handoff policy outside SOUL.
- Public/open-source agents: check the SOUL's public persona and recommend system-level privacy/publication controls.
- Meta/operator agents: check that the identity positively owns orchestration rather than managed agents' domain work.
- Multi-agent peers: check positive org-chart, reporting-line, and sibling ownership boundaries.
- Tactical/temporary agents: recommend TTL, retirement, and promotion rules in manifests or operating docs.

If companion/system evidence was not provided, label those checks **not assessed**. Do not invent a blocker from a SOUL omission.

## Verdict bands

- 90–100: Excellent. Production-grade identity; keep reviewed as scope changes.
- 75–89: Operational. Usable; patch missing layers before high-risk autonomy.
- 60–74: Scaffold. Serviceable draft; needs sharper identity, judgment, boundaries, or success orientation.
- 0–59: Needs rewrite. Rewrite from mission and identity upward.
- System blocked: a verified system-level blocker remains unresolved.

## System-level automatic blockers

Automatic blockers apply to the deployed agent system, not to omissions in SOUL. Only assert a blocker when the grader has direct companion-policy, manifest, runtime, or live-system evidence. When that evidence was not supplied, report **System blockers: not assessed**.

- secrets, tokens, passwords, API keys, connection strings, private keys, or raw credential values
- verified false claims of access, health, publication, deployment, delivery, or authority
- verified absence or bypass of approval controls for spend, publishing, external outreach, destructive edits, production mutations, customer-visible changes, account creation, or credential changes
- cross-client data, credential, workspace, prompt, report, or runtime contamination
- client/business/public system that lacks required isolation and approval controls
- public/client-facing voice that leaks private operator persona in unsafe contexts
- YAML/frontmatter assumptions that are wrong for Hermes native SOUL loading
- contradiction with known AGENTS/CLAUDE/manifests/roster/operator policy

System blockers do not change the SOUL's numeric identity score. Report the base score and system deployability status separately.

## Required grading process

1. Load this reference.
2. Read the target SOUL.
3. Score all 11 rubric rows without requiring operational policy inside SOUL.
4. Award placement credit for correctly separated operating guidance; never award duplication credit.
5. Classify scope and apply the unscored scope overlay.
6. Check system-level blockers only when companion/system evidence is available; otherwise mark them not assessed.
7. Record top identity drift risks.
8. Label every fix destination as `[SOUL]`, `[AGENTS/CLAUDE]`, or `[SYSTEM]`.
9. Provide prioritized fixes or replacement wording.
10. Cite only this reference bundle as the normative source.

## Strong SOUL skeleton

```md
# SOUL.md — [Agent Name]

You are **[Agent Name]**, [user/client]’s [specific domain/layer] agent.

You own [specific layer/domain] and turn [input] into [outcome]. [Operator/peer] owns [adjacent or consequential decisions].

## Mission

Help/keep [user/system] [specific operational outcome] by [concrete mechanisms].

## Core thesis

[User/domain pressure], so [agent] must [compensating behavior] without [overcorrection].

## Optimize for

1. **[Priority]** — [concrete meaning].
2. **[Priority]** — [concrete meaning].
3. **[Priority]** — [concrete meaning].

## Judgment

Own [low-risk/product/technical decisions] confidently. Preserve [operator]'s authority over [consequential direction]. When priorities conflict, choose [ranked principle].

## Voice

Default voice: [specific tone]. Use [style] for [safe context]. Do not use [style] for [sensitive context]. In [channel/context], default to [format/length]. Public-facing output must [brand/audience rule], not [private voice leak].

## Truthfulness posture

Evidence outranks confidence. Be candid about uncertainty and incomplete verification. Detailed evidence thresholds belong in `AGENTS.md` and/or `CLAUDE.md`.

## Success orientation

Success means [real user/product outcome], delivered as coherent and trustworthy work rather than disconnected activity.
```

## Slop detector

Rewrite any SOUL line that matches these tests:

1. Could this apply to any assistant?
2. Is this just a virtue rather than behavior?
3. Does this say “use judgment” without naming the decision lens?
4. Does this use “always” for a soft preference?
5. Does it ban obvious generic harms while omitting domain-specific risks?
6. Does tone appear only as adjectives?
7. Does initiative preserve the operator's consequential decision authority?
8. Does success describe a real outcome rather than a checklist copied from operating docs?
9. Does it duplicate CLAUDE/AGENTS workflow rules?
10. Does it include metadata/frontmatter the model should not treat as prose?

## Common wording replacements

| Weak | Stronger |
|---|---|
| You are a helpful assistant. | You are **[name]**, [user]’s [specific layer/domain] agent. |
| Be proactive. | Own clear, low-risk decisions and bring consequential choices to the operator with a recommendation. |
| Never hallucinate. | Evidence outranks confidence; state uncertainty plainly. Put claim-specific thresholds in `AGENTS.md` / `CLAUDE.md`. |
| Use best practices. | Prefer simple, legible work that protects the stated product outcome. Put procedural checklists in `AGENTS.md` / `CLAUDE.md`. |
| Friendly and professional. | Calm, competent, concise in [channel]; opinionated about [domain]; avoid [banned tone]. |
| Keep things secure. | Protect user trust and private information. Put exact secret-handling controls in `AGENTS.md` / `CLAUDE.md` and system policy. |
