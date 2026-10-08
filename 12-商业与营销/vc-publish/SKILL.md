---
name: vc-publish
description: Use when publishing harness improvements to the remote kit repo. Diffs managed files, shows what changed, bumps version, and pushes. Counterpart to vc-update (pull).
trigger_keywords: publish kit, push harness, release kit, update remote
layer: contract
metadata:
  author: vibecode
  version: "3.0.0"
---

# vc-publish

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Push harness improvements from the current development repo to the remote kit repository (`vibecode-pro-max-kit`). This is the **maintainer** counterpart to `vc-update`.

- `vc-update` = **user** pulls latest harness INTO their project FROM the remote
- `vc-publish` = **maintainer** pushes improvements FROM the development repo TO the remote kit repo

## Prerequisites

- Local checkout of the kit repo (`git clone git@github.com:withkynam/vibecode-pro-max-kit.git`)
- `.vc-publish-config` file in the current repo root (see Configuration below)
- Git push access to the remote kit repo

## Configuration

Create `.vc-publish-config` in the repo root:

```json
{"kitRepoPath": "/path/to/vibecode-pro-max-kit"}
```

If this file is missing, ask the user for the kit repo checkout path and offer to create it.

## Workflow

### Step 1: Load Configuration

1. Read `.vc-publish-config` from the current repo root.
2. If missing, ask the user for the kit repo local checkout path.
3. Verify the path exists and contains `vc-manifest.json`.
4. Verify the kit repo worktree is clean (`git -C <kitRepoPath> status --porcelain`). If dirty, warn and ask whether to proceed or abort.

### Step 2: Read Manifest

5. Read `vc-manifest.json` from the kit repo checkout.
6. Extract the current `version`.
7. Before computing a version bump, check if the current kit version already matches the intended target version (e.g. `3.0.0`). If the kit `version` already equals the target: **skip the bump step entirely** and proceed directly to Step 3 with a `tag-as-is` note — do not increment the version. Record in the publish summary that the version was unchanged.

**Catalog-regen (pre-publish):** Before resolving files in Steps 3–4, regenerate the skills catalog in the dev repo:
```bash
node .claude/skills/vc-audit-context/scripts/generate-skills-catalog.mjs --write
```
This ensures `process/context/generated-skills-catalog.json` is current before it is copied into the kit repo.

### Step 3: Resolve Kit File Set

7. Run the resolver against the **kit repo** to get the kit file list:
   ```bash
   node <kitRepoPath>/resolve-manifest.mjs --root <kitRepoPath> --json
   ```
   Extract `files` (kit managed files) and `kitOnly` (kit-exclusive files).

   **Note:** `resolve-manifest.mjs` reads `vc-manifest.json` from its `--root` directory and also scans files from that same root. `vc-manifest.json` is NOT installed into dev/user projects by `install.sh`, so the resolver must always be pointed at the kit repo checkout (which does have it). There is no separate dev-repo resolver call — the dev-side file comparison happens inside `compute-sync-plan.mjs` in Step 4.

### Step 4: Compute Diff

