---
name: triage-asc-issue
description: Triage App-Store-Connect-CLI GitHub issues against current code, CLI behavior, and App Store Connect API support. Use when the user asks to audit, reproduce, classify, label, scope, prioritize, or decide whether an ASC CLI issue should be fixed or implemented.
---

# Triage an ASC CLI issue

Produce a current, evidence-backed verdict and proposed repository labels. Apply label changes only when authorized under `AGENTS.md`; a read-only triage ends with recommendations.

## Read current evidence

1. Fetch the live issue body, comments, labels, linked PRs, and referenced documentation.
2. Resolve the affected command through current `--help`, source, tests, and recent history. Do not infer support from an old command name.
3. Reproduce reported behavior with the current built or installed CLI when safe.
4. For API claims, verify the exact method and endpoint in `docs/openapi/latest.json`; use the `sosumi.ai` documentation mirror for explanatory context.
5. Check whether the report is already fixed on current `origin/main`, duplicated, unsupported by the public API, or blocked by Apple/platform behavior.

Follow `AGENTS.md` for parallel reads and serialized writes. Live reproductions require authority for mutations and cleanup, even on disposable resources; confirm dry runs are side-effect-free.

## Classify the outcome

Choose one primary verdict:

- Confirmed bug with a reproducible expected-versus-actual mismatch.
- Valid enhancement with a supported and coherent API or client-side design.
- Question requiring clarification or support guidance.
- Already fixed, duplicate, not reproducible, unsupported, or platform-limited.

Separate urgency from implementation size. Explain user impact, blast radius, workaround, compatibility risk, and the smallest proof needed for completion.

## Recommend or apply labels

Recommend one type, priority, and difficulty from `AGENTS.md`'s buckets using the meanings in `CONTRIBUTING.md`. Apply them only when authorized.

For authorized label updates, remove conflicting labels and add the selected replacements, then verify the resulting buckets. If evidence is incomplete, mark the recommendation provisional and identify the missing evidence; uncertainty alone does not establish low urgency or easy implementation.

## Define implementation readiness

When the issue is actionable, provide:

1. Proposed command and UX shape.
2. API endpoint or explicit client-side behavior.
3. Files and shared surfaces likely affected.
4. RED-GREEN test plan and black-box exit/output cases.
5. Safe live verification and cleanup plan.
6. Compatibility or deprecation requirements.

If implementation is requested, continue with `$develop-asc-change` in an isolated branch or worktree. Report the requested implementation and validation as complete when finished, with PR and merge gates separately. Do not close the issue or claim it resolved until merged and verified; closure still requires authority.

## Automation contract

A recurring issue-triage automation may classify new or incompletely labeled issues and report actionable findings. Apply labels or post comments only within its persisted write authority. It must not implement code or close issues without explicit authorization, and labels must follow evidence.
