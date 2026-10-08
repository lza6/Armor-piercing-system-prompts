# Working on this repo

The skill is `SKILL.md` at the repo root. Everything else exists to ship it.

## What the skill is for

It runs while you draft, not after. The two tests near the top are the method. The 36 patterns are a checking pass for what slipped through, and treating them as the method produces the failure the skill was written against: polished sentences that should have been deleted.

## Rules for changing SKILL.md

**The pairs are subtraction.** In every Slop/Real pair, the Real half says only what the Slop half already contained. If a real rewrite would need a fact from the source, put that fact in the Slop half. Never let the Real half introduce a name, number, date, place, or claim the Slop half lacks. Eleven of the original seventeen pairs broke this rule and had to be repaired, so assume a new pair breaks it until you have checked.

**No em dashes or en dashes** anywhere in the repo, except the single deliberate example in the em dash pattern. `scripts/validate-package.py` enforces this.

**Frontmatter is limited to six fields**: `name`, `description`, `license`, `compatibility`, `metadata`, `allowed-tools`. These are the only fields the [Agent Skills spec](https://agentskills.io/specification) allows. Anything else is a hard error on claude.ai upload and through the Skills API, not a warning that gets ignored. Claude Code accepts extra fields, which is exactly why it is easy to add one and not notice.

**`name` must equal the directory name**, so it stays `no-ai-slop` and the repo stays `no-ai-slop`.

**Cross-reference patterns by name, not number.** Inserting a pattern renumbers the list and silently redirects every numeric pointer.

**Keep one version string** across `SKILL.md` frontmatter, `.claude-plugin/plugin.json`, and the README version line. The validator fails if they drift.

## When to split the file

Keep the pattern list in `SKILL.md` until it passes 50 patterns or the file passes 25KB. The spec recommends staying under 500 lines and roughly 5,000 tokens; the file currently sits at about 175 lines and 3.9k tokens. Past that threshold, move the list to `references/patterns.md` and load it during the checking pass only. Do not split earlier: a reference file only helps if the agent actually loads it, and the pattern list is the part used most mechanically.

## Before you push

```bash
python3 scripts/validate-package.py
npx --yes skills@1.5.20 add . --list
claude plugin validate .
```

CI runs all three on every push and pull request.
