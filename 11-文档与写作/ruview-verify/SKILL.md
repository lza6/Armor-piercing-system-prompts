---
name: ruview-verify
description: Verify a RuView build — full Rust workspace tests, the deterministic Python pipeline proof (SHA-256 Trust Kill Switch), firmware hash manifest, and the ADR-028 witness bundle with one-command self-verification. Use after any significant change, before merging a PR, or to produce an attestation bundle for a recipient.
allowed-tools: Bash Read Write Edit Glob Grep
---

# RuView Verification & Witness Bundle

The trust pipeline for RuView. Run this after meaningful changes and before merging.

## 1. Rust workspace tests

```bash
cd v2
cargo test --workspace --no-default-features        # must be 1,400+ passed, 0 failed (~2 min)
```

Single-crate checks (no GPU): `cargo check -p wifi-densepose-train --no-default-features`, `cargo test -p wifi-densepose-signal --no-default-features`, etc.

## 2. Deterministic Python proof (Trust Kill Switch)

Feeds a reference CSI signal through the **production** pipeline and hashes the output. Any behavioural drift changes the hash.

```bash
cd ..
python archive/v1/data/proof/verify.py              # must print VERDICT: PASS
```

If it fails on a hash mismatch after a legitimate numpy/scipy bump:
```bash
python archive/v1/data/proof/verify.py --generate-hash
python archive/v1/data/proof/verify.py
```

Artifacts: `archive/v1/data/proof/verify.py`, `expected_features.sha256`, `sample_csi_data.json` (1,000 synthetic frames, seed=42).

## 3. Python test suite (v1)

```bash
cd archive/v1 && python -m pytest tests/ -x -q
```

## 4. Generate the witness bundle (ADR-028)

```bash
bash scripts/generate-witness-bundle.sh
```

Produces `dist/witness-bundle-ADR028-<sha>.tar.gz` containing:
- `WITNESS-LOG-028.md` — 33-row attestation matrix, evidence per capability
- `ADR-028-esp32-capability-audit.md` — full audit findings
- `proof/verify.py` + `expected_features.sha256` — the deterministic proof
- `test-results/rust-workspace-tests.log` — full cargo test output
- `firmware-manifest/source-hashes.txt` — SHA-256 of all 7 ESP32 firmware files
- `crate-manifest/versions.txt` — all 15 crates + versions
- `VERIFY.sh` — one-command self-verification for recipients

## 5. Self-verify the bundle

```bash
cd dist/witness-bundle-ADR028-*/
bash VERIFY.sh                                       # must be 7/7 PASS
```

## Pre-merge checklist (from CLAUDE.md)

1. Rust tests pass (1,400+, 0 fail)
2. Python proof passes (VERDICT: PASS)
3. `README.md` updated if scope changed (platform/crate/hardware tables, feature summaries)
4. `CLAUDE.md` updated if scope changed (crate table, ADR list, module tables, version)
5. `CHANGELOG.md` — entry under `[Unreleased]`
6. `docs/user-guide.md` updated if new data sources / CLI flags / setup steps
7. ADR index — bump ADR count in README docs table if a new ADR was added
8. Witness bundle regenerated if tests or proof hash changed
9. Docker Hub image rebuilt only if Dockerfile / deps / runtime behaviour changed
10. Crate publishing only if a published crate's public API changed (publish in dependency order — see CLAUDE.md)
11. `.gitignore` updated for new build artifacts/binaries
12. Security review for new modules touching hardware/network boundaries

## Security scan

```bash
npx @claude-flow/cli@latest security scan            # after security-related changes
```

Also see `docs/security-audit-wasm-edge-vendor.md`, `docs/qe-reports/`, ADR-080 (QE remediation plan), ADR-093 (dashboard gap analysis).

## QEMU firmware CI (ADR-061)

11-job workflow ("Firmware QEMU Tests"). Local QEMU helpers: `scripts/qemu-esp32s3-test.sh`, `qemu-mesh-test.sh`, `qemu-chaos-test.sh`, `qemu-snapshot-test.sh`, `install-qemu.sh`. Notes: `espressif/idf:v5.4` container needs `source $IDF_PATH/export.sh` before `pip`; QEMU needs `esptool merge_bin --fill-flash-size 8MB`; WARNs (no real WiFi) are treated as OK in CI.

## Reference

- `docs/WITNESS-LOG-028.md`, `docs/adr/ADR-028-esp32-capability-audit.md`
- `scripts/generate-witness-bundle.sh`, `archive/v1/data/proof/verify.py`
- `CLAUDE.md` → "Validation & Witness Verification" + "Pre-Merge Checklist"
- `CLAUDE.local.md` → QEMU CI pipeline fixes
