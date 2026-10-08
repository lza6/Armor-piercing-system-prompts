# Dependency Upgrade Log

**Date:** 2026-02-19  |  **Project:** destructive_command_guard (dcg)  |  **Language:** Rust

## Summary
- **Updated:** 12 (Cargo.toml version bumps) + 82 semver-compatible lockfile updates
- **Skipped:** 3  |  **Failed:** 0  |  **Needs attention:** 1

## Phase 1: Semver-Compatible Updates (cargo update)

82 transitive packages updated within existing version ranges. All 1935 tests passed.

Key updates: clap 4.5.54 → 4.5.60, regex 1.12.2 → 1.12.3, memchr 2.7.6 → 2.8.0,
rust-mcp-sdk 0.8.2 → 0.8.3, tokio (transitive deps), tree-sitter 0.26.3 → 0.26.5.

## Phase 2: Breaking Version Bumps

### rust-mcp-sdk: 0.8.1 → 0.8.3
- **Breaking:** None (patch bump, already resolved by cargo update)
- **Tests:** Passed

### fancy-regex: 0.14 → 0.17
- **Breaking:** None (new features only: split(), splitn(), RegexBuilder options)
- **Tests:** Passed

### colored: 2.1 → 3.1
- **Breaking:** MSRV bump to 1.80+ (we use nightly, no issue). lazy_static removed internally.
- **Code changes:** None
- **Tests:** Passed

### dirs: 5.0 → 6.0
- **Breaking:** None (dependency maintenance release, full API compat)
- **Tests:** Passed

### console: 0.15 → 0.16
- **Breaking:** `std` feature flag introduced (enabled by default)
- **Code changes:** None
- **Tests:** Passed

### indicatif: 0.17 → 0.18
- **Breaking:** Depends on console 0.16, switched number_prefix to unit-prefix
- **Code changes:** None
- **Tests:** Passed

