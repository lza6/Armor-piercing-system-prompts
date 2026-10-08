---
name: simple-pipeline
skill: ascii-diagram
description: User asks for a 3-box pipeline diagram — should use grid engine
tags: [happy-path, pipeline, basic]
input: draw me an ascii diagram of a pipeline: Input -> Process -> Output
state: clean
assert-contains:
  - "+"
  - "---"
  - Input
  - Process
  - Output
assert-not-contains:
  - I cannot
  - ENOENT
---

## What This Tests

The most common use case: a simple left-to-right pipeline with 3 boxes.
The skill should produce a properly aligned box diagram, either using the
grid engine or the inline class for a 3-node diagram.

## Expected Behavior

1. Skill identifies this as a simple horizontal pipeline (3 boxes, linear)
2. Produces an aligned ASCII box diagram in a fenced code block
3. Boxes contain the labels: Input, Process, Output
4. Boxes connected by arrows (---> or similar)
5. No misaligned corners or broken arrows
