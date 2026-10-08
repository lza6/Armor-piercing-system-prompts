# Feature Checklist

- Scope
  - Problem and success criteria are explicit.
  - Impacted layers identified (server/client/mcp/docs/scripts).

- Implementation
  - Input validation and error handling are explicit.
  - Existing behavior preserved where not in scope.
  - Safety controls preserved.

- Verification
  - Backend: `npm run test:server` when backend changed.
  - Frontend: `npm run test:client` when UI changed.
  - MCP: `npm run mcp:typecheck` + `npm run mcp:build` when MCP changed.

- Documentation
  - `README.md`, `ARCHITECTURE.md`, `SETUP.md`, `INSTALL.md`, `mcp/README.md` updated as needed.
  - Commands in docs match `package.json`.

- Delivery
  - Known risks and unrun checks are clearly stated.
