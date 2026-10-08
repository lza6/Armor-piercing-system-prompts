# vc-publish Reference

Detailed reference for the vc-publish skill.

## .vc-publish-config

Create this file in the project root to tell vc-publish where the kit repo checkout lives:

```json
{
  "kitRepoPath": "/Users/you/vibecode-pro-max-kit"
}
```

- The path must point to a git checkout of `github.com/withkynam/vibecode-pro-max-kit.git`
- The checkout must have `vc-manifest.json` at its root
- If missing, the skill asks the user for the path interactively
- Add `.vc-publish-config` to `.gitignore` -- it contains a local machine path

## Resolver-Driven Diffing

Both repos are resolved through the same `resolve-manifest.mjs` script:

```bash
# Kit repo file list
node <kitRepoPath>/resolve-manifest.mjs --root <kitRepoPath> --json

# Dev repo file list (uses same manifest patterns against dev repo tree)
node <kitRepoPath>/resolve-manifest.mjs --root <devRepoRoot> --json
```

The `--json` output provides all metadata needed for diffing:

```json
{
  "files": ["...sorted managed file paths..."],
  "kitOnly": ["...sorted kit-exclusive file paths..."],
  "merge": [".claude/settings.json"],
  "copyIfMissing": [],
  "strip": [],
  "symlinks": { ".agents/skills": "../.claude/skills" }
}
```

**Key change from v1.0:** The publish step no longer needs to update `managed`/`managedDirs` arrays in the manifest. Glob patterns (`include`/`exclude`/`kitOnly`) are stable -- new files are automatically included. Manifest edits at publish time are: (1) version bump, (2) `legacyDeletions` sync dev→kit (the deprecation ledger is a literal path array, NOT a glob -- it grows on every harness deprecation and must be reconciled every publish), and (3) conscious reconciliation of any other field flagged by the Step 7 drift report. Do NOT blanket-copy the dev manifest over the kit's -- the kit carries packaging-only deltas (test-file excludes, kit-only tooling) that must survive.

## CLAUDE.md / AGENTS.md Content Stripping

When publishing, CLAUDE.md and AGENTS.md must be harness-only.

### Sections to KEEP (harness methodology)

- Coding conventions (TS inference, result pattern, `as const`)
- RIPER-5 methodology (all mode descriptions)
- Orchestrator role and responsibilities
- Mode detection and auto-orchestration patterns
- Routing protocol (intent detection, context gathering, subagent routing)
- Phase transition rules
- Key principles (phase locking, safety, efficiency)
- Skill registry table (skills are harness-provided)
- Available agents table
- Quick start and troubleshooting sections
- Shared process folder structure description

### Sections to STRIP (project-specific)

