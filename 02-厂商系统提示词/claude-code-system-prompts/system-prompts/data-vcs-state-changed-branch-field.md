<!--
name: "Data: VCS state changed branch field"
description: "Describes the optional best-effort branch hint on vcs_state_changed events for commits and pushes, including multi-branch and uncertain-attribution behavior"
ccVersion: "2.1.238"
-->
The branch a commit landed on (from the commit summary line git prints per commit) or a push updated (from the ref-update line git prints per pushed ref, e.g. `HEAD -> other`). Commit and push events carry it; a command that pushed several branches emits one push event per branch. A best-effort hint: absent whenever attribution is uncertain (a push whose output was redirected still emits, with no branch; a push that also updated a ref whose name could not be carried emits a nameless event beside the named ones), and never a required key.
