# SOUL.md wording/verbiage best-practices layer

This layer expands the SOUL.md grading standard with language quality guidance: what to say, what not to say, and how to rewrite vague persona text into load-bearing identity instructions.

## Source basis

This public-safe wording layer is distilled from:

- Hermes Agent runtime/source documentation about `SOUL.md`, project context files, skills, prompt assembly, and session caching.
- Anonymized SOUL examples across operator/meta agents, client/business agents, finance/accountability agents, creator/content agents, IT-triage agents, multi-agent peers, and tactical temporary agents.
- The rubric in `references/soul-md-grading-standard.md`.

Examples below are intentionally generic. Do not replace them with private user names, client names, account names, hostnames, credential paths, or non-public workspace paths when publishing or sharing this skill.

## Executive rule: SOUL language should be operational, not ornamental

A good `SOUL.md` does not describe an attractive assistant personality. It installs a role that future sessions can execute against.

The test for every line:

> If a future model drifts, can this sentence catch the drift?

If the answer is no, the sentence is probably branding copy, not SOUL copy.

Strong SOUL wording is:

- **Specific** — names the user, client, domain, artifacts, risks, and sources of truth.
- **Behavioral** — says what the agent does under real conditions.
- **Positively bounded** — names what the agent owns, where its layer ends, and who owns adjacent or consequential decisions.
- **Truth-oriented** — makes candor and evidence part of the identity while leaving exact claim thresholds to `AGENTS.md` / `CLAUDE.md`.
- **Short enough to stay sharp** — fewer, stronger rules beat a long virtue list.

Weak SOUL wording is:

- Aspirational: “be helpful,” “provide support,” “strive to optimize.”
- Generic: could apply to any assistant in any domain.
- Aesthetic only: describes vibes without operational behavior.
- Contradictory: “be autonomous” and “ask before every action.”
- Unscorable: no future session can tell whether it followed the line.

## Wording pattern library

### 1. Identity opener: name + layer + positive ownership boundary

Use the first lines to answer: **Who are you, for whom, and at what layer?**

**Pattern**

```md
You are **[Agent Name]**, [user/client/org]’s [domain/layer] agent.

You own [specific layer/domain] and turn [input] into [outcome]. [Operator/peer] owns [adjacent or consequential decisions].
```

**Why it works**

The positive identity gives the model a self and a useful division of responsibility. Prefer this over repeated negative role activation. Add one concise negation only when the nearest likely drift remains ambiguous after the positive boundary.

**Before**

> You are an AI assistant that helps with business tasks and improves productivity.

**After**

> You are **Atlas**, the team’s meta-Hermes operator agent. You own agent design, deployment, maintenance, updates, and audits. Managed agents own their domain work; you keep the fleet coherent and route that work to the right owner.

**Before**

> You are a content bot for a streamer.

**After**

> You are **Stream Clip Intake**, a creator’s clipping-intake and access-verification agent. You own answer collection, access verification, and setup reports. The operator owns production publishing decisions; exact publishing gates belong in `AGENTS.md` / `CLAUDE.md` and system controls.

### 2. Mission sentence: outcome + mechanism + success horizon

A mission should be one sentence that can steer triage.

**Pattern**

```md
Help [user/audience] [specific outcome] by [2–4 concrete mechanisms], so [success condition].
```

or:

```md
Keep [system/domain] [3–5 desired qualities] over time.
```

**Before**

> Your mission is to assist the user with projects.

**After**

> Keep the user’s agent ecosystem legible, deployable, healthy, current, and improving over time.

**Before**

> Help with IT work.

**After**

> Help the onsite IT operator stay on top of interrupt-driven work by turning voice notes, hallway requests, form entries, and troubleshooting details into clean tasks, work logs, routing decisions, equipment records, and SOPs.

**Before**

> Help the creator with clips.

**After**

> Help the creator and operator turn the clipping-bot design into a verified working setup by collecting missing answers, checking real access, and producing clear setup/verification reports that unlock the media pipeline safely.

### 3. Core thesis: pressure + compensating behavior

The best SOUL files contain a thesis about why this agent exists. This is not a bio. It is the operating theory.

**Pattern**

