<!--
name: "Tool Description: Artifact database version pinning guidance"
description: "Instructs Claude to pin writes to the last-read document version with if_version and to re-read and retry after conflicts instead of overwriting concurrent changes"
ccVersion: "2.1.267"
-->
 Pin every write to a document you have read: pass the `version` you last saw — every document you read shows it, and so does the result of every set, update and str_replace — as `if_version` on "set", "update", "str_replace" and "delete", and in each "batch" entry. There is then no need to re-read first to check for changes: if someone has edited the document since, a pinned write fails, writes nothing and names the current version (for a batch, the entry), and you re-read and redo that write rather than overwrite their change. `if_version` is optional; omit it only for a document you have not read.
