<!--
name: "Data: DirectoryAdded hook description"
description: "Describes when the DirectoryAdded hook fires, its input fields, and how failures and output are handled for add-dir and register_repo_root sources"
ccVersion: "2.1.219"
-->
Fires after /add-dir or the register_repo_root SDK control request registers a new working directory, after the sandbox configuration has been refreshed — so sandboxed tools and permission state already see the new directory (hook commands themselves run unsandboxed).
Input to command is JSON with directory (absolute path) and source ("slash_command" or "register_repo_root").
Exit code 0 - command completes successfully
Other exit codes - stderr is debug-logged on both paths; for /add-dir, a failure count is summarized to Claude and hook systemMessage output reaches Claude as bounded context; for register_repo_root, everything is debug-logged only
