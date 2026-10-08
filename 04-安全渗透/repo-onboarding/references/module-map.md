# Module Map

## Backend
- `server/index.js`: app composition, startup behavior, periodic maintenance.
- `server/db.js`: schema and prepared statement ownership.
- `server/routes/*.js`: endpoint contracts by domain.
- `server/websocket.js`: WS lifecycle and broadcast behavior.

## Frontend
- `client/src/pages/`: route-level screens.
- `client/src/components/`: reusable UI building blocks.
- `client/src/lib/api.ts`: client API access patterns.
- `client/src/hooks/useWebSocket.ts`: live update pipeline.

## MCP
- `mcp/src/index.ts`: runtime entrypoint.
- `mcp/src/server.ts`: MCP assembly.
- `mcp/src/tools/domains/`: domain tool registration.
- `mcp/src/clients/dashboard-api-client.ts`: resilient API bridge.
- `mcp/src/policy/tool-guards.ts`: mutation/destructive gates.
