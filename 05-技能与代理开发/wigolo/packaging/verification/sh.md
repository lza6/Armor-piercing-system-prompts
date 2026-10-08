# Channel verification — curl|sh installer (S-P1-SH)

Slice: S-P1-SH (spec D2). Branch: `feat/p1-installer`. Base: `2dd47764`.
Verified on: macOS arm64 (darwin/arm64), this machine.

Runner: `scripts/verify-channel-sh.sh` — offline of the npm registry (installs
from a local `npm pack` tarball via `WIGOLO_INSTALL_SOURCE`); the pinned Node
runtime IS fetched from nodejs.org and checksum-verified by the installer.

## Pinned runtime

- Node **v22.23.1** (LTS "Jod") — `NODE_VERSION` at the top of `packaging/install.sh`.
- Tarball + `SHASUMS256.txt` confirmed present on nodejs.org/dist for all four
  target platforms (darwin/linux × x64/arm64). Installer fetches the arch
  tarball, greps its line from `SHASUMS256.txt`, and verifies sha256
  fail-closed before unpack.

## Results (all PASS)

| Check | Outcome |
|-------|---------|
| `shellcheck -s sh packaging/install.sh` | clean, exit 0 |
| `shellcheck scripts/verify-channel-sh.sh` | clean, exit 0 |
| `npm pack` tarball built + installed via `WIGOLO_INSTALL_SOURCE` | OK |
| Runtime downloaded + sha256-verified against SHASUMS256.txt | OK (checksum matched) |
| Shim `~/.wigolo/bin/wigolo` created, mode 0755, executable | OK |
| `wigolo --version` == package.json (0.1.43-beta.2) | OK (matched) |
| `wigolo doctor` exit 0 (post-D5 lazy contract) | OK ("Overall: OK") |
| Re-run install (upgrade path) converges, no error | OK (runtime reused, tool reinstalled) |
| `install.sh --uninstall` removes bin/ tool/ runtime/ | OK (all three gone) |
| `--uninstall` PRESERVES simulated user data (cache/, config.json) | OK (both preserved) |

## Cross-platform pre-grep (owned files)

- CRLF: none — all files LF-only (`packaging/install.sh`,
  `scripts/verify-channel-sh.sh`, `.gitattributes`, `src/cli/uninstall.ts`,
  `tests/unit/cli/uninstall.test.ts`).
- Mode bits: `install.sh` and `verify-channel-sh.sh` both `0755`.
- Bashisms in `install.sh`: none (shellcheck `-s sh` passes; POSIX `set -eu`,
  no pipefail; no `[[`, `local`, arrays, `$(())` outside comments).
- HOME-only env: `$HOME` used only as the fallback for the
  `WIGOLO_INSTALL_PREFIX` override.

## Capability-language grep

`grep -niE 'playwright|onnx|fastembed|searxng|chromium|sqlite|...' packaging/install.sh`
→ CLEAN. User-facing text uses capability language ("bundled runtime",
"browser engine" via package-runner phrasing, "search engine"). No library
names. `src/cli/uninstall.ts` user strings also clean.

## uninstall.ts semantics

`wigolo uninstall` now detects the bootstrap layout (presence of
`<dataDir>/tool` or `<dataDir>/runtime`) and prints layout-aware cleanup
guidance:

- **npm/source layout:** full cleanup is a single `rm -rf <dataDir>`.
- **bootstrap layout:** distinguishes "remove the tool"
  (`install.sh --uninstall`, or `rm -rf <bin> <tool> <runtime>` — keeps
  cache/models/keys) from "wipe everything" (`rm -rf <dataDir>`, which ALSO
  deletes the tool). `--help` documents both paths unconditionally.

Unit tests: `tests/unit/cli/uninstall.test.ts` (4 tests, TDD — written failing
first). All pass.

## Baseline noise (not caused by this slice)

`tests/unit/cli/doctor.test.ts` "LLM fallback section" — 2 failures
(GEMINI_API_KEY / provider-listing) reproduce identically on the clean base
SHA `2dd47764` without this slice's changes. Pre-existing machine-env noise
per MEMORY (documented 4-6 LLM-provider baseline failures). `tsc --noEmit`
clean; the rest of the `tests/unit/cli` lane (1324 tests) passes.
