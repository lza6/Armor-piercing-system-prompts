# ARM64 Image Verification

This document describes the manual procedure to verify that a published `asklokesh/loki-mode` Docker image actually runs on a real ARM64 host. CI does not exercise the runtime on ARM64; see `docs/UNREACHABLE-TESTS.md` for the rationale.

This procedure should be run by a maintainer (or any contributor with an ARM64 host) after each release that bumps the published image tag.

---

## Prerequisites

- An ARM64 host. Examples:
  - Apple Silicon Mac (M1, M2, M3, or M4).
  - AWS Graviton instance.
  - Raspberry Pi 4 or 5 with a 64-bit OS.
- Docker Engine or Docker Desktop installed and running.
- Network access to Docker Hub.

Confirm the host is genuinely ARM64:

```bash
uname -m
# expected output: arm64 (macOS) or aarch64 (Linux)
```

If the output is anything else (`x86_64`, `amd64`), this procedure does not apply -- you need an ARM64 host.

---

## Procedure

Replace `7.X.Y` with the tag you intend to verify.

```bash
docker run --rm --platform linux/arm64 asklokesh/loki-mode:7.X.Y loki version
```

The `--platform linux/arm64` flag forces Docker to pull the ARM64 variant of the manifest list even if a multi-arch image is being served. This catches the case where the image was published as x86_64-only by accident.

### Expected output

The command prints the version string of the bundled CLI and exits 0. The exact format follows the bash CLI's `version` command. A non-zero exit, a Python or Bun runtime error, or a missing-binary error all indicate a broken ARM64 build.

### Additional smoke checks

Run a few more commands to surface ARM64-specific runtime issues:

```bash
docker run --rm --platform linux/arm64 asklokesh/loki-mode:7.X.Y loki doctor
docker run --rm --platform linux/arm64 asklokesh/loki-mode:7.X.Y loki status
```

`loki doctor` prints the runtime environment summary (Bun version, Python version, provider CLIs detected). On the published image, the absence of any provider CLI is expected; the doctor command itself should still complete without an internal error.

`loki status` reads `.loki/` state. With no mounted state directory, it should report `stopped` or an equivalent empty state -- not crash.

---

## Recording the result

When verification passes, note it in the release commit body or in the GitHub Release notes under a "Verified channels" section. Include:

- Host architecture (`uname -m` output)
- Host OS (`uname -s` plus distribution if Linux)
- Docker version (`docker --version`)
- Image tag verified
- Date of verification

When verification fails, open an issue with the same details plus the full output of the failing command. Do not promote the release to "stable" until the failure is fixed.

---

## Why this is manual

The `parity-drift.yml` and image-publish workflows use `buildx` with QEMU emulation, which produces a multi-arch manifest but does not exercise the runtime on real ARM64 silicon. QEMU translation can mask runtime issues that only surface on real hardware (for example, native binary dependencies, mmap layout differences, or timing-sensitive code).

Until the project provisions an ARM64 self-hosted runner, this manual procedure is the only honest verification.
