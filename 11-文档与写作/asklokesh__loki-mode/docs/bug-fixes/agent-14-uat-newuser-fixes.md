# Agent 14: First-Time User Acceptance Testing -- Bug Fixes

**Date:** 2026-03-24
**Scope:** Full first-time user journey audit (install through first build)
**Files Modified:** `autonomy/loki`, `docs/INSTALLATION.md`, `README.md`

---

## Bugs Fixed

### BUG-FTU-001: `loki init` does not tell user to set up AI provider
**Severity:** High -- first-time users scaffold a project but have no idea they need a provider CLI
**Location:** `autonomy/loki`, `cmd_init()` (line ~7793)
**Fix:** Added post-scaffold check that detects whether any AI provider CLI (claude, codex, gemini, cline, aider) is installed. If none found, prints clear installation instructions and suggests running `loki doctor`.

### BUG-FTU-002: `loki web` opens browser before server is ready
**Severity:** Medium -- user sees a blank page or connection refused on first launch
**Location:** `autonomy/loki`, `cmd_web_start()` (line ~3336)
**Root Cause:** The readiness loop (`curl` against `/api/session/status`) ran up to 15 retries, but its result was never checked. The browser opened regardless of whether the server actually responded.
**Fix:** Track readiness in a `server_ready` boolean. Only open browser when `server_ready=true`. If the server is still starting, print a message telling the user to open the URL manually or refresh.

### BUG-FTU-003: `loki quick` with no provider CLI gives unhelpful error
**Severity:** High -- user sees a cryptic `run.sh` error instead of actionable guidance
**Location:** `autonomy/loki`, `cmd_quick()` (line ~7050)
**Fix:** Added pre-flight provider CLI check before `exec "$RUN_SH"`. If the provider CLI is missing, prints the specific install command for that provider (e.g., `npm install -g @anthropic-ai/claude-code` for claude).

### BUG-FTU-005: `loki start` with no provider CLI gives unhelpful error
**Severity:** High -- same root cause as BUG-FTU-003 but for the main `start` command
**Location:** `autonomy/loki`, `cmd_start()` (line ~1095)
**Fix:** Added pre-flight provider CLI check before `exec "$RUN_SH"`. Clear error message with install command.

### BUG-FTU-006: `loki doctor` does not check API keys or "no provider at all"
**Severity:** Medium -- doctor gives green output even when no provider is usable
**Location:** `autonomy/loki`, `cmd_doctor()` (line ~5902)
**Fix:** Added two new sections to doctor output:
1. After listing all provider CLIs, check if zero providers are installed and show a FAIL with install instructions.
2. New "API Keys" section showing status of `ANTHROPIC_API_KEY`, `OPENAI_API_KEY`, `GOOGLE_API_KEY`. For provider CLIs that use their own login sessions, a note is shown instead of a failure.

### BUG-FTU-004/BUG-FTU-007: INSTALLATION.md contains inaccurate references
**Severity:** Medium -- confuses new users with nonexistent paths
**Location:** `docs/INSTALLATION.md`
**Fixes:**
- **Wrong license**: File structure section claimed "MIT License" but actual license is "Business Source License 1.1". Fixed.
- **Wrong directory**: Referenced `examples/` directory (which does not exist) instead of `templates/`. Fixed to show `templates/` with accurate description.
- **Broken next steps**: "Next Steps" section referenced `./autonomy/run.sh examples/simple-todo-app.md` which is a path that does not exist. Replaced with the standard workflow: `loki doctor` -> `loki init` -> `loki start`.
- **Stale note**: "Some files/directories (autonomy, tests, examples)" changed to "templates".
- **Broken relative link**: `[README.md](README.md)` from `docs/` should be `[README.md](../README.md)`. Fixed.

---

## README.md Improvements

### Improved "Get Started in 30 Seconds" section
**Problem:** The quick start jumped directly from `npm install` to `loki start ./prd.md` without explaining where a first-time user gets a PRD file. This was a dead end for anyone who does not already have a PRD.
**Fix:** Added `loki init my-app --template simple-todo-app` and `cd my-app` steps to bridge the gap. Also added a `loki quick` alternative for users who want to skip PRD creation entirely.

---

## Bugs Verified as Already Fixed

### BUG-CLI-001: `--port` flag crashes (unbound variable)
**Status:** Already fixed in current codebase.
**Evidence:** Both `cmd_web_start()` and `cmd_dashboard_start()` properly guard the `--port` flag with `[[ -z "${2:-}" ]]` checks and have default port variables (`PURPLE_LAB_DEFAULT_PORT=57375`, `DASHBOARD_DEFAULT_PORT=57374`). All port references use `${LOKI_DASHBOARD_PORT:-57374}` pattern. No unbound variable risk.

---

## New Bugs Discovered (Not Fixed -- Documenting Only)

### BUG-FTU-008: `INSTALLATION.md` "What's New" section is stale
The section header says "What's New in v6.7.0" but the current version is v6.71.1. The content describes features from v5.15.0 through v6.1.0 -- all many versions old. This misleads first-time users about the product's current state. Recommendation: either update to show recent highlights or remove version-specific "what's new" content from the installation guide entirely (it belongs in the CHANGELOG).

### BUG-FTU-009: `loki doctor` providers all marked "optional"
All five AI providers show as "optional" in doctor output. For a first-time user, this implies none of them are needed, when in fact at least one is required for any functionality. The fix added above (checking for zero providers) mitigates this, but the individual items could be marked "at least one required" for clarity.

---

## Test Matrix

| Journey Step | Before | After |
|---|---|---|
| `loki init my-app` with no provider CLI | No guidance | Prints install instructions |
| `loki start prd.md` with no provider CLI | Cryptic run.sh error | Clear error with install command |
| `loki quick "task"` with no provider CLI | Cryptic run.sh error | Clear error with install command |
| `loki web` on slow server start | Browser opens to blank page | Browser deferred; user told to refresh |
| `loki doctor` with no providers | All green (misleading) | Explicit FAIL + API key section |
| INSTALLATION.md file structure | References nonexistent `examples/` | References correct `templates/` |
| INSTALLATION.md license | Claims MIT | Correctly says BSL 1.1 |
| README.md quick start | Assumes user has a PRD | Guides through `loki init` |
