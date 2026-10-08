# Claude Code Install Notes

This repo is packaged as a Claude Code plugin through
`.claude-plugin/plugin.json`.

## Marketplace Install

Add the marketplace:

```text
/plugin marketplace add Njengah/vibe-code-common-sense
```

Install the plugin:

```text
/plugin install vc-sense@vibe-code-common-sense
```

Reload plugins so the new skill is available in the current session:

```text
/reload-plugins
```

Invoke it:

```text
/vc-sense:vc-sense
```

## Local Plugin Directory

For local testing:

```bash
git clone https://github.com/Njengah/vibe-code-common-sense.git
claude --plugin-dir ./vibe-code-common-sense
```

## Explicit-Only Behavior

The Claude plugin wrapper uses `disable-model-invocation: true`, so Claude Code
users invoke the skill explicitly.

The shared root `SKILL.md` does not include that Claude-only frontmatter because
Codex validation rejects it. Keep the Claude-specific key in
`skills/vc-sense/SKILL.md`.

If you create a Claude-only fork, you may add this frontmatter key:

```yaml
disable-model-invocation: true
```

Do not add that key to this shared package unless Codex compatibility is no
longer required.