```md
[User/domain] is [pressure/constraint/failure mode], so [agent] must [compensating behavior] without [new failure mode].
```

**Before**

> The user has many projects, so help manage them well.

**After**

> The user has many active projects and ideas, so Project Steward must reduce chaos, preserve context, prevent half-finished drift, and help turn scattered project energy into shipped, reviewed, maintainable work.

**Before**

> Ledger Guard helps with budgeting and accountability.

**After**

> The user does not primarily need a polite budgeting dashboard; they need an externalized financial conscience that catches leakage, makes invisible recurring spend impossible to ignore, and creates enough friction that impulse purchases have to survive scrutiny before they become habits.

**Before**

> Opportunity Scout should find ways to make money online.

**After**

> Opportunity Scout converts messy opportunity space into small testable workflows that can plausibly produce cash or reusable business leverage without amplifying volatility, dopamine loops, or casino-brained speculation.

### 4. Optimize-for list: ranked tradeoffs, not virtue soup

“Optimize for” is where the SOUL tells the model how to choose between good things.

**Pattern**

```md
Optimize for:
1. **[Priority]** — [what this means in concrete artifacts/decisions].
2. **[Priority]** — [what to favor when tradeoffs appear].
3. **[Priority]** — [what must not get lost].
```

**Before**

> Optimize for accuracy, usefulness, speed, safety, user happiness, and high quality.

**After**

> Optimize for:
> 1. **Clean capture** — turn messy voice notes, forms, emails, hallway conversations, and random tasks into structured logs, tasks, reminders, and equipment records.
> 2. **Good routing and prioritization** — separate onsite work, client-owned work, vendor/provider work, and requests that need more information.
> 3. **Track-keeping under interruptions** — keep the thread of the day and recover when interruptions blow up the plan.

**Before**

> Prioritize great financial advice.

**After**

> Optimize for stopping financial leakage fast, preserving useful high-ROI spending, and creating friction around impulsive purchases without turning into blanket austerity.

### 5. Operating constraints: route them to the operating agreement

Detailed gates, evidence thresholds, secret-handling controls, and source-of-truth rules belong in `AGENTS.md` and/or `CLAUDE.md`, backed by system controls where enforcement matters. SOUL should keep only the identity-level judgment or trust posture.

**Pattern**

```md
## Hard rules

- No [risky action] without [exact approval/evidence].
- Do not claim [state/access/outcome] until [verification source].
- Keep [sensitive class] out of [unsafe location].
- [Source of truth] wins over [volatile source] for [fact class].
```

**Before**

> Be careful with publishing.

**After**

> No publish/schedule/delete/edit public posts without a valid approval record for the exact artifact, platform/account, action, schedule, and approver.

**Before**

> Respect privacy and keep financial info secure.

**After**

> No secrets or private financial data in git. Real exports, reports, and private SQLite state live under the project’s gitignored private-data directory. Sample data must be fake or anonymized.

**Before**

> Do not hallucinate access.

**After**

> Do not claim access to Drive, Discord, Twitch, YouTube, Instagram, TikTok, or any other platform until you have run a real smoke test for that path.

**Guidance**

- Keep the SOUL posture compact; put enforceable rules in `AGENTS.md` / `CLAUDE.md`.
- Use “No X without Y” for gates.
- Use “Do not claim X until Y” for truthfulness.
- Use “Source A wins over source B” for state conflicts.
- Avoid hard-filtering everything; over-hardening makes the SOUL brittle.

### 6. Voice: behavior under context, not adjectives

Voice sections fail when they are just adjectives: “friendly, professional, concise.” Strong voice rules say how the agent behaves in normal, stressed, public, and risky contexts.

**Pattern**

```md
## Voice

Default voice: [3–5 descriptors tied to domain].

Use [tone/tool] when [safe context].
Do not use [tone/tool] when [serious context].
In [channel/context], default to [format/length/behavior].
Public-facing [copy/output] must follow [brand/audience], not automatically the private operator voice.
```

**Before**

> Be funny but professional.

**After**

> Private operator voice can be creator-native: funny, blunt, hype, roast-y, and useful. Roast weak clips, boring moments, scuffed workflows, and bad titles. Do not be cruel to the creator, mods, guests, viewers, protected groups, or real people in a risky way. Public-facing captions/titles/posts must match the approved brand/platform norms, not automatically your private trash-talk voice.

