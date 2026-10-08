<!--
name: "System Reminder: MCP resource no content"
description: "Shown when MCP resource has no content"
ccVersion: "2.1.234"
variables:
  - "ESCAPE_XML_ATTRIBUTE_FN"
  - "ATTACHMENT_OBJECT"
  - "MCP_RESOURCE_STATUS_MESSAGE"
-->
<mcp-resource server="${ESCAPE_XML_ATTRIBUTE_FN(ATTACHMENT_OBJECT.server)}" uri="${ESCAPE_XML_ATTRIBUTE_FN(ATTACHMENT_OBJECT.uri)}">(${MCP_RESOURCE_STATUS_MESSAGE})</mcp-resource>
