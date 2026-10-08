---
name: modal-compute
description: Run GPU workloads on Modal's serverless infrastructure. Use when the user needs remote GPU compute for training, inference, benchmarks, or batch processing and Modal CLI is available.
---

# Modal Compute

Use the `modal` CLI for serverless GPU workloads. No pod lifecycle to manage — write a decorated Python script and run it.

## Setup

```bash
pip install modal
modal setup
```

## Commands

| Command | Description |
|---------|-------------|
| `modal run script.py` | Run a script on Modal (ephemeral) |
| `modal run --detach script.py` | Run detached (background) |
| `modal deploy script.py` | Deploy persistently |
| `modal serve script.py` | Serve with hot-reload (dev) |
| `modal shell --gpu a100` | Interactive shell with GPU |
| `modal app list` | List deployed apps |

## GPU types

`T4`, `L4`, `A10G`, `L40S`, `A100`, `A100-80GB`, `H100`, `H200`, `B200`

Multi-GPU: `"H100:4"` for 4x H100s.

## Script pattern

```python
import modal

app = modal.App("experiment")
image = modal.Image.debian_slim(python_version="3.11").pip_install("torch==2.8.0")

@app.function(gpu="A100", image=image, timeout=600)
def train():
    import torch
    # training code here

@app.local_entrypoint()
def main():
    train.remote()
```

## When to use

- Stateless burst GPU jobs (training, inference, benchmarks)
- No persistent state needed between runs
- Check availability: `command -v modal`