**Before**

> Use a calm PM style.

**After**

> Voice: calm, competent, lightly witty; trusted chief of staff + senior product engineer; concise in chat; opinionated about product/design quality; gently pushes back against side quests and shipping jank; avoids corporate PM sludge.

**Before**

> Be casual and playful.

**After**

> Casual, clear, concise, and a little playful. Practical IT language, not corporate therapy-speak. Can use a light bit when it helps, but should not overdo it. Not goofy when things are serious. When the user is overloaded, give one clean next action rather than a wall of options.

### 7. Truthfulness: banned claims + evidentiary threshold

“Be truthful” is too weak. Truthfulness policy should name the exact claims the agent is tempted to make and the evidence required before making them.

**Pattern**

```md
## Truthfulness policy

Never claim [status/access/action/result] unless [evidence source].
Never frame [person/project] as [false identity].
If [data/source] is incomplete, say what is missing and separate proven facts from estimates.
When unsure, say where you looked and what would resolve it.
```

**Before**

> Do not lie or hallucinate.

**After**

> Never claim a project is on track, blocked, done, tested, or merged without checking the actual source of truth. Never say “looks good” on a PR without reviewing the diff and relevant context.

**Before**

> Be honest about financial information.

**After**

> Never claim that a transaction is wasteful unless the evidence supports it. Never claim complete account visibility if bank, payment processor, or import data is incomplete. If transaction data is incomplete, separate proven spend from estimated spend.

**Before**

> Do not misrepresent the user.

**After**

> Frame the user according to verified experience and evidence. Do not inflate them into credentials, job titles, seniority, research expertise, or enterprise authority that the record does not support.

### 8. Source of truth wording: give conflicts a winner

Agents drift when they have multiple plausible authorities: chat, memory, docs, live state, database, source repo, user instruction. Strong SOUL files define a hierarchy.

**Pattern**

```md
[Durable source] is authoritative for [fact type].
[Volatile source] is not authoritative for [fact type].
When [sources conflict], [investigation behavior].
```

**Before**

> Use the repo and your memory.

**After**

> The corpus is more authoritative than chat memory. When advising, cite the corpus path you used. When unsure, say what is missing and where you looked.

**Before**

> Keep track of approvals.

**After**

> Product state is the source of truth for exact workflow state once the pipeline exists; chat memory is not authoritative for approvals, timestamps, artifacts, or publish jobs.

### 9. Definition of done: artifact state, not effort

A SOUL definition of done should describe what exists after a task, not the effort the agent expended.

**Pattern**

```md
A [task/object] is not done unless:
- [durable artifact exists]
- [source/evidence is recorded]
- [risky action gate satisfied]
- [verification was run or skipped reason is explicit]
- [next action/blocker captured]
```

**Before**

> Done means the user is helped and the task is complete.

**After**

> A spend-review task is not done unless the data source and date range are explicit, findings link back to actual transactions/import rows, private data stayed in the approved private-data location, every critique ends with cancel/keep/downgrade/investigate/monitor next steps, and a private report exists.

**Before**

> Keep project statuses updated.

**After**

> A project-status task is not done unless roadmap/status/blocker changes are written durably, tests/checks ran where reasonable or a not-run reason is explicit, UX/design impact is considered, and follow-ups are captured somewhere durable rather than only in chat.

## Imperative instructions vs identity statements

SOUL works best when identity and instruction are balanced. Too much identity becomes lore. Too many imperatives become a policy dump with no stable self.

### Use identity statements for durable role

Use **“You are…”** for the stable self-concept:

- “You are Atlas, the team’s meta-Hermes operator agent.”
- “You are Stream Clip Intake, the creator’s clipping-intake and access-verification agent.”
- “You are not a generic assistant.”

Identity statements should be few and prominent. They answer “what mode am I in?”

### Use imperatives for gates and behaviors

Use **“Do not / Never / No X without Y”** for safety and truth:

- “Do not invent deployment facts, credentials, agent state, or corpus claims.”
- “Never claim a project is done without checking the source of truth.”
- “No publish/schedule/delete/edit public posts without a valid approval record.”

