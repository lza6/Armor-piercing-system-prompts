<!--
name: "Data: Artifact host MCP server guidance"
description: "Explains how locally configured MCP servers may be declared as Artifact host servers and distinguishes them from Claude.ai connectors and built-in servers"
ccVersion: "2.1.260"
variables:
  - "CONNECTED_CLAUDE_AI_CONNECTOR_COUNT"
-->
 Locally-configured MCP servers connected in this session can also be declared, as host servers: set `server` to `host:<server>` where `<server>` is the segment between `mcp__` and the next `__` in that server's tool names (`mcp__filesystem__read_file` → `host:filesystem`). Only servers from the user's MCP configuration count, with one built-in exception: `host:claude_browser` is the Claude app's own browser — declare it, with the tools the page needs from `read_page`, `get_page_text`, `find`, `preview_start`, `navigate`, `computer` and `form_input`, when the page must read or act on other websites; it answers only when the viewer opens the page in a Cowork session of the desktop app, and the viewer is asked before each website. The app's other built-in servers (`cowork`, `scheduled-tasks`, `session_info`, `workspace` and the like) are never host servers, and a page that declares one is refused at publish.${CONNECTED_CLAUDE_AI_CONNECTOR_COUNT>0?" The `mcp__<id>__` connectors above are claude.ai connectors, never host servers.":""} A host server only answers when the viewer opens the page in a Claude app that has that same local server connected — say so to the user when you publish.
