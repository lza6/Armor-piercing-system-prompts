---
name: ruview-advanced-sensing
description: Advanced RuView capabilities — RuvSense multistatic sensing (attention-weighted fusion, geometric diversity, persistent field model), cross-viewpoint fusion across multiple nodes, RF tomography (ISTA L1 solver, voxel grids), longitudinal biomechanics drift, pre-movement intention signals, adversarial signal detection, and multistatic mesh security hardening. Use for research-grade or multi-node deployments.
allowed-tools: Bash Read Write Edit Glob Grep
---

# RuView Advanced Sensing

The deep end: multistatic mesh, tomography, persistent field models, and the security model that protects them. Most of this lives in `wifi-densepose-signal/src/ruvsense/` (14 modules) and `wifi-densepose-ruvector/src/viewpoint/` (5 modules).

## RuvSense multistatic mode (ADR-029)

Treat every WiFi link in range — including neighbours' APs — as a bistatic radar pair, then fuse them.

| Module (`signal/src/ruvsense/`) | Purpose |
|--------------------------------|---------|
| `multiband.rs` | Multi-band CSI frame fusion, cross-channel coherence |
| `phase_align.rs` | Iterative LO phase-offset estimation, circular mean |
| `multistatic.rs` | Attention-weighted fusion, geometric diversity |
| `coherence.rs` / `coherence_gate.rs` | Z-score coherence scoring; Accept / PredictOnly / Reject / Recalibrate gate decisions |
| `pose_tracker.rs` | 17-keypoint Kalman tracker with AETHER re-ID embeddings |
| `field_model.rs` | SVD room eigenstructure, perturbation extraction |
| `tomography.rs` | RF tomography, ISTA L1 solver, voxel grid |
| `longitudinal.rs` | Welford stats, biomechanics drift detection |
| `intention.rs` | Pre-movement lead signals (200–500 ms ahead) |
| `cross_room.rs` | Environment fingerprinting, transition graph |
| `gesture.rs` | DTW template-matching gesture classifier |
| `adversarial.rs` | Physically-impossible-signal detection, multi-link consistency |

## Cross-viewpoint fusion (ADR-016 viewpoint module)

Combine 2+ nodes geometrically — more nodes, more independent looks, tighter localization.

| Module (`ruvector/src/viewpoint/`) | Purpose |
|------------------------------------|---------|
| `attention.rs` | CrossViewpointAttention, GeometricBias, softmax with `G_bias` |
| `geometry.rs` | GeometricDiversityIndex, Cramér–Rao bounds, Fisher Information |
| `coherence.rs` | Phase-phasor coherence, hysteresis gate |
| `fusion.rs` | MultistaticArray aggregate root, domain events |

Host-side helpers to explore the geometry before deploying: `node scripts/mesh-graph-transformer.js`, `node scripts/passive-radar.js`, `node scripts/deep-scan.js`.

## Persistent field model (ADR-030)

`field_model.rs` builds an SVD eigenstructure of the room and stores it (RVF, ideally on a Cognitum Seed). New CSI frames are projected against it; the residual *is* the perturbation. Lets you ask "what's different from the empty-room baseline?" and survive restarts.

## RF tomography

`tomography.rs` reconstructs a voxel occupancy grid from the multistatic link set via an ISTA L1 solver (sparse — most voxels are empty). Use with cross-viewpoint geometry for through-wall volumetric imaging. RuVector solver crates back the sparse interpolation (114→56 subcarriers).

## Sensing-first RF mode & adaptive mesh kernel

- ADR-031 (RuView sensing-first RF mode), ADR-081 (adaptive CSI mesh firmware kernel), ADR-083 (per-cluster π compute hop), ADR-095/096 (on-ESP32 temporal modeling with sparse GQA attention — runs the temporal head on-device).

## Security (ADR-032 — multistatic mesh hardening)

Using neighbours' APs as illuminators and pooling links across a mesh expands the attack surface. Mitigations:
- `adversarial.rs` rejects physically impossible signals and cross-checks multi-link consistency.
- `coherence_gate.rs` quarantines low-coherence / suspicious links (Reject / Recalibrate).
- Ed25519 witness chain (ADR-028) attests every measurement.
- Run a security review when touching anything on the hardware/network boundary (see `ruview-verify` and `docs/security-audit-wasm-edge-vendor.md`).

## Validate advanced changes

```bash
cd v2 && cargo test --workspace --no-default-features      # incl. ruvsense + viewpoint tests
cargo test -p wifi-densepose-signal --no-default-features
cargo test -p wifi-densepose-ruvector --no-default-features
cd .. && python archive/v1/data/proof/verify.py
```

## Reference

- ADRs: 014 (SOTA signal processing), 029 (multistatic mode), 030 (persistent field model), 031 (sensing-first RF), 032 (mesh security hardening), 081/083/095/096
- `v2/crates/wifi-densepose-signal/src/ruvsense/` · `v2/crates/wifi-densepose-ruvector/src/viewpoint/`
- `docs/research/`, `docs/security-audit-wasm-edge-vendor.md`
