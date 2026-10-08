---
name: ruview-configure
description: Configure RuView — ESP32 sdkconfig variants, NVS provisioning, WiFi channel / MAC filter overrides (ADR-060), edge intelligence modules (ADR-041), sensing-server flags, multi-node mesh, and Cognitum Seed integration. Use when adjusting how a deployed RuView system behaves without changing code.
allowed-tools: Bash Read Write Edit Glob Grep
---

# RuView Configuration

Everything you can tune in a RuView deployment, from a one-line provision flag to a full mesh + Cognitum Seed setup.

## 1. Firmware build-time config (sdkconfig)

| Variant | File | When |
|---------|------|------|
| 8MB (default) | `firmware/esp32-csi-node/sdkconfig.defaults.template` | ESP32-S3 8MB, full feature set, real WiFi CSI |
| 4MB | `firmware/esp32-csi-node/sdkconfig.defaults.4mb` | ESP32-S3 SuperMini 4MB — display disabled, dual OTA slots (`partitions_4mb.csv`, ~1.856 MB each) |
| Heltec N16R2 | `firmware/esp32-csi-node/sdkconfig.defaults.heltec_n16r2` | Heltec boards |

Switch: `cp firmware/esp32-csi-node/sdkconfig.defaults.<variant> firmware/esp32-csi-node/sdkconfig.defaults`, then rebuild (see `ruview-hardware-setup`). **Never test in mock mode** — the Kconfig fall-threshold bug only showed up with real CSI.

## 2. Runtime device config (NVS via provision.py)

`provision.py` writes the `csi_cfg` NVS namespace over the serial port. **Run `python firmware/esp32-csi-node/provision.py --help` for the authoritative flag list** (on Windows force `PYTHONUTF8=1 PYTHONIOENCODING=utf-8` — the help text contains non-ASCII and crashes under cp1252).

```bash
python firmware/esp32-csi-node/provision.py --port COM8 \
  --ssid "WiFi" --password "secret" \
  --target-ip 192.168.1.20 --target-port 5005 \   # aggregator UDP sink (port default 5005)
  --node-id 1 \                                   # 0-255
  --channel 6 --filter-mac AA:BB:CC:DD:EE:FF       # ADR-060: pin channel + filter transmitter
```

| Flag group | Flags | Notes |
|------------|-------|-------|
| WiFi / sink | `--ssid` `--password` `--target-ip` `--target-port` (5005) `--node-id` | `--node-id` 0-255 |
| TDM mesh | `--tdm-slot` `--tdm-total` | 0-based slot index + total node count — this is how multi-node mesh is slotted |
| Edge processing | `--edge-tier {0,1,2}` | 0=off, 1=stats, 2=vitals (ADR-041) |
| Detection thresholds | `--pres-thresh` (50) `--fall-thresh` (15000 → 15.0 rad/s²) | raise `--fall-thresh` to cut false falls in high-traffic areas (issue #263) |
| Vitals | `--vital-win` (300 frames) `--vital-int` (1000 ms) `--subk-count` (32, top-K subcarriers) | |
| Channel / hopping | `--channel` (1-14 / 36-177, overrides AP auto-detect) `--filter-mac` `--hop-channels` (`1,6,11`) `--hop-dwell` (200 ms) | omit `--channel` + set `--hop-channels` for ADR-061 multi-freq hopping; omit `--filter-mac` to capture all transmitters |
| Cognitum Seed | `--seed-url` (`http://10.1.10.236`) `--seed-token` (Bearer, from pairing) `--zone` (`lobby`) | |
| Swarm | `--swarm-hb` (30 s) `--swarm-ingest` (5 s) | heartbeat + vector ingest intervals |
| Mode | `--dry-run` (build NVS bin, don't flash) `--baud` (460800) `--force-partial` | |

> ⚠️ **NVS namespace is replaced wholesale (issue #391).** Flashing rewrites the *entire* `csi_cfg` namespace — **any key you don't pass on the CLI is erased**. Always pass the full set you want, or use `--force-partial` knowingly. Read the device's current values off the serial boot log first (`adaptive_ctrl` / `csi_collector` lines) if you're unsure.

- NVS partition images for fleet provisioning: `scripts/generate_nvs_matrix.py` (subprocess-first — the `esp_idf_nvs_partition_gen` API changed across versions).

## 3. Sensing server flags

```bash
cd v2
cargo run -p wifi-densepose-sensing-server -- --help

# Common modes:
cargo run -p wifi-densepose-sensing-server                                  # live sink, default port
cargo run -p wifi-densepose-sensing-server -- --pretrain --dataset data/csi/ --pretrain-epochs 50
cargo run -p wifi-densepose-sensing-server -- --train --dataset data/mmfi/ --epochs 100 --save-rvf model.rvf
cargo run -p wifi-densepose-sensing-server -- --model model.rvf --embed
cargo run -p wifi-densepose-sensing-server -- --model model.rvf --build-index env
```

`wifiscan` server (multi-BSSID, ADR-022): `cargo run -p wifi-densepose-sensing-server` consumes `wifi-densepose-wifiscan` output; use neighbour APs as free radar illuminators.

## 4. Edge intelligence modules (ADR-041)

Small Rust/WASM programs that run on the ESP32 itself — no internet, instant response. See `docs/edge-modules/` and `docs/adr/ADR-041-*`. Each module declares its CSI feature inputs (8-dim feature vectors) and an RVF store target (Cognitum Seed). Configure which modules ship in a build via the firmware component config; configure their thresholds via NVS keys.

Helper scripts that mirror edge-module logic on the host (useful for tuning before flashing):
`scripts/apnea-detector.js`, `gait-analyzer.js`, `material-classifier.js`, `passive-radar.js`, `mincut-person-counter.js`, `device-fingerprint.js`, `mesh-graph-transformer.js`, `material-detector.js`.

## 5. Multi-node mesh

- 2+ nodes give real spatial resolution. Each node provisioned to the same `--target-ip` sink.
- TDM protocol + channel hopping coordinated by `wifi-densepose-hardware` (`v2/crates/wifi-densepose-hardware/src/esp32/`).
- Cross-viewpoint fusion combines nodes — see `ruview-advanced-sensing`.

## 6. Cognitum Seed integration ($140 total BOM)

ESP32 streams CSI → bridge forwards to a Cognitum Seed for persistent RVF memory, kNN over environments, and an Ed25519 witness chain.

```bash
node scripts/rf-scan.js --port 5006              # live RF room scan → Seed
node scripts/snn-csi-processor.js --port 5006    # SNN real-time learning on-Seed
```

See `docs/tutorials/cognitum-seed-pretraining.md` and ADR-028 (capability audit + witness verification).

## 7. App-level config

- API: `wifi-densepose-api` (Axum) — config via `wifi-densepose-config` crate; see `example.env` / `pyproject.toml` for the v1 Python service.
- Docker: `docker run -p 3000:3000 ruvnet/wifi-densepose:latest` (env-var overrides documented in `README.md` / `docker/`).
- Dashboard: served on `:3000`; nvsim dashboard (ADR-092) is separate.

## Reference

- `docs/adr/` (96 ADRs) — esp. ADR-022 (wifiscan), ADR-028 (capability audit), ADR-041 (edge modules), ADR-060 (channel/MAC override), ADR-061 (QEMU + mesh), ADR-081 (adaptive CSI mesh kernel)
- `CLAUDE.md` / `CLAUDE.local.md` — crate map, build env, QEMU CI fixes
- `example.env`, `Makefile`, `firmware/esp32-csi-node/`
