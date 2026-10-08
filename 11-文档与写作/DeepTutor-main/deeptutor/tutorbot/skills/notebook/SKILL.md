---
name: notebook
description: "Manage DeepTutor notebooks — list, create, view, import and remove records."
metadata: {"nanobot":{"emoji":"📓","requires":{"bins":["deeptutor"]}}}
always: false
---

# Notebook Management

Use the `exec` tool to manage DeepTutor notebooks via CLI.

## When to Use

- User asks about their **notebooks** or **notes**
- User wants to **create**, **view**, or **organize** notebook records
- User needs to **import Markdown** into a notebook

## Commands

### List all notebooks

```bash
deeptutor notebook list
```

### Create a notebook

```bash
deeptutor notebook create <name> --description "Description text"
```

### Show a notebook and its records

```bash
deeptutor notebook show <notebook_id>
```

### Import a Markdown file as a record

```bash
deeptutor notebook add-md <notebook_id> /path/to/file.md
```

### Replace an existing record

```bash
deeptutor notebook replace-md <notebook_id> <record_id> /path/to/file.md
```

### Remove a record

```bash
deeptutor notebook remove-record <notebook_id> <record_id>
```

## Tips

- Use `notebook list` first to see available notebooks and their IDs.
- Use `notebook show <id>` to see individual records within a notebook.
- Notebooks are distinct from knowledge bases: notebooks store structured notes, KBs store indexed documents for RAG retrieval.
