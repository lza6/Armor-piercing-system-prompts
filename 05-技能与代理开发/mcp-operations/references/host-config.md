# MCP Host Config

## Command
- `node`

## Args
- Absolute path to `mcp/build/index.js`

## Example env
- `MCP_DASHBOARD_BASE_URL=http://127.0.0.1:4820`
- `MCP_DASHBOARD_ALLOW_MUTATIONS=false`
- `MCP_DASHBOARD_ALLOW_DESTRUCTIVE=false`
- `MCP_LOG_LEVEL=info`

## Common mistakes
- Relative path to MCP build entry.
- Dashboard not running while MCP starts.
- Mutating tools used while mutation flag is false.
