# Install

You can use Vibe Code Common Sense with Codex, Claude Code, or any agent that
can read instructions.

## Codex

Ask Codex:

```text
Install the skill from GitHub:
https://github.com/Njengah/vibe-code-common-sense
```

Then invoke it in a project:

```text
Use $vibe-code-common-sense.

I have a new project idea. Do not build product features yet.
First create the professional project rails and progress tracker.

My idea is:
[paste your idea here]
```

## Claude Code

Add the plugin marketplace:

```text
/plugin marketplace add Njengah/vibe-code-common-sense
```

Install the plugin:

```text
/plugin install vc-sense@vibe-code-common-sense
```

Reload plugins:

```text
/reload-plugins
```

Invoke the skill:

```text
/vc-sense:vc-sense
```

## Any Agent

If your agent does not support skills or plugins, paste this:

```text
Read and follow this project setup system:
https://github.com/Njengah/vibe-code-common-sense/blob/main/SKILL.md

Do not build product features yet. First create the project rails.
```
