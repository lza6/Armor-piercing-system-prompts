# Generate Context Reference

Generate and maintain the broad authoritative repository context file at `process/context/all-context.md`. Use `process/context/all-context.md` as the context router before selecting grouped docs to consult.

## Behavior Modes

- **Full Scan Mode**: use when `process/context/all-context.md` does not exist.
- **Delta Update Mode**: use when the file exists; preserve stable content and update changed sections.

## Required Output

Primary output (always produced):

- `process/context/all-context.md`

Per-group output (produced when groups are detected or passed by caller):

- `process/context/{group}/all-{group}.md` for each approved or detected group

Include in `all-context.md`:

- Scanned timestamp.
- Repo HEAD from `git rev-parse HEAD` if available.
- Mode: Full Scan or Delta Update.
- Changes since last update when in Delta mode.
- Open Questions when anything is ambiguous.
- References to source files used.

Do not rewrite grouped context docs that already exist and have good content; if they are stale, missing from the router, or poorly organized, report that `audit-context` should be run.

## Invocation Modes

Three modes govern how this skill runs. The caller or detection context determines which applies.

### Mode: `setup-delegation`

Triggered when vc-setup calls this skill and passes an `approved_groups` list (output of Round 1 Subagent C in vc-setup's STUDY phase).

- Do NOT run the Context Group Detection Table scan — the caller already did it.
- Use the `approved_groups` list exactly as provided.
- Write `all-context.md` (Subagent E responsibility in vc-setup remains — this mode writes per-group files only when called for that purpose; see vc-setup Round 2 delegation note).
- For each group in `approved_groups`, write `process/context/{group}/all-{group}.md` using the seed template (`_all-group-template.md.seed`) as the base structure.
- **Naming is exact:** the file MUST be named `all-{group}.md` where `{group}` is the directory name verbatim (e.g. the `tests` group is `tests/all-tests.md`, never `all-testing.md` or any synonym). Write exactly ONE `all-{group}.md` per group — do not create variant/synonym names alongside it.
- If a group file already exists, merge real content in rather than overwriting. Preserve existing sections that have good content.

### Mode: `standalone-full`

Triggered by direct invocation when no approved-groups list is provided and the caller wants full context regeneration.

- Run the Context Group Detection Table (see below) on the codebase to discover groups.
- Write `all-context.md` with full scan content.
- Write `all-{group}.md` for all detected groups.

### Mode: `delta`

Triggered when context already exists and the goal is to bring it up to date.

- Run the Context Group Detection Table on the codebase.
- For each detected group: create its `all-{group}.md` only if it does not already exist (skip existing files).
- Never delete existing group files — deletion is `vc-audit-context`'s domain.
- Warn (do not fail) on any group name detected that is not in the canonical detection table.
- Update `all-context.md` with delta content only; preserve unchanged sections.

**Delta-mode group-creation rules:**

1. For each group detected: check if `process/context/{group}/all-{group}.md` exists.
2. If absent: create it from `_all-group-template.md.seed` with real scanned content.
3. If present: skip (do not overwrite; emit a skip notice).
4. Never remove or rename existing group files — deletion is `vc-audit-context`'s domain.
5. Warn (do not fail) on any group detected whose name is not in the canonical detection table.

## Context Group Detection Table

Use this table to determine which context groups to create during `standalone-full` and `delta` modes (or during vc-setup Round 1 Subagent C scanning — the canonical detection table lives here).

| Project Signal | Context Group | all-*.md Content |
|---|---|---|
| Prisma/Drizzle/TypeORM/Sequelize + DB config | `database/` | Schema location, migration commands, client setup, key models |
| Docker/Dockerfile/docker-compose present | `container/` | Image structure, services, ports, build commands |
| Auth dependency (Clerk/NextAuth/Auth.js/Passport/Lucia) | `auth/` | Provider, config location, protected routes pattern |
| CI/CD config (.github/workflows, .circleci, .gitlab-ci) | `cicd/` | Pipeline stages, deployment targets, required secrets |
| Infrastructure code (terraform, pulumi, CDK, SST) | `infra/` | Provider, resource types, deployment commands |
| 3+ UI component directories or design system | `uxui/` | Component library, styling approach, design tokens |
| Workflow/queue system (BullMQ, Temporal, Inngest, etc.) | `workflows/` | Queue config, worker setup, job types |

**Rule:** Only create a group when the project has SUBSTANTIAL content for it — at minimum 2+ source files dedicated to that domain. A single config file is not enough.

## Validation

After updating `process/context/all-context.md`, regenerate the routing tables from frontmatter, then validate:

```bash
node .claude/skills/vc-context-discovery/scripts/discover-context.mjs --emit-routing
node .claude/skills/vc-generate-context/scripts/validate-all-context.mjs
```

If the update changes context routing, group membership, or grouped docs, also run:

```bash
node .claude/skills/vc-audit-context/scripts/validate-context-discovery.mjs
```

Fix validation failures before presenting the context as refreshed. Treat warnings as freshness
or quality findings unless the user asks for strict enforcement.

## Data Sources

Inspect as relevant:

- `pnpm-workspace.yaml`
- Root and workspace `package.json` files.
- `tsconfig*.json` and shared TypeScript tooling.
- Apps and packages under `apps/` and `packages/`.
- API routers, database schema/client, validators, container services, and infra modules.
- Tailwind and UI component setup.
- `.env` usage patterns without exposing secrets.
- `process/development-protocols/`, `AGENTS.md`, `.codex/agents/`, and `.agents/skills/` for workflow conventions.
- `process/general-plans/active/*/` and `process/features/*/active/*/` (plans live inside `{slug}_{date}/` task subfolders — scan one level deep).
- `process/context/all-context.md` for context routing and group ownership.
- Existing `process/context/**/*.md` files, loading only relevant grouped docs.
- `.claude/skills/vc-generate-plan/references/example-complex-prd.md` for plan/PRD depth expectations.

## Full Scan Structure

Use these sections unless a focused user request justifies a smaller update:

1. Product and PRD context.
2. Tech stack overview.
3. Monorepo layout.
4. Package manager and scripts.
5. TypeScript and module resolution.
6. API and backend.
7. Database and data layer.
8. Auth, payments, and integrations.
9. UI and styling.
10. Environment variables.
11. Linting, formatting, and quality.
12. Conventions and rules.
13. Security posture.
14. Monitoring and operations.
15. References and key files.
16. Open questions.

### all-context.md Seed Section Population

When populating `all-context.md` from seed templates or from a fresh scan, write real content into each section. Replace seed placeholders with actual project data:

1. **Title**: Replace the project name placeholder with the actual project name.
2. **Project description**: Use the user's own words when available (from vc-setup's ASK step). Supplement with what the code scan reveals. The user's description should be prominent, not buried.
3. **Quick Start section**: Keep the generic routing instructions from the template.
4. **Current Root Entry Points table** and **Current Context Groups table**: do NOT hand-fill these. They live between `<!-- GENERATED:routing -->` markers and are generated from each group doc's frontmatter. First ensure every `all-{group}.md` you wrote has complete frontmatter (`name: context:all-{group}`, `description`, a non-empty `keywords` list of task-vocabulary terms, optional `related: [context:{slug}]` siblings), then run `node .claude/skills/vc-context-discovery/scripts/discover-context.mjs --emit-routing` to populate both tables.
6. **Task Routing table**: hand-author this (it maps editorial task types → entry points). Fill based on what context groups exist.
7. **Repository Structure**: Write actual directory tree output (2-3 levels deep, showing key directories and files).
8. **Technology Stack**: Write specific framework names, versions, and combinations discovered during analysis.
9. **Key Patterns and Conventions**: Document actual patterns found in the codebase (error handling, state management, API patterns, naming conventions, import aliases). Include conventions the user mentioned.
10. **Environment and Configuration**: List actual env var groups found (names only, never values).
11. **Context Group Lifecycle**: Keep the generic instructions from the template.
12. **Scan Metadata** (add at bottom): include generated timestamp, HEAD from `git rev-parse HEAD` if available, mode (fresh/merge/refresh), and package manager detected.

## Delta Update Rules

1. Parse the existing context and preserve unchanged sections.
2. Verify likely drift areas: dependencies, scripts, package layout, API routes, schemas, env vars, active plans, feature folders, and context router/group changes.
3. Add a "Changes since last update" section near the top.
4. Tag product-impacting changes with `[Product]`.
5. Keep stale or contradicted statements out of the final file.
6. If `all-context.md` contradicts a grouped context doc, inspect source code before deciding which statement is stale.
7. Run the context validator and report any remaining warnings.
