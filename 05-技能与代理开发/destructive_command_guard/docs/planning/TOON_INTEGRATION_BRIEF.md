# TOON Integration Brief (dcg / destructive_command_guard)

Last updated: 2026-01-24

This document describes how dcg should support TOON output while **only** using
the **toon_rust** implementation:

- Rust code should use the `toon_rust` crate (preferred; no subprocess).
- dcg must **never** use the Node.js `toon` CLI / `@toon-format/cli`.

## 0. Non-Regression Contract (Critical)

dcg has two distinct machine-facing surfaces:

1. **Hook protocol** (Claude Code / agent hooks):
   - Input: JSON on stdin
   - Output: **JSON on stdout** with camelCase fields (protocol requirement)
   - Must remain JSON; TOON is not appropriate here.

2. **Robot mode** (`--robot` or `DCG_ROBOT=1`):
   - Contract: **JSON on stdout**, silent stderr, standardized exit codes
   - Must remain JSON by default to avoid breaking existing integrations.

TOON support should be scoped to **CLI-only outputs** that are currently offered
as JSON (e.g., `dcg test --format json`) and are not part of the hook protocol.

## 1. Where JSON Is Emitted Today (Code Map)

### Hook protocol (must remain JSON)

- `src/hook.rs`
  - Types:
    - `HookInput` (stdin JSON)
    - `HookOutput` / `HookSpecificOutput` (stdout JSON; camelCase fields)
  - Rationale: Claude Code hook protocol requires JSON and specific field names.

- `src/cli.rs`
  - `HookCommand { batch, parallel, ... }`
  - `BatchHookOutput` (JSONL line-by-line output in `dcg hook --batch`)

### CLI commands with JSON output today (candidate TOON targets)

- `src/cli.rs`
  - `dcg test`:
    - `TestFormat::{Pretty, Json}`
    - `TestOutput` (schema_version + dcg_version + decision fields)
    - `test_command(...)` prints `serde_json::to_string_pretty(&TestOutput)`
  - `dcg packs`:
    - `PacksFormat::{Pretty, Json}`
    - `PacksOutput`
  - Many other commands have their own `{X}Format::{Pretty, Json}` enums.

- `src/scan.rs`
  - `ScanFormat::{Pretty, Json, Markdown, Sarif}` and scan report structs.

## 2. Recommended TOON Enablement Scope

### Phase 1 (safe, minimal)

Add `toon` output for CLI subcommands where JSON is already supported and the
output is useful to an LLM:

- `dcg test --format toon`
- `dcg scan --format toon` (if/when scan supports it; see below)

Keep:

- Hook protocol outputs JSON-only (`dcg` no subcommand, `dcg hook --batch`)
- Robot mode JSON-only by default (`--robot` / `DCG_ROBOT=1`)

### Phase 2 (optional)

If we later want TOON in robot mode, require an explicit opt-in flag (or explicit
`--format toon`) and ensure hook mode continues to ignore any env var overrides.

## 3. Proposed Flag / Env Precedence

For CLI-only commands (NOT hook mode):

1. Explicit `--format <...>` flag
2. `DCG_OUTPUT_FORMAT` (new; recommended) OR keep existing `DCG_FORMAT`
3. `TOON_DEFAULT_FORMAT` (shared cross-tool convention)
4. Command default (currently usually `pretty`)

For hook protocol and robot mode:

- Hook protocol: always JSON / JSONL regardless of env vars.
- Robot mode: default JSON regardless of env vars.

## 4. Implementation Approach (Rust: crate-based)

### Dependency

Prefer using the crate directly (no subprocess):

```toml
# Cargo.toml
toon_rust = { path = "../toon_rust" }
```

If a path dep is undesirable for releases, switch to a git dep:

```toml
toon_rust = { git = "https://github.com/Dicklesworthstone/toon_rust" }
```

### Encoding helper

Add a tiny helper (new module, e.g. `src/output/toon.rs`) that takes any
`serde::Serialize` value and returns a TOON string:

```rust
pub fn encode_toon<T: serde::Serialize>(value: &T) -> Result<String, anyhow::Error> {
    let json = serde_json::to_value(value)?;
    Ok(toon_rust::encode(json, None))
}
```

Then in each command that supports `--format toon`, serialize the existing
payload struct (same fields as JSON) and print the encoded TOON to stdout.

## 5. Sample Payload Shapes (Fixtures)

### `dcg test` allow (JSON)

```json
{
  "schema_version": 1,
  "dcg_version": "X.Y.Z",
  "robot_mode": false,
  "command": "git status",
  "decision": "allow",
  "agent": { "detected": "unknown", "trust_level": "medium", "detection_method": "none" }
}
```

### `dcg test` deny (JSON)

```json
{
  "schema_version": 1,
  "dcg_version": "X.Y.Z",
  "robot_mode": false,
  "command": "rm -rf /",
  "decision": "deny",
  "rule_id": "core.filesystem:rm-rf-root",
  "pack_id": "core.filesystem",
  "pattern_name": "rm-rf-root",
  "reason": "Refusing to remove root directory",
  "source": "pack",
  "severity": "critical",
  "agent": { "detected": "unknown", "trust_level": "medium", "detection_method": "none" }
}
```

The TOON output should be the `toon_rust::encode(...)` of the same JSON payload.

## 6. Test Plan (Design)

### Unit tests

- Format precedence for CLI-only commands:
  - `--format` > `DCG_OUTPUT_FORMAT` (or `DCG_FORMAT`) > `TOON_DEFAULT_FORMAT` > default
- Hook protocol output stability:
  - With `DCG_ROBOT=1`, hook output shape remains JSON with camelCase fields.
- TOON encode sanity:
  - Encode a deny payload and decode it back (via `toon_rust::decode`) and compare to JSON.

### E2E script (future)

Avoid side effects; only use `dcg test`:

1. `dcg test "rm -rf /" --format json`
2. `dcg test "rm -rf /" --format toon`
3. Decode TOON back to JSON and compare payload equivalence.

## 7. Notes / Constraints

- `tru` is the toon_rust CLI binary and must not be confused with Unix `tr` or
  Node `toon`. This repo should not shell out; it should use the crate.
- Keep stdout data-only; stderr is diagnostics (except in `--robot` mode where
  stderr is silent by contract).

