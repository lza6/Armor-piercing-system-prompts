# Docker channel verification (S-P1-DOCKER)

Local, no-push verification of the reworked multi-arch Docker channel. Run via
`scripts/verify-channel-docker.sh` on macOS arm64 (Apple Silicon) with Docker
Desktop 29.6.1 + buildx v0.35.0 (qemu emulation for amd64).

- Branch: `feat/p1-docker` (base `2dd47764` + the doctor/warmup fixes below)
- Host: macOS arm64, Docker Desktop 29.6.1, buildx v0.35.0-desktop.2
- Run date: 2026-07-16 (UTC)
- Script result: **PASS** (exit 0, all 9 checks)

## Contract

- **default (slim):** browser-engine OS libraries baked at build as root; the
  browser binary + on-device models download on first use into the `/data`
  volume (`PLAYWRIGHT_BROWSERS_PATH=/data/browsers`). Runs as `USER node`.
- **full:** browser binary preinstalled at build into an image-baked path
  (`/opt/browsers`), for `--rm` / no-volume use.
- No image-level `HEALTHCHECK` (stdio MCP default has no HTTP endpoint).
- **No `sudo`, no `python3` in the image.** Two src fixes make the truly-slim
  image healthy:
  - `src/cli/doctor.ts` — the python+docker "runtime" degradation is a
    search-engine-sidecar prerequisite; now gated on `searxngConfigured()`
    (same D5 gate as the searxng section), so a runtime-less core-backend
    container is doctor-green.
  - `src/cli/warmup.ts` — `detectDepsStrategy` catches the sudo spawn ENOENT
    (runCommand REJECTS on spawn errors) and returns the `skip` strategy
    instead of crashing the browser install before its launch smoke-test.

## Image sizes (arm64 native, amd64 emulated)

| Image | Target | Size |
|-------|--------|------|
| `wigolo-verify:default`       | default (slim) arm64 | **1.32 GB** |
| `wigolo-verify:full`          | full arm64           | **2.32 GB** |
| `wigolo-verify:default-amd64` | default (slim) amd64 | **1.67 GB** |

(~60 MB below the earlier sudo+python3 draft.) `node_modules` (~759 MB,
playwright + onnxruntime + transformers) appears in the image exactly once; the
browser-engine OS libraries add ~370 MB. An earlier draft that used `FROM deps`
+ a second `COPY` + `chown -R /app` triplicated the node_modules layer (default
ballooned to 3.15 GB) — fixed by building `base` from a clean slim image and
copying with `--chown` (no `chown -R` layer).

## Per-check outcomes

| Check | Result | Notes |
|-------|--------|-------|
| `build:arm64:default`      | PASS | multi-stage build, `--load` (no push) |
| `build:arm64:full`         | PASS | browser binary preinstalled |
| `doctor:arm64:default`     | PASS | `Overall: OK`, exit 0 — WITHOUT python/docker in the image |
| `warmup:arm64:default`     | PASS | lazy browser download + baked-libs launch as `node`, sudo absent |
| `sudo-absent:arm64:default`| PASS | raw spawn = ENOENT; warmup still ok (skip strategy) |
| `fetch:arm64:default`      | PASS | react.dev rendered via browser tier, real markdown |
| `doctor:arm64:full`        | PASS | `Overall: OK`, exit 0 |
| `fetch:arm64:full`         | PASS | rendered with preinstalled browser (no download) |
| `doctor:amd64:default`     | PASS | boot-level doctor under qemu emulation, `Overall: OK`, exit 0 |

### doctor green on the truly-slim image (no python, no docker)

```
  Writable:      yes
[wigolo doctor] Runtime:
  Python 3:      not available
  Docker:        not available
...
[wigolo doctor] Overall: OK        (exit 0)
```

The runtime lines still report absence for transparency, but under the default
core backend they no longer degrade — the check fires only when the
search-engine sidecar backend is configured (covered by unit tests both ways).

### First-run UX — lazy browser download as USER node, sudo absent (trimmed)

Running `warmup --browser --json` as the non-root `node` user on the slim image
with a fresh `/data` volume. This is what a user sees on the first JS-render:

```
whoami=node uid=1000
[wigolo warmup] Starting wigolo warmup
[wigolo warmup] Installing browser engine (chromium)...
[wigolo warmup] playwright installed
[wigolo warmup] Search engine sidecar: skipped — using multi-engine core backend
[wigolo warmup] Summary:
[wigolo warmup]   Browser:       ok
{"playwright":"ok","searxng":"skipped"}
--- (exit 0) ---
```

Proves: browser binary downloads lazily into the volume, the baked OS libraries
let the launch smoke-test pass as non-root, and warmup exits 0 — with NO sudo
binary in the image.

### sudo-absent live behavior (ENOENT proof)

The image ships no `sudo`. A raw probe from inside the container as `node`:

```
sudo_status=null spawn_error=ENOENT
```

`spawn_error=ENOENT` confirms sudo is truly absent and that a spawn surfaces
ENOENT (which `runCommand` turns into a REJECTION — the pre-fix crash path).
The warmup above runs the real `detectDepsStrategy` over this exact condition
and completes `{"playwright":"ok"}`, exit 0: the fix maps the rejection to the
`skip` strategy, and the launch smoke-test then passes on the baked libraries.

### JS-render fetch (organic, reuses warmed binary)

```
multi-browser pool initialized {"types":["chromium"],"strategy":"round-robin"}
{
  "url": "https://react.dev",
  "title": "React",
  "markdown": "The library for web and native user interfaces ..."
}
```

The browser tier launches, react.dev renders, and content extracts — using the
binary downloaded lazily into the volume in the warmup step. Same fetch passes
on the full variant with its preinstalled binary.

### amd64 emulated

The amd64 default image built and booted `doctor` under qemu, reporting
`Overall: OK` (exit 0). Only boot-level doctor is exercised under emulation (no
first-use model/browser download under qemu); the browser lazy-download path is
verified natively on arm64.

## Reproduce

```bash
scripts/verify-channel-docker.sh
# Env overrides: IMAGE_BASE (default wigolo-verify), RENDER_URL (default react.dev)
```

The script builds both targets with `buildx --load` (never `--push`), starts
from a clean volume so the browser download is a genuine first-use, and exits 0
only if every required check passes. amd64 emulation is best-effort: reported
UNVERIFIED (not FAIL) if unavailable on the host.