- Technology stack details (frameworks, databases, versions)
- Feature list / "Current features" entries
- Project-specific context groups
- Hardcoded package manager (replace with generic)
- MCP server instructions (project-specific config)
- Project-specific routing rules
- Absolute paths (`/Users/...`)
- Product name references (the project's product name and repo/directory name)

### Strategy

Maintain harness-only CLAUDE.md and AGENTS.md directly in the kit repo as canonical versions. When publishing:

1. Compare methodology sections between dev repo and kit repo
2. If dev repo has newer methodology (new agents, skills, routing), apply to kit repo's harness-only version
3. Never blindly copy dev repo's CLAUDE.md/AGENTS.md to kit repo

## Error Handling

| Error | When | Action |
|-------|------|--------|
| `.vc-publish-config` missing | Step 1 | Ask user for kit repo path interactively |
| Kit repo path doesn't exist | Step 1 | Print error, stop |
| Kit repo has no `vc-manifest.json` | Step 2 | Print "not a valid kit repo", stop |
| Kit repo has uncommitted changes | Step 2 | Warn user, suggest stashing. Do not block. |
| Resolver script missing or fails | Step 3 | Print error, suggest checking Node >= 22, stop |
| No changes to publish | Step 5 | Report "nothing to publish", stop |
| Leak detection finds project content | Step 8 | Print matches, **stop before committing** |
| Git push fails | Step 10 | Print error, suggest checking access. Commit/tag are local -- user can retry. |
| Version tag already exists | Step 9 | Print error, suggest bumping to next version |

## Version Bump Semantics

| Bump | When | Examples |
|------|------|---------|
| **Patch** (2.1.0 -> 2.1.1) | Bug fixes, minor doc updates | Hook bugfix, skill doc clarification, prompt tweak |
| **Minor** (2.1.0 -> 2.2.0) | New features, additive changes | New skill, new agent, new protocol, new seed template |
| **Major** (2.1.0 -> 3.0.0) | Breaking changes | CLAUDE.md structure change, manifest schema change, removed skills |

## Leak Detection Patterns

Step 8 is a **resolved-set, two-check** gate. It scans the full shipped TEXT
surface (not just `CLAUDE.md`/`AGENTS.md`) so brand leaks in skill/agent prose and
dangling context-doc references are caught at publish time.

**Resolve the shipped set** via the kit resolver, then keep only TEXT surfaces:

```bash
node <kitRepoPath>/resolve-manifest.mjs --root <kitRepoPath> --json
```

Restrict the resolved `files` to:
- `.claude/skills/**` matching `*.md`, `*.cjs`, `*.mjs`, `*.py`, `*.js`, `*.json`
- `.claude/agents/**` matching `*.md`
- `.codex/**`
- `process/development-protocols/**`
- plus `CLAUDE.md`, `AGENTS.md`

Exclude binaries and `**/node_modules/**`.

### Check (a) -- product-name grep over the resolved text set

Scan for the product names in the grep below ONLY. `tRPC`/`Prisma` are **dropped**
from this skill-prose scan to avoid false positives in legitimate generic test
guidance; the hosted-database product name is **kept** (see the pattern):

```bash
grep -rIin "flowser\|CloakBrowser\|OpenClaw\|Supabase" <resolved-text-files>
```

Allowlist the Bucket-4 lines that MUST keep the literal to function (otherwise the
gate flags itself):

- `author: flowser` frontmatter (maintainer tag, not a project leak)
- the `isFlowserActivePlanPath` internal code identifier
- this skill's OWN scrub-grep pattern lines (the narrow grep block below)
- the new `validate-kit-portability.mjs` validator's own pattern strings
- the internal `session-init.cjs` "Flowser plan generation" comment

### Check (b) -- non-portable context-path grep

Any concrete backticked `process/context/...` file reference in the resolved text
set, MINUS the shipped/seeded survivors, is a dangling-link leak (it would dangle on
a clean install) → FAIL with file:line.

Survivors (allowed): `process/context/all-context.md`,
`process/context/tests/all-tests.md`. Portable directory references (e.g.
`process/context/tests/`) and the `process/context/...` placeholder are fine.

### Existing narrow CLAUDE.md/AGENTS.md grep (kept as-is)

This narrow grep stays on just those two files; `tRPC`/`Prisma` plus the
hosted-database product name all REMAIN here, as shown in the pattern below:

```bash
# Project-specific product names
grep -ri "flowser\|tRPC\|Prisma\|Supabase\|CloakBrowser\|OpenClaw" CLAUDE.md AGENTS.md

# Absolute paths
grep -r "/Users/" .

# Machine-specific config
grep -r "localhost:[0-9]" CLAUDE.md AGENTS.md

# Specific service URLs
grep -ri "supabase\.co\|vercel\.app\|clerk\.dev" CLAUDE.md AGENTS.md
```

The brand grep matches product names ONLY. It does NOT match `.ck.json`/`.ckignore`
-- those Phase-2 legacy-fallback literals are intentional and must NOT be flagged.
Do not add `ck`/`ckignore` to any leak grep.

If any check matches: print every match with file path and line number, revert the
kit worktree (`git -C <kitRepoPath> checkout .`), and STOP before commit/push. The
standing `validate-kit-portability.mjs` validator (run by `vc-audit-vc`) mirrors
checks (a) and (b) for between-release drift.

## Diff Summary Output Format

```
vc-publish diff: current repo -> kit repo (v2.1.0)
================================================

FILES:
  [modified]  .claude/agents/vc-execute-agent.md  (+8 -3)
  [modified]  .claude/hooks/lib/scout-checker.cjs  (+2 -1)
  [new]       .claude/skills/vc-new-skill/SKILL.md
  [new]       .claude/skills/vc-new-skill/references/new-skill.md
  [strip]     CLAUDE.md (needs content review)
  [strip]     AGENTS.md (needs content review)
  [unchanged] .claude/settings.json
  ... (350 more unchanged)

Total changes: 4 modified, 2 new, 0 removed

Publish v2.2.0? (patch/minor/major to change, abort to cancel)
```

## Publish Summary Output Format

```
vc-publish complete: v2.1.0 -> v2.2.0

Published to: github.com/withkynam/vibecode-pro-max-kit.git
Tag: v2.2.0
Commit: abc1234

Changes:
  2 managed files modified
  2 new files added
  0 files removed
  Manifest version bumped (2.1.0 -> 2.2.0)

Leak check: PASSED
```
