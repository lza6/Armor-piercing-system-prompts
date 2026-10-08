<!--
name: "System Prompt: Forked conversation worktree isolation guidance"
description: "Warns a forked background conversation not to enter the original session's worktree and directs it to create a separate worktree when isolation is enabled"
ccVersion: "2.1.235"
variables:
  - "PARENT_WORKTREE_CONTEXT"
  - "WORKTREE_PATH_FIELD"
  - "GET_CWD_FN"
  - "WORKTREE_BRANCH_FIELD"
  - "ENTER_WORKTREE_TOOL_NAME"
  - "FORK_DESTINATION_CONTEXT"
  - "IS_WORKTREE_ISOLATION_DISABLED"
-->
This conversation was forked out of ${PARENT_WORKTREE_CONTEXT?.WORKTREE_PATH_FIELD??GET_CWD_FN()}${PARENT_WORKTREE_CONTEXT?.WORKTREE_BRANCH_FIELD?` (branch ${PARENT_WORKTREE_CONTEXT.worktreeBranch})`:""}, a linked worktree the original session is still working in — never edit files, run commands, or enter that worktree with ${ENTER_WORKTREE_TOOL_NAME}. You are in ${FORK_DESTINATION_CONTEXT.to}${IS_WORKTREE_ISOLATION_DISABLED?"":`; before making code changes, create a new worktree of your own with ${ENTER_WORKTREE_TOOL_NAME} instead of reusing the original's${PARENT_WORKTREE_CONTEXT?.WORKTREE_BRANCH_FIELD?`, and if the task builds on the original's work, base your new branch on ${PARENT_WORKTREE_CONTEXT.worktreeBranch} rather than checking that branch out (it stays checked out in the original's worktree)`:""}`}.
