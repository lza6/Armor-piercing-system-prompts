# npm channel verification (S-P1-MATRIX)

Local, no-push verification of the npm channel (`npx` / `npm i -g`). Run via
`scripts/verify-channel-npm.sh`, which packs a local tarball and installs it
into a **throwaway prefix** — the real global tree is never touched.

## Verdict: **PASS (2/2)** — macOS arm64 (verified here)

## Environment

| Item | Value |
|---|---|
| Base SHA | `019a64b2` (branch `feat/p1-matrix`) |
| Host | macOS 15.7.7 (Darwin 24.6.0), Apple Silicon arm64 |
| Node | v22.14.0 |
| npm | 10.9.2 |
| Package | `wigolo@0.1.43-beta.2` |
| Throwaway prefix | fresh `mktemp -d` — `NPM_CONFIG_PREFIX` set (real global untouched) |
| Data dir | fresh `mktemp -d`, `WIGOLO_DATA_DIR` set per run |

Reproduce: `scripts/verify-channel-npm.sh` (exit 0 on pass).

## Pipeline

1. `npm run build` when `dist/` is missing — there is no `prepack`/`prepare`
   script, so a bare `npm pack` on a clean checkout would ship a tarball
   without `dist/`. The published npm tarball is built by release CI; the
   script mirrors that by building locally.
2. `npm pack` → local `wigolo-0.1.43-beta.2.tgz` in a temp dir.
3. `NPM_CONFIG_PREFIX=<temp> npm install -g <tarball>` — 385 packages,
   `prebuild-install` resolves the native `better-sqlite3` binding (no
   from-source node-gyp compile).
4. Assertions against `<prefix>/bin/wigolo`.
5. `cleanup` trap: `npm uninstall -g wigolo` from the throwaway prefix, then
   `rm -rf` every temp dir.

## Per-assertion results

| # | Assertion | Result |
|---|---|---|
| 1 | `<prefix>/bin/wigolo --version` reports the package.json version | **PASS** — prints `wigolo 0.1.43-beta.2` (0.0.0 fallback not triggered) |
| 2 | `<prefix>/bin/wigolo doctor` exit 0 (fresh `WIGOLO_DATA_DIR`) | **PASS** — `Overall: OK`, exit 0 (post-D5 lazy contract) |

### doctor green on a fresh data dir

```
[wigolo doctor] Core sqlite-vec:
  extension:     loaded (vec_version v0.1.9)
[wigolo doctor] Local cache:
  urls:          0 (cache empty — populate via fetch/crawl)
[wigolo doctor] Background embedding queue:
  pending:       0 (idle)
[wigolo doctor] Overall: OK
```

The native cache subsystem (sqlite-vec extension + local cache + embedding
queue) loads at runtime from the packed tarball, confirming the prebuilt
`better-sqlite3` binding resolved.

## Version-output note

`wigolo --version` prints `wigolo <version>` (name + version), not the bare
version string. The assertion matches the version as a substring — the same
intent as the sibling channel reports' "`--version` == package.json" cells.

## Notes

- `--version` reporting the true version (not the `0.0.0` fallback) also
  confirms the packed tarball's `package.json` lookup resolves — the same
  concern the binary channel handles via depth shims. On npm/source the file
  layout is unchanged, so no shim is needed.
- Windows: npm/npx is the covered install path on Windows (there is no native
  installer). Not exercised on this macOS host — the pipeline is
  platform-agnostic (npm pack + npm i -g + prebuilt binding), so Windows npm
  is **documented-covered, CI-verified via the standard npm test matrix**.

## Gate results

- `shellcheck scripts/verify-channel-npm.sh` → clean, exit 0.
- Script run → exit 0, both assertions PASS, real global tree untouched.
