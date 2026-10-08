<!--
name: "Tool Description: Write"
description: "Tool for writing files to the local filesystem"
ccVersion: "2.1.236"
variables:
  - "SHOULD_USE_OUTSIDE_WORKING_DIRECTORY_READ_NOTE"
  - "OUTSIDE_WORKING_DIRECTORY_READ_BEFORE_WRITE_NOTE_FN"
  - "READ_BEFORE_WRITE_NOTE_FN"
-->
Writes a file to the local filesystem.

Usage:
- This tool will overwrite the existing file if there is one at the provided path.${SHOULD_USE_OUTSIDE_WORKING_DIRECTORY_READ_NOTE?OUTSIDE_WORKING_DIRECTORY_READ_BEFORE_WRITE_NOTE_FN():READ_BEFORE_WRITE_NOTE_FN()}
- Prefer the Edit tool for modifying existing files — it only sends the diff. Only use this tool to create new files or for complete rewrites.
- NEVER create documentation files (*.md) or README files unless explicitly requested by the User.
- Only use emojis if the user explicitly requests it. Avoid writing emojis to files unless asked.
