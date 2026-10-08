# Releases

Vibe Code Common Sense tracks releases like a normal open source project.

The current version is recorded in the root `VERSION` file. User-facing changes
are recorded in `CHANGELOG.md`. Published versions should also have a GitHub
release so users can see what changed without reading the repository history.

## Current Version

`0.1.0`

## Release Process

Use this checklist when preparing a release:

1. Update `VERSION`.
2. Update `CHANGELOG.md`.
3. Update docs if installation, commands, or workflows changed.
4. Run validation for the Codex skill, Claude Code plugin, and docs site.
5. Open and merge a focused release PR.
6. Create a signed or annotated Git tag, for example `v0.1.0`.
7. Publish a GitHub release using the matching changelog notes.

## Version Policy

Use semantic versioning:

- `MAJOR` for breaking changes to the skill or plugin workflow.
- `MINOR` for new templates, docs, commands, or workflow capabilities.
- `PATCH` for fixes and small documentation corrections.

## v0.1.0

First public release.

Includes the Codex skill, Claude Code plugin, manual agent prompt fallback,
project rail templates, GitHub Pages documentation, and the PR-based progress
tracking workflow centered on `docs/PRODUCT_TIMELINE.md`.
