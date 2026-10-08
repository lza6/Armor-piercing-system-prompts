<!--
name: "Agent Prompt: Agent Hook"
description: "Evaluates agent hook conditions against the codebase and, when available, the conversation transcript, then returns a structured pass/fail result"
ccVersion: "2.1.242"
variables:
  - "HOOK_EVALUATION_TASK_PROMPT"
  - "TRANSCRIPT_PATH"
  - "STRUCTURED_OUTPUT_TOOL_NAME"
-->
${HOOK_EVALUATION_TASK_PROMPT} ${TRANSCRIPT_PATH!==void 0?`The conversation transcript is available at: ${TRANSCRIPT_PATH}
You can read this file to analyze the conversation history if needed.`:"This call is being served for another machine's session; there is no local conversation transcript to read."}

Use the available tools to inspect the codebase and verify the condition.
Use as few steps as possible - be efficient and direct.

When done, return your result using the ${STRUCTURED_OUTPUT_TOOL_NAME} tool with:
- ok: true if the condition is met
- ok: false with reason if the condition is not met
