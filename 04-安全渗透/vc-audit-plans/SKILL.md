---
name: vc-audit-plans
description: Audit active project plan files for staleness, completion, and routing truth. Use when cleaning up plans, reconciling active work, or archiving completed artifacts.
trigger_keywords: audit plans, plan inventory, stale plans, active plan count
layer: contract
metadata:
  author: vibecode-pro-max-kit
  version: "1.0.0"
---

# Audit Plans

> **Output style:** Follow `process/development-protocols/communication-standards.md` — answer-first, plain language, no unexplained jargon, TL;DR on long responses.

Use this skill to review active plan artifacts and reconcile them with the current codebase.

This is a maintenance and recovery skill, not an automatic post-task hook.

Optional input: a feature, folder, plan filename, or maintenance scope to prioritize.

Prefer it when:

- UPDATE PROCESS was skipped and active-plan cleanup drift accumulated
- the user wants a periodic active-plan cleanup pass
- multiple active plans need reconciliation after a burst of work

## Workflow

1. Read `references/audit-plans.md` for the full audit process.
2. Run the inventory validator:
   ```bash
   node .claude/skills/vc-audit-plans/scripts/validate-plan-inventory.mjs
   ```
3. Inventory plans in `process/general-plans/active/` and `process/features/*/active/`. Plans now live inside `{slug}_{date}/` task subfolders — scan one level deep. Do NOT count `_REPORT_`, `_REF_`, or `_SPEC_` files inside task folders as plans; only `_PLAN_` files count.
   For feature-scoped audits, first run `find process/features/{feature}/ -type f | sort` for full
   artifact visibility. For full audits, run `find process/features/ -type f | sort` to see all
   feature artifacts across all subdirs (active, completed, backlog, references, reports).
3.5. Scan task folder contents (co-located REPORT/REF/SPEC files) alongside each plan. Per **task-folder artefact colocation**, the correct home for every artefact (plan, spec, reports, references) is INSIDE its `{slug}_{date}/` task folder; flag any task artefact found in the deprecated sibling `reports/`/`references/` dirs or any ad-hoc location as mis-located, and recommend moving it into the owning task folder. Match by feature slug, date proximity (7 days), or content reference to the plan filename.
4. Cross-check each plan against the actual codebase with file existence checks and targeted `rg` searches.
5. Classify each plan as `Completed`, `Partially Done`, `Obsolete`, `Stale`, `Active`, or `Reference`.
6. Move only clearly completed or obsolete plans to the appropriate `completed/` folder. Use `git mv active/{slug}_{date}/ completed/{slug}_{date}/` — move the WHOLE task folder; no `completed_` prefix added.
7. Ask before deleting anything.
8. Re-run the inventory validator after moving or editing plan files.

## Output

Return a concise summary table with classification, action taken, and any user decisions needed. Include stale artifact findings (reports/references tied to completed or obsolete plans) with recommended actions.