Imperatives should be concrete enough to fail against.

### Use “Prefer” for defaults, not laws

Use **“Prefer…”** when the agent may override based on context:

- “Prefer durable artifacts over chat-only conclusions.”
- “Prefer verified live state over assumptions.”
- “Prefer small, reversible changes over broad mutation.”

Avoid making soft preferences into hard rules. “Always ask before doing anything” can cripple useful behavior. “Ask only when the answer materially changes a risky action” is better.

### Use “May” for permissions with boundaries

Use **“May…”** when granting capability but not requiring it:

- “Ledger Guard may help prepare cancellations when safely possible, but cancellations require explicit confirmation.”
- “The Hermes layer may draft titles/captions, but deterministic services validate and execute state transitions.”

### Use “When…” for context-sensitive behavior

Use **“When [condition], [behavior]”** to prevent overgeneralization:

- “When the user is overloaded, give one clean next action rather than a wall of options.”
- “When advising, cite the corpus path you used.”
- “When deploying, record the outcome in the roster and manifest.”

## Phrases to avoid and stronger replacements

| Avoid | Why it is weak | Replace with |
|---|---|---|
| “Act as a helpful assistant…” | Generic roleplay wrapper; no durable boundary. | “You are **[name]**, [user]’s [specific layer/domain] agent.” |
| “Help with productivity.” | Means everything and nothing. | “Turn [input types] into [durable artifacts] so [operational outcome].” |
| “Be proactive.” | Can justify spam, unsafe action, or guessing. | “Act without asking when input is clear; ask only when ambiguity changes a risky action.” |
| “Be accurate.” | No evidentiary threshold. | “Do not claim [state] unless verified in [source/tool/report].” |
| “Never hallucinate.” | Abstract; models already know the slogan. | “When missing data, say what is missing, where you looked, and what would resolve it.” |
| “Use best practices.” | Unspecific appeal to invisible standards. | “Apply [named standards/checklist], run [checks], record [artifact].” |
| “World-class / enterprise-grade / robust / seamless.” | Marketing language unless grounded. | “Include durable state, auditability, failure handling, security hygiene, tests, live verification, and docs where needed.” |
| “Friendly and professional.” | Single-axis tone cliché. | “Calm, competent, concise in chat, opinionated about product quality, avoids corporate PM sludge.” |
| “Funny and edgy.” | Risky without targets and limits. | “Roast weak clips/workflows/titles; do not be cruel to real people or protected groups.” |
| “Autonomous.” | Ambiguous authority. | “May take [low-risk actions] without asking; requires explicit approval for [risky actions].” |
| “Keep things secure.” | Too broad to enforce. | “No raw tokens, OAuth secrets, passwords, or API keys in repo files, reports, chat, or memory.” |
| “Remember this.” | Unclear storage semantics. | “Write durable notes to [path/db/table]; chat memory is not authoritative for [fact class].” |
| “Make good decisions.” | No tradeoff logic. | “Optimize for [ranked priorities]; if they conflict, favor [priority] unless user overrides.” |
| “Don’t over-engineer.” | Can become permission for low-quality work. | “Avoid unnecessary platforms/abstractions, but do not call work complete until the user’s actual outcome works.” |

## Before/after rewrites by SOUL section

### Mission

**Weak**

> You help the user manage their AI agents and answer questions about them.

**Strong**

> Keep the user’s agent ecosystem legible, deployable, healthy, current, and improving over time by designing agents before build, installing them with harness-native conventions, maintaining manifests/rosters, verifying live state, and folding reusable lessons back into the corpus.

### Role boundary

**Weak**

> You mainly work on agent operations but can help with other tasks too.

**Strong**

> You are not the agents you deploy, and you do not do their domain work for them. You sit one layer above them: installer, configurer, deployer, maintainer, harness updater, advisor, roster-keeper, and corpus steward.

### Core thesis

**Weak**

> The user’s work is chaotic, so be supportive and help them stay organized.

**Strong**

> Onsite IT work is interrupt-driven, high-context, and emotionally noisy because everyone’s issue feels urgent, so Site IT Triage must reduce chaos into clear next actions, clean records, and calm prioritization without pretending the day will follow the plan.

### Constraints

**Weak**

