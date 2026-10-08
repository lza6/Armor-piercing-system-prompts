<p align="center">
  <img src="assets/readme/soul-grader-repo-card.png" alt="SOUL.md Grader Skill repository card" width="100%">
</p>

<h1 align="center">SOUL.md Grader Skill</h1>

<p align="center">
  <a href="SKILL.md"><img alt="Hermes Skill" src="https://img.shields.io/badge/Hermes-Skill-C69A4A?style=for-the-badge&labelColor=050507"></a>
  <a href="references/soul-md-grading-standard.md"><img alt="100 point rubric" src="https://img.shields.io/badge/SOUL.md-100pt%20Rubric-9B7CFF?style=for-the-badge&labelColor=050507"></a>
  <a href="references/soul-md-field-guide.html"><img alt="Field Guide" src="https://img.shields.io/badge/Field%20Guide-Included-69E6FF?style=for-the-badge&labelColor=050507"></a>
  <a href="LICENSE"><img alt="MIT License" src="https://img.shields.io/badge/License-MIT-F2EEDF?style=for-the-badge&labelColor=050507&color=F2EEDF"></a>
  <a href="https://x.com/cobi_bean"><img alt="@cobi_bean" src="https://img.shields.io/badge/%40cobi__bean-X-6FE7FF?style=for-the-badge&labelColor=050507"></a>
</p>

<p align="center">
  <a href="#install">Install</a> ·
  <a href="#use-it">Use it</a> ·
  <a href="#whats-included">What's included</a> ·
  <a href="#grading-model">Grading model</a> ·
  <a href="#public-safety">Public safety</a> ·
  <a href="https://hermes-agent.nousresearch.com/docs">Hermes docs</a>
</p>

A drop-in agent skill for grading, reviewing, and rewriting Hermes Agent
`SOUL.md` files with a research-backed rubric instead of vibes.

`SOUL.md` is the identity layer for a Hermes Agent profile: who the agent is,
who it serves, what it owns, how it makes tradeoffs, and what character it brings
to the work. The grader assumes the agent also has an operating agreement in
`AGENTS.md` and/or `CLAUDE.md`; it grades SOUL only and rewards a clean split.

> **Unofficial community skill.** This repo is not an official Nous Research or
> Hermes Agent release. It is a public-safe community skill by
> [cobi](https://github.com/cobibean) / [@cobi_bean](https://x.com/cobi_bean).

## Install

### Hermes Agent

Clone this repository into the active Hermes profile's skills directory.

For the default profile:

```bash
mkdir -p ~/.hermes/skills/community
git clone https://github.com/cobibean/soul-grader-skill.git \
  ~/.hermes/skills/community/soul-grader
```

For a named profile:

```bash
profile="my-profile"
mkdir -p ~/.hermes/profiles/$profile/skills/community
git clone https://github.com/cobibean/soul-grader-skill.git \
  ~/.hermes/profiles/$profile/skills/community/soul-grader
```

Then start a fresh Hermes session or reload skills in a running session/gateway:

```text
/reload-skills
```

Ask Hermes to load it directly when needed:

```text
Use the soul-grader skill to grade this SOUL.md.
```

### Codex / Claude / other Markdown skill-aware agents

Copy the whole repository into your agent's skill directory, keeping the bundled
`references/` folder next to `SKILL.md`:

```bash
mkdir -p ~/.codex/skills
git clone https://github.com/cobibean/soul-grader-skill.git \
  ~/.codex/skills/soul-grader
```

If your agent does not have a formal skill loader, paste the `SKILL.md`
instructions into your project or agent instructions and keep the `references/`
files available.

## Use it

Good prompts:

```text
Use soul-grader to grade agents/my-agent/SOUL.md and tell me whether it is deployable.
```

```text
Use soul-grader to compare these two SOUL.md drafts and recommend the stronger one.
```

```text
Use soul-grader to rewrite this weak SOUL.md into a production-ready identity file, but do not invent access, credentials, live services, or authority.
```

The skill tells the agent to load the bundled grading standard first:

```text
references/soul-md-grading-standard.md
```

Then it can use the full field guide and wording layer when it needs more detail
or examples.

## What it grades

The rubric scores a `SOUL.md` out of 100:

- **Mission clarity** — who/what the agent serves and what outcome matters
- **Identity + positive role boundaries** — what the agent owns and how adjacent responsibilities split
- **Core thesis** — the durable decision lens for the user/domain/problem
- **Optimization hierarchy** — ranked tradeoffs instead of virtue soup
- **Judgment + decision ownership** — tradeoff judgment and consequential operator authority
- **Soft preferences** — defaults that do not become brittle bans
- **Relationship + authority posture** — initiative, challenge, correction, and operator ownership
- **Voice + truthfulness posture** — distinctive tone plus candor and evidence orientation
- **Success orientation** — real user/product outcomes rather than activity
- **Artifact separation + placement** — correct placement earns credit; duplication does not
- **Runtime hygiene** — correct Hermes loading, session-cache, and frontmatter assumptions

System-level blockers include verified missing safety controls, false live-state
claims, cross-client contamination, wrong Hermes frontmatter assumptions, and
known policy contradictions. Missing operating policy in SOUL is not a blocker;
when companion/system evidence is unavailable, deployability is marked not assessed.

## What's included

```text
SKILL.md
references/
  soul-md-grading-standard.md
  soul-md-field-guide.html
  soul-md-wording-verbiage-layer.md
  fleet-soul-grading-workflow.md
  research-deliverable-and-fleet-remediation.md
  phase-coupled-soul-refresh.md
  live-soul-upgrade-production-handoff.md
agents/
  openai.yaml
scripts/
  validate_skill.py
assets/readme/
  soul-grader-repo-card.png
.github/workflows/
  validate.yml
```

## Grading model

The core idea:

> A good `SOUL.md` is a compact identity constitution, not a costume or operating manual.

Every line should catch a future drift. If a sentence cannot change behavior
under pressure, cut it or rewrite it.

The bundled standard intentionally keeps the source hierarchy narrow: the target
`SOUL.md` is evidence, but the bundled references are the normative source for
quality judgments. Scope is applied only after the base rubric, and suggested fixes
are routed to `[SOUL]`, `[AGENTS/CLAUDE]`, or `[SYSTEM]`. That keeps the grader from
importing random prompt-engineering advice, personal taste, or operating checklists
into the identity score.

## Public safety

The examples and references in this repository are intended to be public-safe:

- no API keys, tokens, passwords, private keys, or connection strings
- no private customer names, hostnames, live IPs, or credential locations
- no private transcripts or raw fleet deployment facts
- no claims that this is an official Hermes Agent release

Run the local validator before publishing changes:

```bash
python3 scripts/validate_skill.py
```

The validator checks frontmatter, required references, unexpected file layout,
and common secret/private-term patterns. It is a lightweight guard, not a
substitute for human review.

## Credits

Created by [cobi](https://github.com/cobibean) for practical Hermes Agent and
multi-agent operations work.

Find cobi on X/Twitter: [@cobi_bean](https://x.com/cobi_bean)

Hermes Agent is created by Nous Research:

- Docs: [hermes-agent.nousresearch.com/docs](https://hermes-agent.nousresearch.com/docs)
- GitHub: [NousResearch/hermes-agent](https://github.com/NousResearch/hermes-agent)
- X/Twitter: [@NousResearch](https://x.com/NousResearch)

## License

MIT. See [LICENSE](LICENSE).