### inquire: 0.7 → 0.9
- **Breaking:** RenderConfig now Copy, bitflags v2, MSRV 1.82+
- **Code changes:** None (our usage doesn't touch RenderConfig directly)
- **Tests:** Passed

### toml: 0.8 → 1.0
- **Breaking:** Deserializer::new() returns Result, FromStr for Value changed
- **Code changes:** None (our usage is all toml::from_str/to_string_pretty which are unchanged)
- **Tests:** Passed

### toml_edit: 0.22 → 0.25
- **Breaking:** InternalString removed, Time fields wrapped in Option, table position changes
- **Code changes:** None (our usage is DocumentMut, Table, value(), ArrayOfTables)
- **Tests:** Passed

### rusqlite: 0.35 → 0.38
- **Breaking:** u64/usize ToSql/FromSql changes, statement caching optional, SQLite 3.34.1 min
- **Code changes:** None (we use bundled SQLite and don't use u64/usize conversions)
- **Tests:** Passed

### rand: 0.8 → 0.10
- **Breaking:** thread_rng() → rng(), gen_range() → random_range(), Rng → RngExt, features renamed
- **Code changes:**
  - `src/interactive.rs`: `use rand::Rng` → `use rand::RngExt`
  - `src/interactive.rs`: `rand::thread_rng()` → `rand::rng()`
  - `src/interactive.rs`: `.gen_range()` → `.random_range()`
  - `Cargo.toml`: features changed from `["std", "std_rng"]` to `["std", "thread_rng"]`
- **Tests:** Passed

### criterion: 0.5 → 0.8 (dev-dependency)
- **Breaking:** `criterion::black_box` deprecated in favor of `std::hint::black_box`
- **Code changes:**
  - `benches/heredoc_perf.rs`: Switched to `use std::hint::black_box`
  - `benches/regex_automata_comparison.rs`: Switched to `use std::hint::black_box`
- **Tests:** Passed

### which: 7.0 → 8.0 (dev-dependency)
- **Breaking:** Minor API changes
- **Code changes:** None
- **Tests:** Passed

## Skipped

### serde_yaml: 0.9 (deprecated)
- **Reason:** Crate is deprecated; successor is `serde_yml` or `serde_yaml_ng`. Migration requires evaluating replacements. Logged for future work.

### vergen-gix: 10.0.0-beta.5 (pre-release)
- **Reason:** Already on latest pre-release. Stable max is 9.1.0. Staying on beta track.

### rich_rust: 0.2.0
- **Reason:** Already on latest stable version.

## Needs Attention

### serde_yaml deprecation
- The `serde_yaml` crate (0.9.x) is officially deprecated. Consider migrating to `serde_yml` or `serde_yaml_ng` in a future session.

## Validation
- `cargo fmt --check`: Passed
- `cargo clippy --all-targets -- -D warnings`: Passed
- `cargo test --lib`: 1935/1935 passed

---

# Round 2 — 2026-04-20

## Inventory — /dp-developed libraries

| Crate | Current in Cargo.toml | Latest local (/dp) | Latest on crates.io | Action |
|-------|------------------------|--------------------|-----------------------|--------|
| `fsqlite` | 0.1.2 | 0.1.2 | 0.1.2 | already latest |
| `fsqlite-types` | 0.1.2 | 0.1.2 | 0.1.2 | already latest |
| `fsqlite-error` | 0.1.2 | 0.1.2 | 0.1.2 | already latest |
| `rich_rust` | 0.2.0 | 0.2.1 | 0.2.1 | bump → 0.2.1 |
| `toon-rust` | 0.1.3 | renamed to `tru` 0.2.2 | `toon-rust` 0.1.3 (stale), `tru` 0.2.2 | migrate to `tru` 0.2.2 |

## Round 2 Updates

### rich_rust: 0.2.0 → 0.2.1 (local /dp)
- **Breaking:** None (patch bump — bug fixes: remove nightly-only feature gate, preserve Tree label spans, wide-char overflow in narrow columns, gutter overflow in narrow columns, clamp interactive max_length, CRLF in LiveWriter, Python fixtures parity, cursor positioning)
- **Code changes:** None
- **Tests:** Passed

### toon-rust 0.1.3 → tru 0.2.2 (local /dp; crate was renamed)
- **Breaking:**
  - Package name `toon-rust` → `tru` (crate lib name is `toon`)
  - `encode(&serde_json::Value, Option<EncodeOptions>) -> Result<String>` → `encode(impl Into<JsonValue>, Option<EncodeOptions>) -> String` (infallible)
  - `decode(&str, Option<DecodeOptions>) -> Result<serde_json::Value>` → `try_decode(&str, Option<DecodeOptions>) -> Result<JsonValue>` (returns crate's own JsonValue)
  - Numbers roundtrip as `f64` (integer/float distinction is lossy in decode)
- **Code changes:**
  - `Cargo.toml`: `toon-rust = "0.1.3"` → `tru = "0.2.2"`
  - `src/cli.rs` (test output encoder): `toon_rust::encode(&json, None).expect(...)` → `toon::encode(json, None)`
  - `src/cli.rs` (roundtrip test): updated to use `toon::try_decode` + `.into::<serde_json::Value>()` and compare canonically (normalize Numbers to f64) since the new lib is lossy for integer vs float distinction.
- **Tests:** Passed (2210/2210)

### Semver-compatible updates via `cargo update` (Cargo.lock only)
Pulled ~60 transitive/direct updates forward including:
tokio 1.49.0 → 1.52.1, clap 4.5.60 → 4.6.1, clap_complete 4.5.66 → 4.6.2,
toml 1.0.3 → 1.1.2, toml_edit 0.25.3 → 0.25.11, chrono 0.4.43 → 0.4.44,
proptest 1.10.0 → 1.11.0, rayon 1.11.0 → 1.12.0, rand 0.10.0 → 0.10.1,
semver 1.0.27 → 1.0.28, tempfile 3.25.0 → 3.27.0, console 0.16.2 → 0.16.3,
once_cell 1.21.3 → 1.21.4, tracing-subscriber 0.3.22 → 0.3.23,
vergen-gix 10.0.0-beta.5 → 10.0.0-beta.6, which 8.0.0 → 8.0.2,
rustls 0.23.36 → 0.23.38, and more (see Cargo.lock).
- **Tests:** Passed

### Pre-existing clippy fixes (surfaced by newer nightly clippy)
Not strictly dep-related but blocking `cargo clippy -D warnings`:
- `src/heredoc.rs` (quoted-delimiter parser): collapsed two `if let Some(end)` blocks using `?` operator on `find()`
- `src/packs/mod.rs` (compound-command splitter): merged identical `&&`/`||` branches into a single OR condition

## Round 2 Skipped list — all cleared in Round 3 below

## Validation (Round 2)
- `cargo check --all-targets`: Passed
- `cargo clippy --all-targets -- -D warnings`: Passed
- `cargo test --lib`: 2210/2210 passed

---

# Round 3 — 2026-04-20 (Deferred major bumps — done carefully one at a time)

### sha2: 0.10 → 0.11
- **Breaking:** `Digest::finalize()` now returns `hybrid_array::Array<u8, U32>` instead of `GenericArray<u8, U32>`. The new `Array` type does not implement `LowerHex`, so `format!("{:x}", hash)` no longer works directly on the digest output.
- **Code changes:**
  - `tests/update_rollback.rs`: hex-encode digest byte-by-byte (`digest.iter().fold(...)` with `write!(s, "{:02x}", b)`) instead of `format!("{:x}", digest)`.
- **Tests:** update_rollback 20/20 + pending_exceptions 32/32 passed.

### hmac: 0.12 → 0.13
- **Breaking:** `Hmac<D>::new_from_slice` is now provided by the `KeyInit` trait and requires `use hmac::KeyInit;` (in 0.12 it worked via the `Mac` trait).
- **Code changes:**
  - `src/pending_exceptions.rs`: `use hmac::{Hmac, Mac};` → `use hmac::{Hmac, KeyInit, Mac};`.
- **Tests:** pending_exceptions 32/32 passed.

### ast-grep-core / ast-grep-language: 0.40 → 0.42
- **Breaking:** None (for our usage surface — `AstGrep`, `Pattern`, `SupportLang`, recursive `Doc`/`Node` traversal).
- **Code changes:** None.
- **Tests:** heredoc 130/130 + ast_matcher 79/79 passed.

### self_update: 0.42 → 0.44
- **Breaking:**
  - Enabling the `rustls` feature no longer transitively pulls in an HTTP client. You must now explicitly enable one of `reqwest` or `ureq`. Without either, the code references `http_client::get` (absent) and fails to compile.
- **Code changes:**
  - `Cargo.toml`: `features = ["rustls", ...]` → `features = ["reqwest", "rustls", ...]`.
- **Tests:** update 46/46 + update_rollback 20/20 passed.

### rust-mcp-sdk: 0.8.3 → 0.9.0
- **Breaking:**
  - `ToolInputSchema::new(required, properties, ...)` — `properties` type changed from `Option<HashMap<String, Map<String, Value>>>` to `Option<BTreeMap<String, Map<String, Value>>>` (deterministic iteration order).
  - `McpServerOptions<T>` gained a new required field `message_observer: Option<Arc<dyn McpObserver<...>>>`.
- **Code changes:**
  - `src/mcp.rs` (`tool_input_schema`): `HashMap::new()` → `std::collections::BTreeMap::new()`.
  - `src/mcp.rs` (`run_mcp_server_async`): added `message_observer: None` to the `McpServerOptions { ... }` initializer.
  - `src/mcp.rs`: removed now-unused `use std::collections::HashMap;`.
- **Smoke test (actual binary):** stdio MCP server handshake verified end-to-end:
  - `initialize` → server returns capabilities, name `dcg`, version `0.4.3`, protocol `2025-11-25`
  - `tools/list` → returns the three tools (`check_command`, `scan_file`, `explain_pattern`) with correct JSON schemas
- **Tests:** all lib tests 2210/2210 passed.

## Validation (Round 3 — all five deferred bumps applied)
- `cargo check --all-targets`: Passed
- `cargo clippy --all-targets -- -D warnings`: Passed
- `cargo test --lib`: 2210/2210 passed
- `cargo build --release`: Passed
- End-to-end binary smoke tests (hook deny, hook allow, `dcg test`, `dcg mcp-server`): All passed
