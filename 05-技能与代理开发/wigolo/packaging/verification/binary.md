# S-P1-BIN — Single-file binary channel verification

**Product integration of the validated @yao-pkg/pkg recipe** (validation gate:
`feat/p1-pkg-validation:packaging/pkg-validation/report.md`). The round-2 CJS-bundle
recipe passed everything except sqlite-vec; this slice wires the recipe into the
product build and fixes the sqlite-vec dylib extraction. All ten assertions now PASS.

## Verdict: **PASS (10/10)** — macOS arm64 (verified here)

Other platforms: **documented-unverified-until-CI** (see the dormant matrix in
`.github/workflows/binary-build.yml`). Gatekeeper: needs release-time notarization.

## Environment

| Item | Value |
|---|---|
| Base SHA | `2dd47764` (branch `feat/p1-binary`) |
| Host | macOS (Darwin), Apple Silicon arm64 |
| Node | v22.14.0 |
| pkg | **@yao-pkg/pkg@6.21.0** (exact-pinned devDependency) |
| esbuild | **0.28.0** (exact-pinned devDependency) |
| pkg target | `node22-macos-arm64` |
| wigolo version | `0.1.43-beta.2` (binary `--version` == package.json — depth shims OK) |
| Bundle | single-file **CJS**, 14.7 MB (esbuild) |
| Final binary | **197 MB** (206,920,176 bytes), Mach-O arm64 |
| Clean data dir | fresh `mktemp -d` per assertion, `WIGOLO_DATA_DIR` set (never reads `~/.wigolo`) |

Reproduce: `npm run build:binary` then `scripts/verify-channel-binary.sh`.

## Build pipeline (`npm run build:binary`)

