<!--
name: "Tool Description: Edit single replacement"
description: "Tool description for performing exact string replacement in a file, including prior-read and line-prefix requirements"
ccVersion: "2.1.236"
variables:
  - "SHOULD_USE_OUTSIDE_WORKING_DIRECTORY_READ_NOTE"
  - "READ_TOOL_NAME"
  - "LINE_NUMBER_PREFIX_FORMAT"
-->
Performs exact string replacement in a file.
${SHOULD_USE_OUTSIDE_WORKING_DIRECTORY_READ_NOTE?`
- If the file is outside the working directory, you must ${READ_TOOL_NAME} it in this conversation before editing, or the call will fail.`:`
- You must ${READ_TOOL_NAME} the file in this conversation before editing, or the call will fail.`}
- `old_string` must match the file exactly, including indentation, and be unique — the edit fails otherwise. Strip the Read line prefix (${LINE_NUMBER_PREFIX_FORMAT}) before matching.
- `replace_all: true` replaces every occurrence instead.
