# Claude Code Plugin

The Claude Code plugin uses:

```text
.claude-plugin/plugin.json
.claude-plugin/marketplace.json
skills/vc-sense/SKILL.md
```

The short command is:

```text
/vc-sense:vc-sense
```

Validate locally:

```bash
claude plugin validate . --strict
claude --plugin-dir . plugin details vc-sense
```

The Claude wrapper skill uses `disable-model-invocation: true`, so users invoke
it explicitly.
