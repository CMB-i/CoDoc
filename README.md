# CoDoc

A real-time collaborative document editor with a pen/ink layer, built with React. Multiple people edit the same page at once, see each other's cursors, and can switch to a pen to draw or annotate right on the page, like a doc and a whiteboard in one.

> **Status:** early development. See [`docs/`](docs/) for the plan, contracts and conventions.

## Features

**Core**
- Real-time multi-user text editing (CRDT-based, no lost edits)
- Live cursors and selections with name tags
- Presence list (who is online)
- Pen, highlighter and eraser drawn over the page, synced live
- Connection status (connected / reconnecting / offline) with offline edits merging on reconnect
- Rooms: join a document by URL

**Extensions**
- Per-user undo/redo (only undoes your own changes)
- Optimistic document title with confirm and rollback
- Export to Markdown (text) and PNG (ink)

**Out of scope (future work):** AI writing tools, auth and permissions, version history, comments, database persistence.

## Tech stack

| Area | Choice |
|---|---|
| UI | React + Vite |
| Shared UI state | Context API with `useReducer` |
| Editor | TipTap (ProseMirror) |
| Sync | Yjs (CRDT) + `y-websocket` |
| Presence | Yjs awareness |
| Ink | Canvas API accessed through React refs |
| Routing | react-router-dom |
| Lint / format | Oxlint + Prettier |

## How the brief is covered

| Requirement | Where |
|---|---|
| Props and callbacks | `Toolbar`, `ModeToggle`, `TitleBar`, `PresenceList` |
| Context API, state lifting | `context/CollabContext.jsx` |
| WebSocket | `hooks/useCollaboration.js` + y-websocket provider |
| Real-time cursor tracking | `hooks/usePresence.js`, `CursorOverlay`, `PenCursors` |
| Collaborative editing | TipTap + Yjs |
| Canvas API + refs | `Ink/InkCanvas.jsx`, `hooks/useCanvasDraw.js` |
| Optimistic updates | Yjs local-first, plus `hooks/useOptimisticTitle.js` |
| Custom hooks | `hooks/` |
| Render props | `components/Presence/Presence.jsx` |
| Higher-order component | `hoc/withCollaboration.jsx` |

## Getting started

**Requirements:** Node 22 or 24 LTS, and npm.

```bash
git clone https://github.com/CMB-i/CoDoc.git
cd CoDoc

# client
cd client
npm install

# server (in a second terminal)
cd ../server
npm install
```

Run (two terminals):

```bash
# terminal 1: sync server
cd server && npm start

# terminal 2: client
cd client && npm run dev
```

Open `http://localhost:5173`, create a room, then open the same room URL in a second window to test sync.

> The `npm start` script and server entry point are added by the Sync owner. Until then, only the client runs.

## Project structure

```
CoDoc/
├── client/        React app (Vite)
├── server/        sync server (y-websocket)
└── docs/          brief, architecture, contracts, conventions, AI context
```

Full layout and file ownership: [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md).

## Documentation

- [`docs/PROJECT_BRIEF.md`](docs/PROJECT_BRIEF.md): the brief and our decisions
- [`docs/ARCHITECTURE.md`](docs/ARCHITECTURE.md): structure, ownership, data flow
- [`docs/CONTRACTS.md`](docs/CONTRACTS.md): shared data shapes and interfaces (**read before coding**)
- [`docs/CONVENTIONS.md`](docs/CONVENTIONS.md): code style, git workflow
- [`docs/AI_CONTEXT.md`](docs/AI_CONTEXT.md): paste into every LLM session
- [`docs/experiments/`](docs/experiments/): why we chose a CRDT over plain `contentEditable`

## Team

| Role | Owner | Area |
|---|---|---|
| Lead / Integrator | _name_ | context, HOC, app shell, reviews |
| Editor | _name_ | TipTap editor, toolbar, Markdown export |
| Sync / Backend | _name_ | server, Yjs provider, connection status |
| Presence | _name_ | awareness, cursors, presence list |
| Ink | _name_ | canvas overlay, pen tools, PNG export |
| UX / QA / Docs | _name_ | undo/redo, optimistic title, tests, styling, README |

## Known limitations

- Ink is anchored to fixed-size pages, not to text. Text that reflows does not move the ink with it.
- Eraser removes whole strokes, not parts of a stroke.
- No auth: anyone with a room URL can edit.
