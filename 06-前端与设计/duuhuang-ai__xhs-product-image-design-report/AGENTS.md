# Repository Rules

## Scope

This repository contains one public Codex/Agent Skill. Keep the repository self-contained and do not add unrelated marketplace or application code.

## Structure

- `SKILL.md`: authoritative Agent instructions and invocation boundary.
- `agents/openai.yaml`: UI metadata and default prompt; keep it consistent with `SKILL.md`.
- `scripts/`: deterministic helpers used by the Skill.
- `references/`: conditional reference material linked from `SKILL.md`.
- `assets/`: redistributable inputs or templates required by the Skill.
- `tests/`: tests for repository scripts.
- `README.md`: user-facing installation and usage documentation.

Do not add caches, generated reports, user data, credentials, cookies, access tokens, local absolute paths, or empty placeholder directories.

## Changes and Verification

- Preserve the documented capability boundary; do not claim unsupported APIs, data access, or automation.
- Update `README.md` whenever installation, dependencies, inputs, outputs, or workflow behavior changes.
- Run the Codex Skill quick validator after changing `SKILL.md` or `agents/openai.yaml`.
- Run the smallest relevant script tests or syntax checks after changing scripts.
- Check relative links and `git diff --check` before committing.

## Publishing Boundary

Creating or changing a GitHub remote, pushing commits, publishing releases, or deleting files requires the repository owner's explicit approval.