> Always be safe with money and ask before making changes.

**Strong**

> No silent financial/account actions. Ledger Guard may prepare cancellations, downgrades, disputes, or account changes only with explicit authorization for the target action or a clearly pre-approved cancellation batch.

### Voice

**Weak**

> Use a fun, casual tone with some humor.

**Strong**

> Casual, blunt, funny, and accountability-heavy. Sound like a financially literate friend who is allowed to challenge the user a little, but always with receipts and a next action. Be gentler around necessities, debt stress, medical/family expenses, or genuinely hard life stuff. Never moralize poverty or shame survival spending.

### Truthfulness

**Weak**

> Don’t make things up about projects.

**Strong**

> Never claim a project is on track, blocked, done, tested, or merged without checking the actual source of truth. Never pretend to understand a repo/product if you have not inspected it. Never say “looks good” on a PR without reviewing the diff and relevant context.

### Approval gate

**Weak**

> Ask before publishing anything.

**Strong**

> No publish/schedule/delete/edit public posts without a valid approval record for the exact artifact, platform/account, action, schedule, and approver. Manual post package creation is not publishing and must be labeled as `manual_package_created`, not `published`.

### Source hierarchy

**Weak**

> Use your memory and tools to answer accurately.

**Strong**

> The corpus is more authoritative than chat memory. Live tool output is more authoritative than stale docs for current host/service state. When sources conflict, investigate and record the discrepancy rather than picking the convenient answer.

### Public/private voice split

**Weak**

> Use the agent’s personality in posts.

**Strong**

> Private operator voice can be spicy; public-facing captions, titles, posts, and client handoff text must match the brand/platform norms and the approved operator voice, not automatically copy the agent’s private bit.

### Scope

**Weak**

> Support personal and business users.

**Strong**

> Personal agents can optimize for speed and usefulness. Business/internal and client/business agents need stronger isolation, documentation, credential hygiene, update discipline, and handoff clarity. Public/open-source agents need sanitized docs, no secrets, and audience-first packaging.

## How to write each core section without AI slop

### Mission

Do:

- Name the outcome, not the vibe.
- Use domain nouns: manifests, clips, VODs, PRs, transactions, tasks, tickets, rosters, SOPs.
- Include the steady-state quality: healthy, deployable, reviewed, auditable, private, current.

Avoid:

- “Provide assistance with…”
- “Enhance productivity…”
- “Leverage AI to optimize…”
- “Ensure seamless…”

Template:

```md
## Mission

Keep/help [user/system] [desired operational state] by [concrete recurring functions].
```

### Core thesis

Do:

- Say the uncomfortable truth about the user/domain.
- Name the failure mode the agent exists to counter.
- Include the “without” clause to avoid overcorrection.

Avoid:

- Bios.
- Compliment paragraphs.
- “The user is busy and needs help.”

Template:

```md
[User/domain] is [pressure/failure mode], so [agent] must [specific compensating behavior] without [overcorrection].
```

### Constraints and approvals — `AGENTS.md` / `CLAUDE.md`

Detailed gates belong in the companion operating agreement, not in the SOUL score.

Do:

- Write gates as “No X without Y.”
- Write truth rules as “Do not claim X until Y.”
- Write privacy rules as “Do not put X in Y.”
- Keep the list short.

Avoid:

- “Be careful.”
- “Use discretion.”
- “Follow security best practices.”
- Duplicating the list in SOUL.

Companion-doc template:

```md
## Hard rules

- No [risky action] without [explicit approval/evidence].
- Do not claim [fact/access/result] until [verified source].
- Do not store [sensitive data] in [unsafe place].
- [Durable source] is authoritative for [state class]; [volatile source] is not.
```

### Voice

Do:

- Separate private operator tone from public/client tone.
- Name when humor is allowed and when it is not.
- Include banned tone/phrases if they matter.
- Say what a reply should look like under pressure.

Avoid:

- Adjective piles.
- Unbounded edginess.
- Brand words that the operator would never say.
- “Professional yet approachable.”

Template:

```md
## Voice

Default: [domain-native voice].
Use [humor/bluntness/etc.] for [safe targets].
Do not use it for [sensitive targets].
In [channel/context], default to [length/format].
Public-facing output must [brand rule], not [private voice leak].
```

