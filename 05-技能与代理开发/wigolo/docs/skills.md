# Skills

Agent skills are small on-demand instruction packs — a `SKILL.md` (plus any assets) that a coding agent loads when the task matches, instead of carrying everything in its system prompt. wigolo ships skills that teach an agent to use its tools *well*: cache-before-search, keyword-array queries, domain scoping, when to reach for `research` vs `agent`, how to read evidence scores.

## The catalog — 11 packs

| Pack | Teaches |
| --- | --- |
| `wigolo` | The hub: tool selection across all ten tools, cache-first pattern, response-field literacy. |
| `wigolo-search` | Multi-query search, depth tiers, domain scoping, freshness filters. |
| `wigolo-fetch` | Clean-markdown fetching, sections, JS rendering, authenticated sessions. |
| `wigolo-crawl` | Site indexing strategies and cache warming. |
| `wigolo-cache` | Querying the local knowledge cache before the network. |
| `wigolo-extract` | Structured extraction modes and schema design. |
| `wigolo-find-similar` | Hybrid semantic discovery and cold-start handling. |
| `wigolo-research` | Research briefs, depth choice, reading gaps/cross-references. |
| `wigolo-agent` | Autonomous data gathering with schemas and budgets. |
| `wigolo-diff` | Comparing page versions. |
| `wigolo-watch` | Change monitoring and webhooks. |

## Install

```bash
wigolo skills add                          # all packs, detected agents, project scope
wigolo skills add wigolo-search wigolo-fetch --agent claude-code,cursor
wigolo skills add --global                 # user scope instead of project/cwd
wigolo skills list                         # install state per agent
wigolo skills remove --dry-run             # show the removal plan first
```

Flags: `--global` (user scope; default is the current project), `--agent <id,...>` (default: detected agents), `--dry-run` (print the plan, touch nothing), `--json`, `--force` (overwrite user-modified files / replace symlinks).

## Supported hosts

| Host | Project scope | Global scope |
| --- | --- | --- |
| `claude-code` | `.claude/skills/<pack>/` | `~/.claude/skills/<pack>/` |
| `codex` | `.agents/skills/<pack>/` | `~/.agents/skills/<pack>/` |
| `cursor` | `.agents/skills/<pack>/` | `~/.cursor/skills/<pack>/` |
| `gemini-cli` | `.agents/skills/<pack>/` | `~/.agents/skills/<pack>/` |
| `cline` | `.cline/skills/<pack>/` | `~/.cline/skills/<pack>/` |
| `windsurf` | `.windsurf/rules/wigolo.md` | fenced block in `~/.codeium/windsurf/memories/global_rules.md` |

Windsurf doesn't read skill directories, so it receives a single all-in-one digest (`wigolo-digest`) as a rules file at project scope, or an owned fenced block inside the shared global rules file — removal deletes exactly that block, nothing else.

## The receipts model

Every install is recorded in a receipt store (`~/.wigolo/skills/receipts.json`) with a per-file content hash. That buys you:

- **Idempotent re-adds** — running `skills add` again is a no-op for unchanged packs and a clean upgrade for changed ones.
- **Byte-exact installs** — files are written deterministically (LF line endings), so hashes are stable and drift is detectable.
- **Safe removal** — `skills remove` deletes only files wigolo wrote whose hashes still match. A file you edited is left alone (and named), unless you pass `--force`.
- **Adopt-and-upgrade** — packs already present from an older wigolo or from other skills tooling are recognized and adopted into the receipt store rather than duplicated or clobbered.

Receipts are claims, not authority: every recorded path is bounds-checked against the known target layout before any delete, so a hand-edited receipts file can't direct a deletion outside skill directories.

## Interop with the skills ecosystem

Packs are standard `SKILL.md`-format directories with `name`/`description` frontmatter — the same shape the broader npx `skills` ecosystem CLI manages, installed into the same per-agent directories. Both tools can manage the same tree: wigolo's adopt-and-upgrade path recognizes existing installs instead of fighting them, and packs installed by wigolo are visible to any tool that reads those directories.

[← Docs index](./README.md) · [Next: Plugins](./plugins.md)
