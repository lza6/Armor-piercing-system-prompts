---
name: cowart-open-canvas
description: Open, reopen, or explicitly refresh the native Cowart canvas when the user asks to see it, or when a bare @Cowart invocation has no other requested workflow.
---

# Cowart Open Canvas

The actual canvas-opening capability is the `cowart_mcp` MCP server and its `render_cowart_canvas_widget` tool. Use this skill when the user asks to open, reopen, or explicitly refresh the canvas. A bare `@Cowart` invocation with no other actionable request may also open the canvas. Do not use this skill as a prerequisite for image generation, annotation editing, HTML, Slides, or follow-up requests sent from an already-open Cowart widget.

## Workflow

1. Ensure the `cowart_mcp` MCP server is loaded or discoverable and that its `render_cowart_canvas_widget` tool is available. The host may expose the complete tool name as `mcp__cowart_mcp__render_cowart_canvas_widget`. Call the tool once for the user's open, reopen, or explicit refresh request. Pass the user's active Codex workspace as `projectDir`; do not pass the Cowart plugin repository directory.

```json
{
  "projectDir": "/absolute/path/to/user/codex-project"
}
```

The tool returns `openai/outputTemplate: ui://widget/cowart/canvas.html`, which tells Codex to render the widget directly. Do not start `scripts/start-canvas.sh` or open a localhost URL for normal use.

2. Confirm the widget opens for the user. The canvas data is stored in the active project:

```text
canvas/pages/<page-id>/cowart-canvas.json
canvas/pages/<page-id>/assets/
```

3. If the MCP tool is not visible in the current turn, use tool discovery for Cowart widget/render capabilities and retry within the same task after an explicit Cowart canvas invocation. Do not claim that a new task is required merely because one turn did not expose the tool. Only suggest a new Codex task when the plugin was just installed or upgraded and the MCP schema is still unavailable after discovery and an explicit retry.

## Constraints

Do not call `render_cowart_canvas_widget` again merely because another Cowart skill applies or an existing widget sends a follow-up request. The existing widget synchronizes MCP-backed canvas changes itself. Do not launch the old local web service, inspect canvas files, run builds, check storage layout, take screenshots, or perform other validation steps unless opening the widget fails or the user explicitly asks for those checks. The `scripts/start-canvas.sh` path is now only a local-development fallback.
