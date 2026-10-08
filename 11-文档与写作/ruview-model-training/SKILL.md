---
name: ruview-model-training
description: Train RuView models — camera-free WiFlow pose (10 sensor signals, no labels), camera-supervised pose (MediaPipe + ESP32 CSI → 92.9% PCK@20, ADR-079), RuVector contrastive embeddings (AETHER, ADR-024), domain generalization (MERIDIAN, ADR-027), local SNN environment adaptation, plus GPU training on GCloud and Hugging Face publishing. Use when building, fine-tuning, evaluating, or shipping a model.
allowed-tools: Bash Read Write Edit Glob Grep
---

# RuView Model Training

RuView trains several kinds of model. Pick the track that matches the goal; all of them run on a laptop, with an optional GPU path.

## Track A — Camera-free pose (WiFlow), no cameras, no labels

Trains 17-keypoint pose from 10 sensor signals. Fast, fully unsupervised, modest accuracy.

```bash
cd v2
# Pretrain on raw CSI (contrastive)
cargo run -p wifi-densepose-sensing-server -- --pretrain --dataset data/csi/ --pretrain-epochs 50
# Train pose head, save an RVF artifact
cargo run -p wifi-densepose-sensing-server -- --train --dataset data/mmfi/ --epochs 100 --save-rvf model.rvf
```

~84 s on an M4 Pro. Benchmarks: `node scripts/benchmark-wiflow.js`, eval: `node scripts/eval-wiflow.js`.

## Track B — Camera-supervised pose (ADR-079) → 92.9% PCK@20

Uses a webcam + MediaPipe as ground truth, paired with ESP32 CSI. ~19 min on a laptop.

```bash
# 1. Collect paired data (camera + CSI)
python scripts/collect-ground-truth.py        # MediaPipe pose landmarks
python scripts/collect-training-data.py       # CSI capture, time-synced
node scripts/align-ground-truth.js            # align camera ↔ CSI timestamps

# 2. Train (the camera-supervised path through the sensing-server / train crate)
cd v2
cargo run -p wifi-densepose-sensing-server -- --train --dataset data/paired/ --epochs <N> --save-rvf model.rvf

# 3. Evaluate
cd .. && node scripts/eval-wiflow.js          # reports PCK@20
```

Requires `data/pose_landmarker_lite.task` (MediaPipe model). See `docs/adr/ADR-079-camera-ground-truth-training.md`.

## Track C — RuVector contrastive embeddings (AETHER, ADR-024)

CSI subcarrier amplitude/phase → embeddings for re-ID and retrieval (171K emb/s on M4 Pro). Driven by `wifi-densepose-train` + `wifi-densepose-ruvector` (RuVector v2.0.4). Spectrogram embeddings: ADR-076.

```bash
cd v2
cargo check -p wifi-densepose-train --no-default-features      # sanity
cargo run -p wifi-densepose-sensing-server -- --model model.rvf --embed
cargo run -p wifi-densepose-sensing-server -- --model model.rvf --build-index env
```

## Track D — Domain generalization (MERIDIAN, ADR-027)

Make a model transfer across environments without retraining. Configured through the training pipeline's domain-generalization options; see ADR-027 and `wifi-densepose-train` + `ruview_metrics`.

## Track E — Local SNN environment adaptation

Spiking neural network that adapts to a new room in <30 s, on-device or on a Cognitum Seed:

```bash
node scripts/snn-csi-processor.js --port 5006
```

See `docs/tutorials/cognitum-seed-pretraining.md`, ADR-084/085 (RaBitQ similarity sensor), ADR-086 (edge novelty gate).

## GPU training on GCloud

Project `cognitum-20260110` has L4 / A100 / H100 quota.

```bash
gcloud auth login
gcloud config set project cognitum-20260110

bash scripts/gcloud-train.sh --dry-run                      # smoke test, synthetic data
bash scripts/gcloud-train.sh --gpu l4 --hours 2             # prototyping
bash scripts/gcloud-train.sh --gpu a100 --config scripts/training-config-sweep.json
bash scripts/gcloud-train.sh --sweep                        # full hyperparameter sweep
# VM is auto-deleted after training unless --keep-vm. Cost: L4 ~$0.80/hr, A100 40GB ~$3.60/hr.
```

Local Mac training: `bash scripts/mac-mini-train.sh`. Model benchmark: `python scripts/benchmark-model.py`.

## Publishing a trained model

```bash
python scripts/publish-huggingface.py        # or: bash scripts/publish-huggingface.sh
```

Pushes the RVF artifact + card to Hugging Face. See `docs/huggingface/`.

## Data layout

| Path | Contents |
|------|----------|
| `data/recordings/` | Raw CSI captures (`*.csi.jsonl`), overnight runs |
| `data/csi/` | CSI datasets for pretraining |
| `data/mmfi/` | MM-Fi dataset (ADR-015) |
| `data/paired/` | Camera ↔ CSI paired samples (ADR-079) |
| `data/ground-truth/` | MediaPipe pose landmarks |
| `data/pose_landmarker_lite.task` | MediaPipe model file |
| `models/` | Trained artifacts |

Record more data: `python scripts/record-csi-udp.py` (UDP CSI capture from a live node).

## Validation after a training change

```bash
cd v2 && cargo test --workspace --no-default-features          # 1,400+ pass, 0 fail
cd .. && python archive/v1/data/proof/verify.py                # VERDICT: PASS
```

Then hand off to `ruview-verify` for the witness bundle.

## Reference

- ADRs: 015 (MM-Fi + Wi-Pose datasets), 016 (RuVector training integration — complete), 017 (RuVector signal + MAT), 024 (AETHER), 027 (MERIDIAN), 076 (spectrogram embeddings), 079 (camera ground truth), 084/085 (RaBitQ), 095/096 (on-ESP32 temporal modeling, sparse GQA)
- Crates: `wifi-densepose-train`, `wifi-densepose-nn`, `wifi-densepose-ruvector`, `wifi-densepose-sensing-server`
- `scripts/gcloud-train.sh`, `mac-mini-train.sh`, `benchmark-wiflow.js`, `eval-wiflow.js`, `benchmark-model.py`
