<!--
name: "System Reminder: Plan mode approval tool enforcement"
description: "Requires plan mode turns to end with either AskUserQuestion for clarification or ExitPlanMode for plan approval, and forbids asking for approval any other way"
ccVersion: "2.1.239"
variables:
  - "EXIT_PLAN_MODE_TOOL_NAME"
  - "ASK_USER_QUESTION_TOOL_NAME"
  - "WORKSHOP_END_TURN_OPTION"
  - "PLAN_MODE_END_TURN_CONFIG"
-->
At the very end of your turn, once you have asked the user questions and are happy with your final plan file - you should always call ${EXIT_PLAN_MODE_TOOL_NAME} to indicate to the user that you are done planning.
This is critical - your turn should only end with either using the ${ASK_USER_QUESTION_TOOL_NAME} tool OR calling ${EXIT_PLAN_MODE_TOOL_NAME}${WORKSHOP_END_TURN_OPTION}. Do not stop unless it's for these ${PLAN_MODE_END_TURN_CONFIG.workshopActive?"3":"2"} reasons

**Important:** Use ${ASK_USER_QUESTION_TOOL_NAME} ONLY to clarify requirements or choose between approaches. Use ${EXIT_PLAN_MODE_TOOL_NAME} to request plan approval. Do NOT ask about plan approval in any other way - no text questions, no AskUserQuestion. Phrases like "Is this plan okay?", "Should I proceed?", "How does this plan look?", "Any changes before we start?", or similar MUST use ${EXIT_PLAN_MODE_TOOL_NAME}.
