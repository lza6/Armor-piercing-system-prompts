<!--
name: "Tool Description: Artifact files guidance"
description: "Explains how to list and read individual files from a multi-file Artifact and when to use the default scratchpad destination or an explicit output directory"
ccVersion: "2.1.265"
-->


**Artifact files**: a multi-file artifact's individual files can be read without fetching the whole page — `action: "list_files"` (with `url`) prints each file's path, type, and size; `action: "read_file"` (with `url` and `path`) saves that file under its published path in a folder of your scratchpad directory and tells you where — a small text file's contents also come back in the result; a larger or binary one, Read from there; pass `out_dir` only when the user wants the file somewhere else, since saving outside the scratchpad asks them each time. Works for artifacts you can open in your organization.
