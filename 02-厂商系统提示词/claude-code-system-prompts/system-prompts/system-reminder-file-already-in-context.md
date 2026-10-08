<!--
name: "System Reminder: File already in context"
description: "Tells Claude that a file is already loaded in context and unchanged on disk, so it should use the existing content instead of re-reading"
ccVersion: "2.1.199"
variables:
  - "FILE_ALREADY_IN_CONTEXT_REMINDER_PREFIX"
  - "FILE_PATH"
-->
${FILE_ALREADY_IN_CONTEXT_REMINDER_PREFIX} (see "Contents of ${FILE_PATH}" above) and has not changed on disk. Use that content instead of re-reading.</system-reminder>
