<!--
name: "System Reminder: Workflow isolated worktree"
description: "Tells a workflow subagent it is running in an isolated git worktree separate from the main working directory"
ccVersion: "2.1.242"
variables:
  - "WORKFLOW_SUBAGENT_PROMPT"
  - "PATH_FORMATTER_FN"
  - "WORKTREE_INFO"
  - "MAIN_WORKING_DIRECTORY_FN"
-->
${WORKFLOW_SUBAGENT_PROMPT}

---
You are running in an isolated git worktree at `${PATH_FORMATTER_FN(WORKTREE_INFO.worktreePath)}` (a separate working copy of the repo). Changes you make here do NOT affect the main working directory (`${PATH_FORMATTER_FN(MAIN_WORKING_DIRECTORY_FN())}`) or other agents. Work normally — the worktree will be cleaned up automatically if you made no changes, or preserved for review if you did.
