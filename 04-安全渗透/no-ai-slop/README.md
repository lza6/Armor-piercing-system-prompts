# no-ai-slop

Rewriting finished text fixes the sentences and leaves the empty paragraph underneath.

So this one runs while you draft. Two tests decide what goes in, applied to every sentence and every detail you are weighing. The 36 patterns are a checking pass for what slipped through, never the method.

One `SKILL.md` on the [Agent Skills](https://agentskills.io) standard, so it loads in Claude Code, Codex, Cursor, Copilot, Gemini CLI, and anything else that reads skills.

## Install

**Any agent**, through the [skills](https://skills.sh) CLI:

```bash
npx skills add doniyor117/no-ai-slop
```

**Claude Code**, as a plugin:

```
/plugin marketplace add doniyor117/no-ai-slop
/plugin install no-ai-slop@no-ai-slop
```

**By hand**, into your skills directory:

```bash
git clone https://github.com/doniyor117/no-ai-slop ~/.claude/skills/no-ai-slop
```

Use `.claude/skills/no-ai-slop` instead to scope it to one project.

**claude.ai web chat**: zip the repo folder and upload it under Settings, Capabilities, Skills. That path validates the frontmatter strictly and accepts only the six fields in the Agent Skills spec, which is why this skill declares nothing else.

## Usage

Invoke it directly and paste the text:

```
/no-ai-slop

[your text]
```

Or just ask, since the description triggers on most writing requests:

```
Rewrite this cover letter so it doesn't sound like AI wrote it
```

Point it at a file and it changes prose only, leaving code, data, frontmatter, and link targets alone:

```
Run no-ai-slop over docs/getting-started.md
```

To match your voice, give it a sample. A supplied sample overrides every style rule in the skill except two: it will still refuse to invent facts, and it will still refuse to use dashes.

## The two tests

These do most of the work, and they run on every sentence and on every detail you are deciding whether to include.

**1. Would this still be true of something else?** Swap in a different company, project, person, or product. If the sentence survives, it is filler.

> Slop: "This project demonstrates strong analytical skills and a passion for solving real problems."
>
> Real: "Set an 18-day delivery promise at a 3.44% late rate, against the platform's own 21 days at 4.5%."

**2. Could anyone do this, or does everyone already?** Details that are ordinary for the audience are noise, and listing them signals inexperience, because it shows you cannot tell what is impressive. A developer's CV should not list "used Git". A README should not explain what a terminal is.

The first test kills empty phrasing. The second kills true but pointless content, which is harder to catch, because nothing about the sentence looks wrong.

## The 36 patterns

Adapted from Wikipedia's ["Signs of AI writing"](https://en.wikipedia.org/wiki/Wikipedia:Signs_of_AI_writing), maintained by WikiProject AI Cleanup, which is the best catalogue that exists because editors built it from cleaning up real cases at volume.

### Content

| # | Pattern | Slop | Real |
|---|---------|------|------|
| 1 | Inflated importance | "marking a pivotal moment in the evolution of" | "part of a wider decentralisation of administrative functions" |
| 2 | Name-dropping as proof | "cited in four papers, and maintains a large following" | Keep the citation, cut the follower count |
| 3 | Shallow -ing analysis | "reflecting the community's deep connection to the land" | Delete the clause |
| 4 | Vague connection | "is closely associated with modern data practice" | "teams use it to schedule nightly jobs" |
| 5 | Sales language | "nestled in the breathtaking region, a vibrant hub" | "the town is in the Gonder region" |
| 6 | Vague sourcing | "experts believe it plays a crucial role" | Name the source or drop the claim |
| 7 | Challenges and outlook | "despite these challenges, it continues to thrive" | State the actual problems |

### Language

| # | Pattern | Slop | Real |
|---|---------|------|------|
| 8 | Machine vocabulary | delve, leverage, robust, seamless, testament, landscape | Rewrite the thought, not the word |
| 9 | Avoiding is and are | "serves as", "boasts", "features" | "is", "has" |
| 10 | Not X but Y | "it's not just a song, it's a statement" | "the song is a statement" |
| 11 | Forced threes | "sessions, discussions, and opportunities" | Use the number the meaning needs |
| 12 | Elegant variation | "the author... the writer... the novelist" | One name for one person |
| 13 | False ranges | "from the Big Bang to dark matter" | List the actual items |
| 14 | Missing actors | "no configuration file needed" | "you do not need a configuration file" |

### Style

| # | Pattern | Fix |
|---|---------|-----|
| 15 | Em dashes and en dashes | Comma, full stop, colon, or parentheses |
| 16 | Bold scattered through prose | Remove it |
| 17 | Lists with bold mini-headings | Write the prose |
| 18 | Title Case In Headings | Sentence case |
| 19 | Emojis as section markers | Remove them |
| 20 | Horizontal rules and decorative tables | Decoration standing in for structure |
| 21 | Curly quotes | Match the document |
| 22 | Hyphen pile-ups | Hyphenate before the noun, not after |
| 23 | Fake deeper truth | "at its core, this is really about trust" |
| 24 | Heading restated in the first sentence | Cut the sentence |
| 25 | Dramatic fragments | "It had no plan. No backup. Nothing." |
| 26 | Aphorisms | State the specific claim |
| 27 | Fake candour | "Honestly? It depends." |
| 28 | Objections nobody raised | Remove the defence, keep any real claim |
| 29 | Rejected fake alternatives | "one tempting approach would be X, but" |

### Chatbot residue

| # | Pattern | Example |
|---|---------|---------|
| 30 | Assistant sign-off | "I hope this helps" |
| 31 | Knowledge-limit hedges | "while available information is limited" |
| 32 | Agreeableness | "Great question." |
| 33 | Placeholder text left in | "[Your Name]", fatal in a cover letter |

### Filler and hedging

| # | Pattern | Example |
|---|---------|---------|
| 34 | Padded connectives | "in order to", "due to the fact that" |
| 35 | Stacked qualifiers | "may potentially somewhat help to" |
| 36 | Upbeat closers | "the possibilities are endless" |

## What not to flag

A checklist applied without judgement strips what made the writing good, so the skill carries a guard against its own patterns. Repetition used deliberately for rhythm stays. A formal word that is the correct word stays. A real objection, scope limit, or legal note stays. Quoted material is never rewritten. One short sentence for emphasis is fine, and so is a long sentence when the thought is long, because even mid-length cadence is itself a tell.

## Two rules that always win

**Never invent a fact.** Names, numbers, dates, quotes, and citations come from the source or from the user. If a sentence needs a detail that was not supplied, the skill asks for it or leaves a marked gap. Inventing a plausible number to make a sentence land is the worst failure available, worse than any amount of slop.

This applies to the skill's own examples. In every Slop/Real pair, the Real half says only what the Slop half already contained. De-slopping is subtraction.

**Never use an em dash or en dash.** A writing sample you supply overrides everything else in the skill, but not these two rules.

## How this differs from humanizer

[blader/humanizer](https://github.com/blader/humanizer) works from the same Wikipedia catalogue and is worth your time. Two differences:

It is a rewriter, so it runs after the damage. This skill is built for drafting, and says outright that the patterns are a checking pass and never the method.

It has nothing on detail selection. Cutting details that are true but ordinary for the audience is not a phrasing problem, and no rewriting pass catches it. That is the second test above, and neither humanizer nor the Wikipedia article has it, because an encyclopedia has no reason to.

## Version

- **1.0.0** first public release.

## License

MIT. The pattern catalogue is adapted from Wikipedia's "Signs of AI writing", which is available under CC BY-SA 4.0.
