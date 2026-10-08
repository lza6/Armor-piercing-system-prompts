# Cangjie Skill Website MVP Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use executing-plans to implement this plan task-by-task.

**Goal:** Build a minimal, functional static website that teaches Skill usage, exposes the current Skill Pack registry, supports search and filtering, and generates GitHub-native submissions for external repositories or local Skill folders.

**Architecture:** Add an Astro static site under `website/` and keep structured catalog data under the repository-level `registry/`. Build-time Node scripts validate YAML entries with JSON Schema and generate all pages without a runtime database. Client JavaScript is limited to search, copy actions, form persistence, local folder validation, and submission bundle generation.

**Tech Stack:** Astro 7, TypeScript, Vitest, js-yaml, Ajv, Fuse.js, JSZip, GitHub Actions, GitHub Pages.

---

### Task 1: Scaffold the Astro application and test harness

**Files:**
- Create: `website/package.json`
- Create: `website/astro.config.mjs`
- Create: `website/tsconfig.json`
- Create: `website/vitest.config.ts`
- Create: `website/src/env.d.ts`

**Steps:**
1. Define production and development scripts for registry validation, tests, Astro checks, builds, and preview.
2. Install pinned dependencies and generate `website/package-lock.json`.
3. Add a smoke unit test proving Vitest is operational.
4. Run `npm test` and confirm PASS.

### Task 2: Define and seed the Registry

**Files:**
- Create: `schemas/registry-entry.schema.json`
- Create: `registry/<slug>/entry.yaml` for all 22 current Packs.
- Create: `website/scripts/validate-registry.mjs`
- Create: `website/src/lib/catalog.ts`
- Test: `website/src/lib/catalog.test.ts`

**Steps:**
1. Write tests for loading entries, rejecting duplicate slugs, and computing 22 Packs / 300 Skills.
2. Run the tests and confirm they fail before the loader exists.
3. Implement the schema, YAML loader, validation script, and catalog statistics.
4. Seed the 22 entries currently listed in `README.md`.
5. Run `npm run validate:registry` and `npm test`; both must pass.

### Task 3: Build shared layout and the homepage

**Files:**
- Create: `website/src/layouts/BaseLayout.astro`
- Create: `website/src/components/Header.astro`
- Create: `website/src/components/Footer.astro`
- Create: `website/src/components/PackCard.astro`
- Create: `website/src/styles/global.css`
- Create: `website/src/pages/index.astro`

**Steps:**
1. Implement accessible header, footer, skip link, buttons, cards, and responsive content container.
2. Render Hero, computed stats, three-step usage explanation, six featured Packs, and submission CTA.
3. Use restrained white/ink styling with one accent and no visual dependencies.
4. Run Astro checks and build.

### Task 4: Build tutorial, library, and Pack detail pages

**Files:**
- Create: `website/src/data/platforms.ts`
- Create: `website/src/pages/learn.astro`
- Create: `website/src/pages/skills/index.astro`
- Create: `website/src/pages/skills/[slug].astro`

**Steps:**
1. Implement verified/experimental platform guidance and copyable install paths.
2. Render the complete Pack list with search text and filter metadata.
3. Add client-side search, source filter, domain filter, quality filter, reset, count, and empty state.
4. Generate one static detail page per Pack with metadata, use cases, install guidance, and GitHub link.
5. Run tests, Astro checks, and build.

### Task 5: Implement submission generation and local validation

**Files:**
- Create: `website/src/lib/submission.ts`
- Test: `website/src/lib/submission.test.ts`
- Create: `website/src/pages/submit.astro`

**Steps:**
1. Write tests for slug normalization, YAML generation, missing fields, frontmatter parsing, and secret detection.
2. Implement pure submission utilities until tests pass.
3. Build a five-step form supporting `external` and `bundled` modes.
4. Persist form values locally, preview generated YAML, and provide copy/download actions.
5. For local folders, validate client-side and generate a ZIP containing `entry.yaml` plus the Skill folder.
6. Provide GitHub create-file/fork instructions without storing credentials.
7. Run tests, Astro checks, and build.

### Task 6: Add contribution guidance and CI

**Files:**
- Create: `CONTRIBUTING.md`
- Create: `.github/PULL_REQUEST_TEMPLATE.md`
- Create: `.github/workflows/website-ci.yml`
- Create: `.github/workflows/deploy-pages.yml`

**Steps:**
1. Document external and bundled submission structures.
2. Add a PR checklist for provenance, license, tests, and secrets.
3. Add read-only pull request CI that validates Registry data, runs tests/checks, and builds the site without executing contributed scripts.
4. Add GitHub Pages deployment from `main` with the repository base path.
5. Validate workflow YAML structure by inspection and run the same CI commands locally.

### Task 7: End-to-end verification

**Files:**
- Modify only files required to fix verification findings.

**Steps:**
1. Run `npm run validate:registry`.
2. Run `npm test`.
3. Run `npm run check`.
4. Run `npm run build`.
5. Start the local site and verify every route returns 200.
6. Capture desktop and mobile screenshots, inspect hierarchy, overflow, form states, and empty search state.
7. Fix all visible or functional defects and rerun the complete verification set.