9. **Computation via `compute-sync-plan.mjs`:** Use the shared computation core to produce the diff between the dev repo's managed files and the kit repo's current managed files.

   > **Direction note:** `compute-sync-plan.mjs` loads `vc-manifest.json` from `--kit-root` and runs the resolver with `--root <kit-root>`. Since `vc-manifest.json` lives in the kit repo (not the dev repo), `--kit-root` must always be the kit repo checkout. `--root` is the dev repo (the project being compared). This is the same direction as a normal install — vc-publish uses it to see what a fresh install FROM dev INTO the kit would change.

   ```bash
   # --root = dev repo (the "project" being compared against the kit source)
   # --kit-root = kit repo (where vc-manifest.json lives; source of truth for file lists)
   # --resolver overrides the resolver path because compute-sync-plan
   # would otherwise look for resolve-manifest.mjs inside --kit-root,
   # which IS the kit repo here, so --resolver is optional but explicit for clarity.
   node <kitRepoPath>/compute-sync-plan.mjs \
     --root <devRepoRoot> \
     --kit-root <kitRepoPath> \
     --resolver <kitRepoPath>/resolve-manifest.mjs \
     --json
   ```

   Parse the JSON output: `{ toAdd, toModify, toDelete, toPreserve, staleWarnings }`.
   - `toAdd` — files to copy from dev to kit (present in dev, not yet in kit).
   - `toModify` — files to overwrite in kit (tracked in both, content differs).
   - `toDelete` — files to remove from kit (no longer in dev managed set).
   - `toPreserve` — files to leave untouched (merge/copyIfMissing survivors, unchanged files).
   - `staleWarnings` — paths that failed the namespace guard — print to user; do NOT delete.

   The ownedPaths for the publish direction are the dev repo's resolved `ownedPaths`. CLAUDE.md and AGENTS.md are always in the `merge` category — they require special stripping regardless of diff status (see Step 7).

### Step 5: Print Diff Summary

10. Print a summary table:

```
vc-publish diff: current repo -> kit repo (v2.1.0)
================================================

FILES:
  [modified]  .claude/agents/vc-execute-agent.md  (+8 -3)
  [modified]  .claude/hooks/lib/scout-checker.cjs  (+2 -1)
  [new]       .claude/skills/vc-new-skill/SKILL.md
  [merge]     CLAUDE.md (needs content review)
  [merge]     AGENTS.md (needs content review)
  [unchanged] .claude/settings.json
  ... (350 more unchanged)

Total changes: 4 files modified, 1 new, 0 removed
```

### Step 6: STOP -- Confirm Publish

11. **STOP** and ask the user:
    - Confirm they want to publish these changes.
    - Specify version bump type: **patch**, **minor**, or **major**.
    - Optionally provide **release notes** (1–3 sentences for the GitHub Release body). Leave blank to auto-generate from the diff summary (e.g. "4 modified, 1 new, 0 removed.").
    - Or abort.

Version bump semantics:
- **Patch** (2.1.0 -> 2.1.1): hook fixes, skill doc updates, minor agent prompt tweaks
- **Minor** (2.1.0 -> 2.2.0): new skills, new agents, new development protocols
- **Major** (2.1.0 -> 3.0.0): CLAUDE.md structure changes, manifest schema changes, breaking workflow changes

### Step 7: Apply Changes