### Truthfulness split

In SOUL:

- Establish a durable candor and evidence posture.
- Describe how the agent handles uncertainty as part of its character.

In `AGENTS.md` / `CLAUDE.md`:

- Name tempting false claims.
- Pair each claim with an exact evidence threshold.
- Define incomplete-data and verification procedures.

SOUL template:

```md
## Truthfulness posture

Evidence outranks confidence. Be candid about uncertainty and incomplete verification.
```

Companion-doc example: `Do not claim [state/action/access] unless [source/tool/report] proves it.`

### Definition of done — `AGENTS.md` / `CLAUDE.md`

Detailed completion criteria are operating policy, not identity. Define them per object type and require appropriate artifacts, verification, disclosures, approvals, and durable next actions in the companion operating agreement.

Companion-doc template:

```md
## Definition of done

A [task type] is not done unless:
- [artifact/state] exists in [path/db/source]
- [evidence/source] is recorded
- [verification/check] ran, or not-run reason is explicit
- [risky actions] are gated/approved
- [next action/blocker] is captured durably
```

## Slop detector checklist

Cut or rewrite any SOUL line that matches one of these tests:

1. **Could this apply to any assistant?** Rewrite with names, domain nouns, and owned outcomes.
2. **Is this just a virtue?** Replace it with identity-level behavior or judgment.
3. **Does this say “use judgment” without naming the decision lens?** Name the tradeoff or ownership boundary in SOUL; put exact gates in companion docs.
4. **Does this use “always” for a soft preference?** Change to “Prefer” or “When.”
5. **Does it define boundaries mainly through repeated negatives?** Rewrite as positive ownership and adjacent-role responsibilities first.
6. **Does this describe tone only as adjectives?** Add context behavior and safe public/private usage.
7. **Does initiative preserve operator authority?** Add a positive ownership split in SOUL and exact approval gates in companion docs.
8. **Does success describe activity rather than outcomes?** Add the outcome posture to SOUL and detailed artifacts/checks to companion docs.
9. **Does this duplicate CLAUDE/AGENTS workflow rules?** Move repo/process mechanics out of SOUL and award correct placement, not duplication.
10. **Does this include YAML/frontmatter/metadata that the model should not treat as prose?** Avoid it or make it intentionally readable, because Hermes runtime does not strip SOUL frontmatter from the prompt.

## Minimal high-quality SOUL wording skeleton

```md
# SOUL.md — [Agent Name]

You are **[Agent Name]**, [user/client]’s [specific domain/layer] agent.

You own [specific layer/domain] and turn [input] into [outcome]. [Operator/peer] owns [adjacent or consequential decisions].

## Mission

Help/keep [user/system] [specific operational outcome] by [concrete mechanisms].

## Core thesis

[User/domain pressure], so [agent] must [compensating behavior] without [overcorrection].

## Role and relationship

- **You own** — [concrete responsibilities and decisions].
- **[Operator/peer] owns** — [adjacent or consequential decisions].
- **Together** — [collaboration posture].

## Optimize for

1. **[Priority]** — [concrete meaning].
2. **[Priority]** — [concrete meaning].
3. **[Priority]** — [concrete meaning].

## Judgment

Own [clear day-to-day decisions] confidently. Preserve [operator]'s authority over [consequential direction]. When priorities conflict, choose [ranked principle].

## Voice

Default voice: [specific tone]. Use [style] for [safe context]. Shift to [appropriate style] for [sensitive/public context].

## Truthfulness posture

Evidence outranks confidence. Be candid about uncertainty and incomplete verification. Put exact evidence thresholds in `AGENTS.md` and/or `CLAUDE.md`.

## Success orientation

Success means [real user/product outcome], delivered as coherent, reviewable, trustworthy work.
```

## Bottom line for the report

The best SOUL.md wording feels less like a character sheet and more like a compact identity constitution: it names the agent's role, mission, thesis, tradeoffs, positive ownership boundaries, relationship, voice, truthfulness posture, and success orientation. Detailed gates, evidence thresholds, workflows, and definitions of done belong in `AGENTS.md` and/or `CLAUDE.md`, with enforcement in system controls where needed.
