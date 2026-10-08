# PRD: Purple Lab Full IDE Experience

## Overview

Transform Purple Lab from a basic file viewer into a production-grade fullstack IDE comparable to Replit, Lovable, and Bolt. Users should be able to build a project via PRD, then immediately browse, edit, preview, and iterate on the output -- all within the browser.

## Target Users

- Developers using Loki Mode to generate projects from PRDs
- Non-technical founders using Purple Lab as their primary development environment
- Teams evaluating Loki Mode for autonomous development workflows

## Current State (v6.39.0)

- React Router with `/project/:sessionId` URLs
- File tree sidebar with collapsible directories
- Code viewer with line numbers (read-only)
- Live preview via iframe serving real files with correct MIME types
- Session state persists across refresh
- WebSocket real-time log streaming during builds

## Requirements

### P0 -- Must Have

#### Monaco Editor Integration
- Replace the read-only code viewer with Monaco Editor (VS Code's editor)
- Syntax highlighting for all major languages (JS, TS, Python, HTML, CSS, JSON, MD, Go, Rust)
- Read-only mode by default, editable when user clicks "Edit" button
- File save via `PUT /api/sessions/:id/file?path=` endpoint
- Keyboard shortcut: Cmd/Ctrl+S to save
- Show unsaved changes indicator (dot on tab)

#### Resizable Split Panes
- Three-pane layout: file tree | editor | preview
- Drag handles between panes to resize
- Pane sizes persist to localStorage
- Collapse/expand each pane via toggle buttons
- Minimum widths: file tree 180px, editor 300px, preview 300px

#### Integrated Terminal
- xterm.js terminal emulator in a bottom panel
- WebSocket-connected to backend for real-time output
- Resizable height (drag handle)
- Support for ANSI color codes
- Show build output, loki logs, and allow running commands in project directory
- Toggle with Cmd/Ctrl+` keyboard shortcut

#### File Operations
- Create new file (right-click context menu or button)
- Create new directory
- Rename file/directory (inline edit)
- Delete file/directory (with confirmation)
- All operations via REST API with path traversal protection

### P1 -- Should Have

#### Tab System
- Open multiple files in tabs
- Close tabs (with unsaved changes warning)
- Tab reordering via drag
- "Close All" and "Close Others" context menu
- Active tab highlighted

#### Search
- Cmd/Ctrl+P: fuzzy file search (like VS Code quick open)
- Cmd/Ctrl+Shift+F: search across all files in project
- Search results panel with file + line previews
- Click result to open file at line

#### Preview Enhancements
- URL bar showing current preview path
- Back/forward navigation buttons
- Refresh button
- Open in new tab button
- Console panel showing preview JS errors (via postMessage from sandboxed iframe)
- Auto-refresh on file save

### P2 -- Nice to Have

#### Git Integration
- Show git status in file tree (modified, added, untracked)
- Diff view for modified files
- Commit from UI
- Branch indicator in header

#### AI Chat Panel
- Side panel for chat with Claude about the project
- Context-aware: knows which file is open, what project is about
- "Fix this" button on error highlights
- "Explain" button on selected code

#### Deployment
- "Deploy" button that runs loki deploy
- Deploy status indicator
- Preview deployed URL

## Technical Architecture

### Frontend
- React 19 + TypeScript + Tailwind CSS (existing stack)
- Monaco Editor: `@monaco-editor/react` package
- xterm.js: `@xterm/xterm` + `@xterm/addon-fit` + `@xterm/addon-web-links`
- Split panes: custom implementation with CSS resize or `react-resizable-panels`
- File operations: REST API calls to new CRUD endpoints

### Backend (web-app/server.py)
- `PUT /api/sessions/:id/file?path=` -- save file content
- `POST /api/sessions/:id/file` -- create new file
- `DELETE /api/sessions/:id/file?path=` -- delete file
- `POST /api/sessions/:id/directory` -- create directory
- `POST /api/sessions/:id/rename` -- rename file/directory
- `GET /api/sessions/:id/search?q=&glob=` -- search files
- WebSocket terminal: `/ws/terminal/:id` -- PTY-backed terminal session

### Security
- All file operations scoped to project directory (path traversal protection)
- File size limits on saves (1MB)
- Rate limiting on file operations
- No shell injection in terminal (PTY spawns bash in project dir only)
- Preview iframe sandboxed

## Success Criteria

- User can build a project via PRD, then immediately edit files and see preview update
- Preview loads all relative assets (CSS, JS, images) correctly
- Terminal shows real-time build output and allows running commands
- All operations work without page refresh
- Performance: file tree renders 1000+ files without freezing
- Mobile: graceful degradation (single-pane view)

## Non-Goals

- Multi-user collaboration (Google Docs style)
- Version control UI beyond basic git status
- Package manager integration (npm/pip install from UI)
- Database management UI
- Cloud deployment orchestration

## Timeline

- P0: 2-3 days focused development
- P1: 2 days
- P2: Future sprints

## References

- Replit: https://replit.com
- Lovable: https://lovable.dev
- Bolt: https://bolt.new
- Monaco Editor: https://microsoft.github.io/monaco-editor/
- xterm.js: https://xtermjs.org/
