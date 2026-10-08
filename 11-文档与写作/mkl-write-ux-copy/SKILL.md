---
name: "mkl-write-ux-copy"
description: "Write or revise interface labels, errors, empty states, and confirmation text from actual product behavior. Use for UI microcopy with clear next actions, preserved localization tokens, and explicit length constraints."
---

# Write interface copy

Identify the surface, user action, current state, actual failure or success, and available next actions. Inspect the relevant UI and handlers when available. A button label must describe what clicking it does; an error message must not imply an action the product cannot perform.

Preserve string keys, interpolation tokens, markup, accessible names, and plural syntax unless the user explicitly includes a structural change. For ICU or another message format, inspect the format before editing branches. Ask about an unknown product behavior when it determines whether the copy would be misleading; style choices usually do not need clarification.

For errors, explain the actionable problem without blaming the user and give a supported recovery step. For an empty state, explain what belongs there and how to add it. For an irreversible action, make its actual consequence clear. Do not add guarantees about stored, recovered, or deleted data without implementation evidence.

Fit the requested tone, locale, and length budget. If only a character limit is given, state the counting convention when it could affect acceptance. Measure the final string, accounting for dynamic values separately; a short template does not prove every rendered message fits. Check the real layout when available and report when it was not inspected.

Return the final strings mapped to their keys or components. Keep rationale outside production copy. Apply file changes when requested and preserve the host format.

## Worked example

Context: an upload over `{max_mb}` MB is rejected; the user can choose another file. No automatic compression or retry exists.

Before: "Oops! Something went wrong with your file."

Suitable error: "This file exceeds {max_mb} MB. Choose a smaller file."

Suitable button: "Choose file"

Acceptance: the limit token remains exact and recovery matches the available action. No claim that a failed upload was saved. This is an authored example, not a recorded client evaluation.
