---
name: ruview-applications
description: Run RuView sensing applications — presence/occupancy, breathing & heart rate, activity & fall detection, 17-keypoint pose estimation (WiFlow), sleep monitoring & apnea screening, environment mapping, Mass Casualty Assessment (MAT), and the 3D point-cloud fusion demo. Use when someone wants to actually *do* something with a working RuView setup.
allowed-tools: Bash Read Write Edit Glob Grep
---

# RuView Applications

What RuView can sense, and how to run each one. Assumes you have either the Docker demo (simulated CSI) or a live ESP32 sink (see `ruview-quickstart` / `ruview-hardware-setup`).

## Application catalogue

| Application | What it does | Entry point |
|-------------|--------------|-------------|
| **Presence / occupancy** | Detect people through walls, count them, track entries/exits (trained model + PIR fusion, ~0.012 ms latency) | sensing-server live mode; `examples/environment/` |
| **Vital signs** | Breathing 6–30 BPM (bandpass 0.1–0.5 Hz), heart rate 40–120 BPM (bandpass 0.8–2.0 Hz), contactless while sleeping/sitting | `wifi-densepose-vitals` crate (ADR-021); `examples/medical/` |
| **Activity recognition** | Walking, sitting, gestures, falls — from temporal CSI patterns | RuvSense `gesture.rs` (DTW), `pose_tracker.rs`; `scripts/gait-analyzer.js` |
| **Pose estimation** | 17 COCO keypoints via WiFlow architecture; dual-modal webcam+WiFi fusion demo | `cargo run -p wifi-densepose-sensing-server` + pose-fusion demo (ADR-059); see `ruview-model-training` to train |
| **Sleep monitoring** | Overnight monitoring, sleep-stage classification, apnea screening | `examples/sleep/`; `scripts/apnea-detector.js` |
| **Environment mapping** | RF fingerprinting identifies rooms, detects moved furniture, spots new objects | sensing-server `--build-index env`; RuvSense `field_model.rs`, `cross_room.rs` |
| **Mass Casualty Assessment (MAT)** | Disaster survivor detection — find people in rubble/smoke | `wifi-densepose-mat` crate; `docs/wifi-mat-user-guide.md`; `examples/medical/` |
| **3D point cloud** *(optional fusion)* | Camera depth (MiDaS) + WiFi CSI + mmWave radar → unified spatial model (~22 ms, 19K+ pts/frame) | `scripts/mmwave_fusion_bridge.py`; ADR-094 (GitHub Pages deploy) |
| **Novel RF apps** | Passive radar, material classification, device fingerprinting, mincut person-counting | `scripts/passive-radar.js`, `material-classifier.js`, `device-fingerprint.js`, `mincut-person-counter.js` (ADR-077/078) |

## Quick recipes

```bash
# Docker demo — everything, simulated CSI
docker run -p 3000:3000 ruvnet/wifi-densepose:latest    # http://localhost:3000

# Live sensing server (consumes ESP32 UDP CSI)
cd v2 && cargo run -p wifi-densepose-sensing-server

# Live RF room scan (Cognitum Seed on :5006)
node scripts/rf-scan.js --port 5006
node scripts/snn-csi-processor.js --port 5006

# Embed a trained model + build an environment index
cd v2
cargo run -p wifi-densepose-sensing-server -- --model model.rvf --embed
cargo run -p wifi-densepose-sensing-server -- --model model.rvf --build-index env

# Python live demo
python examples/ruview_live.py

# Spectrogram / graph visualisers
node scripts/csi-spectrogram.js
node scripts/csi-graph-visualizer.js
```

## Picking the right modality

- **Through a wall, no line of sight** → presence + activity; expect ≤5 m depth (Fresnel-zone geometry).
- **Person stationary (sleeping / sitting)** → vitals (breathing first, heart rate needs cleaner signal) + sleep staging.
- **Need skeletons** → pose (WiFlow). Camera-free works but is modest; camera-supervised gets 92.9% PCK@20 — train it (`ruview-model-training`).
- **Search & rescue** → MAT (`docs/wifi-mat-user-guide.md`).
- **"What changed in this room?"** → environment mapping / RF fingerprint index.
- **Best spatial accuracy** → 2+ ESP32 nodes + cross-viewpoint fusion (`ruview-advanced-sensing`), optionally + Cognitum Seed.

## Examples directory map

`examples/environment/` · `examples/medical/` · `examples/sleep/` · `examples/stress/` · `examples/happiness-vector/` · `examples/ruview_live.py` — each has a README.

## Reference

- `README.md` — feature matrix, latency/throughput numbers
- `docs/user-guide.md`, `docs/wifi-mat-user-guide.md`
- ADRs: 021 (vitals), 024 (AETHER contrastive embeddings), 027 (MERIDIAN domain generalization), 041 (edge modules), 059 (live ESP32 pipeline), 077/078 (novel RF apps), 082 (pose tracker output filter), 094 (point cloud)
- RuvSense modules: `v2/crates/wifi-densepose-signal/src/ruvsense/` (14 modules)
