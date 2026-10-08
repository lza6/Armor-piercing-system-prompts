---
name: "mkl-debug-ml-training"
description: "Investigate training failures, NaNs, missing gradients, misleading losses, and non-reproducible runs in PyTorch, Lightning, or TensorFlow/Keras. Use for a concrete training bug or regression, not an open-ended architecture or hyperparameter search."
---

# Debug a training run

Turn the reported symptom into an observable failure on a bounded input. Preserve the user's framework, backend, and intended training semantics. A CPU or eager-mode reproduction can help isolate a bug, but does not establish what happens on the original accelerator, compiled graph, or distributed setup.

## Establish the failing boundary

Record the command, revision, Python and framework versions, imported package paths, device, dtype, precision, and execution mode. For a resume or reproducibility issue, include the data order, seed handling, checkpoint, optimizer, scheduler, and gradient-scaler state that affect the failing step. Classify dependency/import failures separately from a reproduced training bug.

Reduce to a fixed batch while retaining the relevant shapes, label semantics, masks, and value ranges. Inspect the tensors actually entering the loss. Compare their shapes to that loss's contract: class indices, probability targets, and scalar regression targets have different requirements. Do not apply a blanket reshape or squeeze to silence an error.

For example, scalar predictions of shape `[N, 1]` minus labels of shape `[N]` produce an `[N, N]` residual through broadcasting. With predictions `[[1], [3]]` and labels `[1, 3]`, the raw mean squared residual is `2` despite perfect per-example predictions. Aligning these scalar labels to `[N, 1]` makes it `0`. This is a bug in that loss expression; built-in losses may validate or normalize shapes differently.

Trace finite values through inputs, outputs, loss, gradients, and updated parameters. Find the first unexpected value or disconnected gradient. Clipping, replacing NaNs, or reducing the learning rate may hide the symptom; they are fixes only when supported by the diagnosed cause.

## Use the framework's relevant controls

Read the matching section; do not migrate frameworks to simplify the investigation.

### PyTorch

Check whether the failing path needs training behavior, gradient recording, or both. `model.eval()` changes layers such as dropout and batch normalization; it does not disable autograd. Conversely, `no_grad()` does not select evaluation behavior.

Inspect unexpected `detach()`, `.item()`, tensor reconstruction, and optimizer parameter membership when gradients are absent. Distinguish intentionally unused parameters from disconnected ones. Track gradient clearing and accumulation at optimizer-step boundaries. Use anomaly detection narrowly when a backward failure needs a forward traceback. If disabling AMP or compilation isolates the issue, recheck the proposed fix under the original mode.

Seed controls do not promise identical results across releases or CPU/GPU execution. Deterministic algorithms can cost performance or reject unsupported operations. See [PyTorch reproducibility](https://docs.pytorch.org/docs/stable/notes/randomness.html).

### Lightning

`Trainer(fast_dev_run=True)` is useful for a loop smoke test, but disables loggers and checkpoint/early-stopping callbacks. For bugs involving those features, keep the affected configuration and bound work with integer `limit_train_batches` and `limit_val_batches` instead.

Identify automatic versus manual optimization before changing backward calls, optimizer stepping, or accumulation. Account for sanity validation before training when interpreting hook order and counts. Overfitting one small batch is a diagnostic for the learning path, not evidence of generalization. See [Lightning debugging controls](https://lightning.ai/docs/pytorch/stable/debug/debugging_basic.html).

### TensorFlow / Keras

Record whether the code uses `tf.keras` or standalone `keras`, and the actual backend. Keras can use backends other than TensorFlow; `tf.GradientTape` advice applies to a TensorFlow path.

Check that forward and loss operations occur inside the tape, variables are watched, and NumPy conversions have not severed differentiation. Inspect `None` gradients before filtering or applying them. `compile(run_eagerly=True)` can expose intermediate values during debugging; restore graph/XLA settings to verify a graph-specific failure. See [Keras debugging tips](https://keras.io/examples/keras_recipes/debugging_tips/).

For determinism investigations, consider `tf.keras.utils.set_random_seed` and `tf.config.experimental.enable_op_determinism()` alongside data order and environment equivalence. These controls do not guarantee matching results across TensorFlow versions. See [TensorFlow determinism requirements](https://www.tensorflow.org/api_docs/python/tf/config/experimental/enable_op_determinism).

## Compare the baseline and fix

Start each comparison from equivalent model and optimizer state with the same batch and unchanged behavioral assertion. Check the expected intermediate result and parameter update, not merely that training exits successfully. Choose numerical tolerances from the operation and precision; do not loosen them after seeing the candidate result.

Report the minimal command, environment, baseline failure, candidate outcome, and modes actually exercised. For stochastic symptoms, use a bounded repeated comparison and report variation instead of choosing a favorable run. Separate a verified reproduction from claims about model quality, full training, accelerator behavior, or agent performance.
