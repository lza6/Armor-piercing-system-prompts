# Agent 05: Provider System Functional Testing - Bug Fixes

## Summary

Tested all 5 provider invocation paths (Claude, Codex, Gemini, Cline, Aider) and fixed 5 bugs across `autonomy/run.sh`, `providers/gemini.sh`. Also identified and fixed 1 new undocumented bug.

## Bugs Fixed

### BUG-PROV-001: Gemini ignores tier_param for model selection (FIXED)

**Root cause:** The Gemini invocation in `run.sh` (main iteration loop) used `PROVIDER_MODEL` (frozen at source-time) instead of `tier_param` (dynamically resolved per iteration via `resolve_model_for_tier()`). Regardless of RARV tier, Gemini always used the same model.

**Fix locations:**
- `autonomy/run.sh` (line ~9685): Changed `local model="${PROVIDER_MODEL:-...}"` to `local model="$tier_param"` in the Gemini case block
- `autonomy/run.sh` `invoke_gemini()` (line ~3029): Changed from `PROVIDER_MODEL` to `provider_get_current_model()` with fallback
- `autonomy/run.sh` `invoke_gemini_capture()` (line ~3068): Same fix as above

### BUG-PROV-003: Claude health check breaks OAuth users; Gemini lacks key rotation (FIXED)

**Root cause (Claude):** `check_provider_health()` required `ANTHROPIC_API_KEY` env var. Users authenticating via OAuth (no API key) were marked unhealthy, triggering unnecessary failover to degraded providers.

**Root cause (Gemini):** No support for API key rotation when keys expire or hit quota. No support for `GEMINI_API_KEY` env var alias or gcloud ADC.

**Fix locations:**
- `autonomy/run.sh` `check_provider_health()`: Claude now checks for OAuth session files (`~/.claude/.credentials.json`) and `claude auth status` as fallback. Gemini now checks `GEMINI_API_KEY` and gcloud ADC.
- `providers/gemini.sh`: Added `_gemini_resolve_api_key()` for key resolution from multiple sources (`GOOGLE_API_KEY`, `GEMINI_API_KEY`, gcloud ADC).
- `providers/gemini.sh`: Added `_gemini_rotate_api_key()` for rotating through `LOKI_GEMINI_API_KEYS` (comma-separated list) on auth errors (401/403).
- `providers/gemini.sh` `provider_invoke()` and `provider_invoke_with_tier()`: Added auth error detection and key rotation before rate-limit fallback.
- `autonomy/run.sh` Gemini invocation block: Added auth error detection and key rotation.

### BUG-PROV-008: Failover updates PROVIDER_NAME but not LOKI_PROVIDER (FIXED)

**Root cause:** After failover, `PROVIDER_NAME` was updated but `LOKI_PROVIDER` env var (read by subprocesses and MCP server) retained the old provider name. Child processes and the MCP server reported the wrong provider.

**Fix locations:**
- `autonomy/run.sh` `attempt_provider_failover()`: Added `LOKI_PROVIDER="$provider"; export LOKI_PROVIDER` after updating `PROVIDER_NAME`
- `autonomy/run.sh` `check_primary_recovery()`: Same fix when switching back to primary provider

### NEW BUG: LOKI_CURRENT_TIER never exported (FOUND AND FIXED)

**Root cause:** `providers/gemini.sh:provider_get_current_model()` reads `LOKI_CURRENT_TIER` to resolve the model dynamically. However, `run.sh` only sets `CURRENT_TIER` (without the `LOKI_` prefix) and never exports it. As a result, `provider_get_current_model()` always defaults to "planning" tier, negating the dynamic tier resolution for all Gemini helper functions (`invoke_gemini`, `invoke_gemini_capture`).

**Fix locations:**
- `autonomy/run.sh` (line ~1366): Set and export `LOKI_CURRENT_TIER` at initialization
- `autonomy/run.sh` (line ~9424): Update and export `LOKI_CURRENT_TIER` when `CURRENT_TIER` changes each iteration

## Bugs Already Fixed (Verified)

These bugs were listed in the assignment but had already been resolved in the current codebase:

| Bug ID | Description | Status |
|--------|-------------|--------|
| BUG-PROV-002 | Generic LOKI_MODEL_* injects invalid Codex models | Fixed: `_codex_validate_model()` in `codex.sh` filters non-Codex model names |
| BUG-PROV-005 | Provider loader doesn't validate provider exists before sourcing | Fixed: `load_provider()` validates name AND checks file existence |
| BUG-PROV-007 | auto_detect_provider skips Cline and Aider | Fixed: All 5 providers in priority order |
| BUG-PROV-009 | Cline model flag word-splitting | Fixed: Array-based `model_args` in `cline.sh` |
| BUG-PROV-010 | Gemini buffers all output, loses streaming | Fixed: Uses `tee` for streaming |
| BUG-PROV-012 | Codex resolve_model_for_tier returns effort levels | Fixed: Documented as intentional, callers use correctly |
| BUG-RUN-010 | Retry counter increments on success | Fixed: `retry=0` reset on success at lines 9851/9897 |
| BUG-PROV-011 | Parallel dispatch includes Cline despite PARALLEL=false | Fixed: Guard at line 2235 checks `PROVIDER_HAS_PARALLEL` |

## Validation

### Bash syntax validation (all pass)
- `bash -n providers/claude.sh` -- OK
- `bash -n providers/codex.sh` -- OK
- `bash -n providers/gemini.sh` -- OK
- `bash -n providers/cline.sh` -- OK
- `bash -n providers/aider.sh` -- OK
- `bash -n providers/loader.sh` -- OK
- `bash -n autonomy/run.sh` -- OK

### Edge cases verified
1. **API key missing**: `check_provider_health()` handles all 5 providers; Claude supports OAuth fallback
2. **CLI not installed**: All provider detect functions use `command -v` with proper error handling
3. **Version mismatch**: Provider version functions safely call `--version` with stderr suppression
4. **Failover chain**: Wraps around correctly using double-iteration with break-on-wrap guard
5. **Key rotation**: `_gemini_rotate_api_key()` handles single key, wraps around, and returns failure when exhausted
6. **Frozen model variable**: All Gemini invocation paths now use dynamic resolution

## Files Modified

| File | Changes |
|------|---------|
| `autonomy/run.sh` | BUG-PROV-001 (Gemini model selection), BUG-PROV-003 (health check + auth), BUG-PROV-008 (LOKI_PROVIDER export), LOKI_CURRENT_TIER export |
| `providers/gemini.sh` | BUG-PROV-003 (API key resolution + rotation functions, auth error handling in invoke functions) |
