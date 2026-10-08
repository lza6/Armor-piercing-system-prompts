<!--
name: "Data: SDK set model system prompt field"
description: "Schema description for the internal set_model system_prompt field, including render timing, non-empty updates, feature overrides, and compatibility behavior"
ccVersion: "2.1.265"
-->
@internal Replaces the custom system prompt (the --system-prompt / initialize systemPrompt slot). With the system prompt recorded (the default) it takes effect at the next render point (compaction), or from the next turn under systemPromptSnapshot: false. Applied only when the model request is accepted; must be non-empty (there is no revert-to-built-in form); re-send the current model for a prompt-only update. The CLAUDE_CODE_SYSTEM_PROMPT_GB_FEATURE per-turn read, where configured, still wins. Transports that do not implement it, and older builds, ack success without applying it.