12. On confirm:
    - Copy all **modified** and **new** managed files from current repo to kit repo checkout.
    - For each **removed** file: delete it from the kit repo checkout.
    - **CLAUDE.md and AGENTS.md stripping**: Do NOT copy the current repo's project-specific versions directly. Instead:
      1. Read the current repo's CLAUDE.md/AGENTS.md.
      2. Read the kit repo's existing harness-only version as base.
      3. Apply only methodology/structural changes from the dev repo to the kit's harness-only version.
      4. Strip all project-specific content:
         - Technology stack details (frameworks, databases, versions)
         - Feature list / "Current features" entries
         - Project-specific context groups
         - Hardcoded package manager (replace with generic)
         - MCP server instructions (project-specific config)
         - Project-specific routing rules
         - Absolute paths (`/Users/...`)
         - Product name references (the project's product name and repo/directory name)
      5. Verify the result is harness-only methodology with no project leaks.
    - **Manifest reconciliation** (`vc-manifest.json`) — the manifest is NOT a normally-copied
      managed file (it is dev-only on one side and kit-resolved on the other), so its fields do
      NOT auto-sync. Handle it explicitly:
      1. **`version`**: bump per the chosen bump type. Kit-authoritative.
      2. **`legacyDeletions`** (the deprecation ledger): **dev is authoritative and always the
         superset.** Whenever dev removes a skill/dir/file from the harness it appends the path
         here so downstream projects clean it up on their next `vc-update`. Set
         `kit.legacyDeletions = dev.legacyDeletions` (preserve dev order). This field is a literal
         path array, NOT a glob — it genuinely changes every time the harness deprecates something,
         so it MUST be reconciled at every publish. (Historically this was skipped, which silently
         stranded the kit with stale deletions — never skip it.)
      3. **All OTHER non-version fields** (`include`, `exclude`, `strip`, `merge`, `copyIfMissing`,
         `symlinks`, `kitOnly`): these legitimately diverge — the kit carries packaging-only rules
         (e.g. excluding `**/*.test.mjs` and `__tests__/**` so tests are not shipped to users;
         `kitOnly` tooling like `compute-sync-plan.mjs`). **Do NOT blindly overwrite them** — a
         blanket dev→kit copy would strip the kit's test-excludes and ship test files. Instead run
         the **field-level drift report** below and reconcile any unexpected drift by hand.

      Drift-report command (run during Step 4 summary AND here before writing):
      ```bash
      node -e '
      const d=require("<devRepoPath>/vc-manifest.json");
      const k=require("<kitRepoPath>/vc-manifest.json");
      let drift=0;
      for(const key of new Set([...Object.keys(d),...Object.keys(k)])){
        if(key==="version") continue;
        if(JSON.stringify(d[key])!==JSON.stringify(k[key])){
          drift++;
          console.log("DRIFT field:",key);
          console.log("  dev:",JSON.stringify(d[key]));
          console.log("  kit:",JSON.stringify(k[key]));
        }
      }
      console.log(drift?("\n"+drift+" manifest field(s) drift — legacyDeletions auto-syncs dev→kit; reconcile the rest consciously."):"manifest in sync (besides version)");
      '
      ```
      `legacyDeletions` appearing in the drift report is EXPECTED and is auto-resolved (dev→kit).
      Any OTHER field in the report is a conscious decision: confirm the kit value is the intended
      packaging rule, or update dev/kit so they converge. Never let a drift go unexamined.
    - Create symlinks if missing (`.agents/skills -> ../.claude/skills`).

### Step 8: Leak Detection

13. Verify no project-specific content leaked into the kit repo. This is a
    **resolved-set, two-check** gate that scans the full shipped TEXT surface, not
    just `CLAUDE.md`/`AGENTS.md`.

    **Resolve the shipped set** via the kit's resolver, then restrict to TEXT
    surfaces:

    ```bash
    node <kitRepoPath>/resolve-manifest.mjs --root <kitRepoPath> --json
    ```

    Take the resolved `files` and keep only TEXT surfaces:
    - `.claude/skills/**` matching `*.md`, `*.cjs`, `*.mjs`, `*.py`, `*.js`, `*.json`
    - `.claude/agents/**` matching `*.md`
    - `.codex/**`
    - `process/development-protocols/**`
    - plus `CLAUDE.md`, `AGENTS.md`

    Exclude binaries and `**/node_modules/**`.

    **Check (a) -- product-name grep over the resolved text set.** Scan for the
    product names in the grep below ONLY. `tRPC`/`Prisma` are DROPPED from this
    skill-prose scan to avoid false positives in legitimate generic test guidance;
    the hosted-database product name is KEPT (see the pattern):

    ```bash
    grep -rIin "flowser\|CloakBrowser\|OpenClaw\|Supabase" <resolved-text-files>
    ```

    Allowlist the Bucket-4 lines that MUST keep the literal to function (otherwise
    the gate flags itself): `author: flowser` frontmatter; the `isFlowserActivePlanPath`
    identifier; this skill's OWN scrub-grep pattern lines below; the new validator's
    own pattern strings; and the one internal plan-generation validation comment in
    `session-init.cjs`.

    **Check (b) -- non-portable context-path grep:** any concrete backticked
    `process/context/...` file reference in the resolved text set, MINUS the
    shipped/seeded survivors, is a dangling-link leak → FAIL with file:line.
    Survivors (allowed): `process/context/all-context.md`,
    `process/context/tests/all-tests.md`. Portable directory refs (e.g.
    `process/context/tests/`) and the `process/context/...` placeholder are fine.

    **Keep the existing narrow `CLAUDE.md`/`AGENTS.md` grep** (this stays as-is on
    just those two files; `tRPC`/`Prisma` plus the hosted-database product name all
    REMAIN here, as shown in the pattern below):

    ```bash
    # Must return empty -- any matches indicate leaked content
    grep -ri "flowser\|tRPC\|Prisma\|Supabase\|CloakBrowser\|OpenClaw" CLAUDE.md AGENTS.md

    # Must return empty -- no absolute paths
    grep -r "/Users/" .
    ```

    **Check (c) -- README badge counts:** Verify the kit README.md badge counts match actual agent and skill counts:
    ```bash
    actual_agents=$(ls <kitRepoPath>/.claude/agents/*.md | wc -l | tr -d ' ')
    actual_skills=$(ls -d <kitRepoPath>/.claude/skills/vc-*/ | wc -l | tr -d ' ')
    readme_agents=$(grep -oE '[0-9]+-Agents' <kitRepoPath>/README.md | grep -oE '[0-9]+')
    readme_skills=$(grep -oE '[0-9]+-Skills' <kitRepoPath>/README.md | grep -oE '[0-9]+')
    echo "Agents: actual=$actual_agents badge=$readme_agents"
    echo "Skills: actual=$actual_skills badge=$readme_skills"
    [ "$actual_agents" = "$readme_agents" ] && [ "$actual_skills" = "$readme_skills" ] && echo "PASS" || echo "FAIL: badge counts mismatch"
    ```
    If FAIL: update README.md badges to match actual counts before committing.

    NOTE: the brand grep matches product names ONLY. It does NOT match
    `.ck.json`/`.ckignore` -- those Phase-2 legacy-fallback literals are intentional
    and must NOT be flagged. Do not add `ck`/`ckignore` to any leak grep.

    The standing `validate-kit-portability.mjs` validator (run by `vc-audit-vc`)
    mirrors checks (a) and (b) for between-release drift; this Step-8 gate is the
    publish-time enforcement.

14. If leak detection fails:
    - Print the offending lines (file:line).
    - Revert the changes in the kit repo (`git -C <kitRepoPath> checkout .`).
    - STOP and report the leak. Do NOT commit or push.

### Step 9a: Commit and Tag

15. In the kit repo. If the version was already at the target (skip-bump path from Step 2): use `tag-as-is` and commit with the existing version number. Otherwise: bump the version in `vc-manifest.json` and commit with the new version.

```bash
cd <kitRepoPath>
git add -A
git commit -m "Release vX.Y.Z"
git tag vX.Y.Z
```

### Step 9b: STOP — Explicit Push Approval (separate from Step 6 confirm)

Leak detection passed. The commit and tag are ready locally. Before running `git push`, you **MUST** stop and get explicit user approval:

> "Leak detection passed. The commit is ready locally (`git log --oneline -1` shows the new commit).
> **Type 'push' to publish to remote, or 'abort' to keep the commit local.**"

Do NOT run `git push` or `git push --tags` until the user types 'push' (or a clear affirmative). This is a **separate gate** from the publish-confirm at Step 6 — even if the user approved publishing in Step 6, they must re-confirm before the actual remote push.

If the user says 'abort':
- The local commit and tag are preserved.
- Print: "Commit and tag preserved locally. Run `git push origin main && git push --tags` when ready."
- Stop.

Only on explicit 'push': proceed to Step 10 (`git push origin main && git push --tags`).

### Step 10: Push

16. Push to remote (only after Step 9b approval):

```bash
git push origin main && git push --tags
```

17. If push fails (e.g., rejected, auth error), report the error. The commit and tag are preserved locally for retry.

### Step 11: Create GitHub Release

18. After a successful push, create a GitHub Release so watchers are notified and the release appears in the Releases tab:

    ```bash
    gh release create vX.Y.Z \
      --repo <remote-owner>/<remote-repo> \
      --title "vX.Y.Z: <first sentence of release notes>" \
      --notes "<full release notes>"
    ```

    - If the user provided release notes at Step 6, use them verbatim.
    - If left blank, auto-generate: `"N modified, M new, P removed. See commit log for details."`
    - The `--title` one-liner should be the first sentence of the notes (truncate at 72 chars if longer).
    - If `gh` is unavailable or the push to remote failed, skip this step and note it in the summary.

### Step 12: Post-Publish Remote Verify

After a successful push, clone the kit from remote to a temp dir and verify the catalog works on a fresh install:
```bash
TS=$(date +%s)
git clone <remote-kit-url> /tmp/vc-kit-verify-$TS
node /tmp/vc-kit-verify-$TS/.claude/skills/vc-context-discovery/scripts/discover-skills.mjs 2>&1
rm -rf /tmp/vc-kit-verify-$TS
```
Expected: exit 0 and expected skill count in output. If FAIL: note the error in the publish summary — the push succeeded but the remote install may have a catalog issue.

### Step 13: Print Summary

19. Print publish summary:

```
vc-publish complete
===================
Version:       v2.2.0 (was v2.1.0)
Files changed: 4
Remote:        git@github.com:withkynam/vibecode-pro-max-kit.git
Tag:           v2.2.0
Release:       https://github.com/<owner>/<repo>/releases/tag/v2.2.0
```

## Key Changes from v1.0

- **Glob arrays are stable; the deprecation ledger is not.** The glob patterns in
  `include`/`exclude`/`kitOnly` are stable — adding a new skill or agent requires zero manifest
  edits (new files are auto-included by the globs). BUT `legacyDeletions` is a literal path array,
  not a glob: it grows every time the harness deprecates a skill/dir/file, and it MUST be
  reconciled dev→kit at every publish (see Step 7 Manifest reconciliation). Skipping it strands
  the kit with a stale deletion ledger so downstream `vc-update`s never clean up the newly
  deprecated dirs. So publish-time manifest edits are: (1) version bump, (2) `legacyDeletions`
  sync, (3) conscious reconciliation of any other field flagged by the drift report.
- **Resolver-driven diffing.** The kit repo is resolved via `resolve-manifest.mjs` (which requires `vc-manifest.json` in its `--root`). The dev-side file comparison is handled inside `compute-sync-plan.mjs`, which reads the manifest from `--kit-root` (always the kit checkout). Dev repos do not carry `vc-manifest.json`.
- **No `managed`/`managedDirs` arrays to update.** The old workflow of adding new files to these arrays is eliminated.

## Rules

- **ALWAYS** run the full resolver diff (Steps 3-4) even when changes already exist in the kit repo. Direct kit edits (README, translations, community files) do not replace the dev→kit diff. Both change sources must be captured in the same publish.
- **NEVER** copy project-specific files: `process/context/all-context.md` (with real content), `process/features/*`, `process/general-plans/*` (with real plans)
- **ALWAYS** verify no project-specific content leaked before committing (Step 8)
- **ALWAYS** show the diff summary before publishing (Step 5-6)
- **NEVER auto-push.** `git push` must be preceded by a separate explicit 'push' confirmation (Step 9b), distinct from the publish-confirm at Step 6. This rule holds even when `VC_KIT_SOURCE` is a local path.
- CLAUDE.md and AGENTS.md require special handling -- never copy the development repo's project-specific versions directly
- Kit repo checkout path is stored in `.vc-publish-config` (add to `.gitignore`)
- The only manifest edit at publish time is the version bump -- glob patterns are stable

## Reference

See `references/vc-publish.md` for the detailed algorithm, CLAUDE.md/AGENTS.md stripping rules, error handling, and example outputs.