1. `npm run build` — fresh tsup + tsc dist. The dist entry (`dist/index.js`) now
   carries **no top-level await** (src/index.ts exports an async `main()`), so it
   is directly esbuild-able to CommonJS — no throwaway text-transform entry
   (the round-2 recipe's Step 1 generator is replaced by the real refactor).
2. `packaging/binary/bundle.mjs` — esbuild `--bundle --format=cjs --platform=node
   --target=node22`, externalizing native/asset carriers + the Ink stack, aliasing
   `react-devtools-core` to a no-op stub, and rewriting `import.meta.url` to
   `pathToFileURL(__filename)` via a banner shim. Output: `dist/cli/agents/wigolo.bundle.cjs`.
3. `packaging/binary/pack.sh` — drops the package.json depth shims, copies the pkg
   config to the repo root, runs pkg for the host target, cleans up the root
   artifacts. Output: `packaging/binary/dist/wigolo` (gitignored).

### Depth-shim vs normalize — CHOICE: **depth shims** (least code)

The bundle collapses module depth, so the four `import.meta.url`-relative
`package.json` lookups in src resolve from a single location only if the bundle
sits at the right depth. Chosen placement (validated): bundle at
`dist/cli/agents/wigolo.bundle.cjs` + package.json copies at `dist/package.json`
and `dist/cli/package.json`:

- `../../..` (`cli/agents/utils.ts`, `cli/tui/version.ts`) → repo-root package.json + real `assets/` ✓
- `../..` (`cli/status.ts`, `cli/help.ts`) → `dist/package.json` shim ✓
- `..` (`server.ts`) → `dist/cli/package.json` shim ✓ (MCP serverInfo reports the true version)

This adds ZERO source changes over the pre-authorized entry refactor — the
alternative (normalizing all four lookup depths in src) would touch five files
for no runtime benefit.

## Per-assertion results (fresh 197 MB binary, clean `WIGOLO_DATA_DIR`)

| # | Assertion | Result |
|---|---|---|
| a | `--help` exit 0 | **PASS** |
| b | `--version` == package.json (`0.1.43-beta.2`) | **PASS** — 0.0.0 fallback not triggered |
| c | `doctor --json` parses | **PASS** — status=ok, exit 0, `install_channel:"binary"` |
| d | cache write + sqlite-vec probe | **PASS** — was the round-2 FAIL layer |
| e | `warmup --embeddings` (ORT .node + sibling dylib) | **PASS** — BGE-small, dim=384 |
| f | `mcp` handshake + ListTools | **PASS** — 10 tools, serverInfo version correct |
| g | `warmup --browser` (playwright cli spawn seam) | **PASS** — `playwright installed`, `Browser: ok` |
| h | Gatekeeper / quarantine | **DOCUMENTED** — unsigned binary blocked by GK |

### (d) sqlite-vec — the round-2 FAIL layer, now PASS

Round 2 failed with:
```
warn "sqlite-vec extension failed to load — vector search disabled"
dlopen(/snapshot/.../vec0.dylib.dylib): no such file
```
Fix (src/cache/db.ts): under a packaged binary (`isPackagedBinary()`), copy the
extension out of the `/snapshot` VFS to `<dataDir>/native/vec0.dylib` and
`db.loadExtension(realPath)` from there. Passing the full existing `.dylib` path
also stops SQLite re-suffixing to the doubled `.dylib.dylib`.

Live evidence (this run):
- No `sqlite-vec extension failed` / `vector search disabled` / `dylib.dylib` /
  `no such table: vec_id_map` on stderr.
- 7 `vec_*` virtual tables present in the DB (`vec_documents`, `vec_documents_info`,
  `vec_documents_chunks`, `vec_documents_rowids`, `vec_documents_vector_chunks00`, …)
  — the 001-sqlite-vec migration's `CREATE VIRTUAL TABLE ... USING vec0(...)`
  succeeded, which is only possible with the extension loaded.
- `<dataDir>/native/vec0.dylib` extracted (161,896 bytes, byte-size matches source).

NPM/source path is unchanged — the copy is strictly guarded on `isPackagedBinary()`;
unit tests (`tests/unit/cache/db-vec-packaged.test.ts`) assert the non-packaged path
never creates `native/`.

### (h) Gatekeeper — documentation only

```
$ cp wigolo /tmp/wigolo-quarantined
$ xattr -w com.apple.quarantine "0081;;;Safari" /tmp/wigolo-quarantined
$ /tmp/wigolo-quarantined --version
→ BLOCKED (operation not permitted)
```
The unsigned binary with the quarantine xattr is blocked from executing.
Distribution via download requires code-signing + notarization (release-time,
secrets-gated in the CI matrix), or `xattr -d com.apple.quarantine` by the user.
`curl` downloads set no quarantine attr, so curl-install is unaffected.

## TUI decision (D8 headless-first)

The Ink TUI stack cannot boot inside the binary (dependency-level top-level await
in ink/yoga — externalized in the bundle). Inside the binary, `init --wizard` and
the `config` TUI print an actionable fallback and route to the headless flow:

> interactive wizard unavailable in the standalone binary — use the flag-driven
> `wigolo init` (works fully headless) or run via npm (`npx wigolo init --wizard`)

Headless `init`, `config --plain`, `doctor`, and all 10 tools work in the binary
(verified: doctor exit 0, MCP 10 tools, fetch/find_similar/warmup).

## Cross-platform note

The pkg config (`packaging/binary/pkg.config.json`) pins the **macOS-arm64** native
asset set (better-sqlite3 `.node`, `onnxruntime-node/bin/napi-v3/darwin/arm64/*`,
`sqlite-vec-darwin-arm64/vec0.dylib`, keyring darwin-arm64, wreq darwin-arm64,
tokenizers darwin-universal, `@img/sharp-*darwin-arm64`). Per-platform asset lists
for linux-x64 / linux-arm64 / win-x64 are authored in the dormant CI workflow and
are **UNVERIFIED until CI runs** (a broad ORT glob would triple binary size by
bundling all three platforms' ~209 MB of ORT runtimes — keep it per-platform).

## Machine-cache caveat (unchanged from round 2)

`browser:ok` / `warmup --browser` partly reflect the machine-level
`~/Library/Caches/ms-playwright` cache, which lives OUTSIDE `WIGOLO_DATA_DIR`.
The cli-spawn seam itself is proven (the round-2 `./lib/program` miss was fixed by
shipping `playwright/lib/**/*.js`), but a truly cold browser download path is only
fully exercised on a CI runner with no ms-playwright cache.
