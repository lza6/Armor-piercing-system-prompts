# Homebrew formula verification

Local verification of `packaging/homebrew/wigolo.rb` on macOS (Apple Silicon).
Run via `scripts/verify-channel-brew.sh`.

## Environment

| Field | Value |
|---|---|
| Date | 2026-07-15 |
| Host | Darwin 24.6.0, arm64 (Apple Silicon) |
| Homebrew | 6.0.9 |
| node@22 (brew) | v22.23.1 |
| Package version | 0.1.43-beta.2 |

## Working invocation

Homebrew 6.x **refuses to install a formula from a bare file path** — it must
live in a tap (`Error: Homebrew requires formulae to be in a tap`). The verify
script works around this with an ephemeral local tap:

```sh
brew tap-new --no-git wigolo-verify/local
# write temp formula (file://<tarball> + real sha256) to its Formula/wigolo.rb
brew install --build-from-source wigolo-verify/local/wigolo
```

Full sequence (what the script does):

1. `npm pack --pack-destination <tmp>` → local tarball. NOTE: `dist/` must be
   built first (`npm run build`) — there is no `prepack`/`prepare` script, so a
   bare `npm pack` on a clean checkout would ship a tarball without `dist/`.
   The published npm tarball is built by release CI; local verification builds
   it manually to mirror that.
2. Temp formula in a local tap: `url` → `file://<tarball>`, `sha256` → the
   tarball's real digest. The committed formula keeps the canonical registry
   URL + `PLACEHOLDER_REFRESHED_AT_RELEASE`.
3. `brew install --build-from-source wigolo-verify/local/wigolo`.
4. Assertions (below).
5. `brew uninstall --force wigolo` + `brew untap wigolo-verify/local`.

Run it with: `HOMEBREW_NO_AUTO_UPDATE=1 HOMEBREW_NO_ENV_HINTS=1 scripts/verify-channel-brew.sh`

## Results

| Check | Result |
|---|---|
| `brew install` | PASS — installed to `/opt/homebrew/Cellar/wigolo/0.1.43-beta.2` (24,613 files, 727.5 MB), ~50s (node@22 already poured) |
| `$(brew --prefix)/bin/wigolo --version` == package.json | PASS — reports `0.1.43-beta.2` |
| `wigolo doctor` exit code | PASS — exit 0, `Overall: OK` (post-D5 lazy contract) |
| better-sqlite3 binding | PREBUILT (see below) |
| `brew uninstall` + `untap` | PASS — machine left tidy; node@22 kept |

Doctor cache-subsystem sections (proves the native binding loads at runtime):

```
[wigolo doctor] Core sqlite-vec:
  extension:     loaded (vec_version v0.1.9)
[wigolo doctor] Local cache:
  urls:          191
[wigolo doctor] Background embedding queue:
  pending:       0 (idle)
```

## better-sqlite3: PREBUILT (not node-gyp source build)

Under brew's node@22 (v22.23.1, ABI 127), better-sqlite3's `install` script
(`prebuild-install || node-gyp rebuild --release`) resolves via
**`prebuild-install`** — it fetches the prebuilt `.node` for the node@22 ABI.
Verified by the presence of `build/Release/better_sqlite3.node` with **no `*.o`
object files** under `build/` (a source compile leaves object files behind).

## Formula defect found and fixed during verification

The first formula draft used the plain `std_npm_args` (which defaults to
`--ignore-scripts`). That skipped every native postinstall, so
`better_sqlite3.node` was never produced and doctor reported *"Could not locate
the bindings file"* for sqlite-vec, local cache, and the embedding queue — the
cache subsystem would be dead at runtime.

Flipping to `std_npm_args(ignore_scripts: false)` then failed the whole install
(brew exit 1) because the macOS-only optional dep **`fsevents`** has no
`binding.gyp` and its `node-gyp rebuild` errors out
(`gyp: binding.gyp not found ... fsevents`).

Final fix (in the committed formula): install the tree with the std
`--ignore-scripts` (so fsevents cannot abort the install), then **explicitly**
build the one native module the cache needs:

```ruby
system "npm", "install", *std_npm_args
cd node_modules/"better-sqlite3" do
  system "npm", "run", "install"
end
```

This is verified green end-to-end by the run above.

## Deviations / notes

- The spec files named in the task brief
  (`docs/superpowers/specs/2026-07-15-p1-distribution-channels-design.md` and
  `docs/superpowers/plans/2026-07-15-p1-distribution.md`) do **not exist** at
  base SHA 2dd47764 nor on any branch. Implemented strictly from the task
  brief's formula contract + runbook spec.
- Homebrew flagged the toolchain as a "Tier 2 configuration" and warned a newer
  Command Line Tools release is available. Neither affected the outcome — the
  install completed and better-sqlite3 used a prebuilt (no local compile
  needed). Not an unverified cell.
- `packaging/install.sh` (referenced by the runbook step 4) is owned by the
  sibling `feat/p1-installer` slice and is not present here; the runbook
  documents the copy-to-`site/public/` step as a dormant release action.
