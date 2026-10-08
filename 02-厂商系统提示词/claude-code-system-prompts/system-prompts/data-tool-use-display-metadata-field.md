<!--
name: "Data: Tool use display metadata field"
description: "Documents the tool_use_meta wire field carrying per-block display metadata for a message's tool_use blocks; it is wrapper-level UI metadata and is not replayed to the model"
ccVersion: "2.1.181"
-->
@internal Display metadata for this message's tool_use blocks, keyed by block id. display_name is the MCP server's `tool.annotations.title` when provided, otherwise a readable transform of the wire name; server_display_name is the MCP server's own display name; icon_url is the MCP server's directory icon URL (claude.ai connectors only). Omitted for blocks whose display label equals the wire name (built-in tools). Wrapper-level sibling — never inside `message.content` — so it is not replayed to the model.
