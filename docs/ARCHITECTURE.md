# Architecture

## Overview

```
 Browser A                                   Browser B
 ┌──────────────────────────┐                ┌──────────────────────────┐
 │ React UI                 │                │ React UI                 │
 │  CollabContext (UI state)│                │  CollabContext           │
 │  TipTap  ── Y.XmlFragment│                │  TipTap  ── Y.XmlFragment│
 │  InkCanvas ─ Y.Array     │                │  InkCanvas ─ Y.Array     │
 │  Presence ── awareness   │                │  Presence ── awareness   │
 │        Y.Doc             │                │        Y.Doc             │
 │   y-websocket provider   │                │   y-websocket provider   │
 └───────────┬──────────────┘                └──────────────┬───────────┘
             │              WebSocket (one room per doc)    │
             └────────────────►  sync server  ◄─────────────┘
```

**Key idea:** the Yjs document is the source of truth for *content* (text, strokes, title). React Context holds only *UI state* (who I am, which room, which tool, connection status, remote users). Never copy document text or strokes into React state.

## Who owns what

| Role | Owns |
|---|---|
| **Lead (L)** | `context/`, `hoc/`, `App.jsx`, `pages/`, `stubs/`, `docs/`, dependencies, merges |
| **Editor (E)** | `components/Editor/`, `components/Toolbar/Toolbar.jsx`, `utils/exportMarkdown.js` |
| **Sync (S)** | `server/`, `hooks/useCollaboration.js`, `hooks/useConnectionStatus.js`, `components/StatusBadge.jsx` |
| **Presence (P)** | `hooks/usePresence.js`, `components/Presence/`, `utils/colors.js`, `utils/names.js`, `utils/throttle.js` |
| **Ink (I)** | `components/Ink/`, `components/Toolbar/ModeToggle.jsx`, `hooks/useCanvasDraw.js`, `utils/exportPng.js` |
| **QA/Docs (Q)** | `hooks/useUndoRedo.js`, `hooks/useOptimisticTitle.js`, `components/TitleBar.jsx`, `styles/`, `__tests__/`, README, `docs/experiments/` |

Shared files (`DocPage.jsx`, `App.jsx`, context) are touched only by the Lead. Everyone else asks for changes through a PR or the group chat.

## Folder structure

```
CoDoc/
├── README.md
├── docs/
│   ├── PROJECT_BRIEF.md  ARCHITECTURE.md  CONTRACTS.md
│   ├── CONVENTIONS.md    AI_CONTEXT.md
│   └── experiments/naive-contenteditable.md
├── server/
│   ├── package.json
│   └── index.js                       S
└── client/
    └── src/
        ├── main.jsx  App.jsx                          L
        ├── pages/        Home.jsx  DocPage.jsx        L
        ├── context/      CollabContext.jsx  collabReducer.js  useCollab.js   L
        ├── hoc/          withCollaboration.jsx        L
        ├── hooks/        useCollaboration.js  useConnectionStatus.js          S
        │                 usePresence.js                                       P
        │                 useCanvasDraw.js                                     I
        │                 useUndoRedo.js  useOptimisticTitle.js                Q
        ├── components/
        │   ├── Editor/   Editor.jsx  Page.jsx                                 E
        │   ├── Toolbar/  Toolbar.jsx (E)   ModeToggle.jsx (I)
        │   ├── Ink/      InkCanvas.jsx  PenCursors.jsx                        I
        │   ├── Presence/ Presence.jsx  PresenceList.jsx  CursorOverlay.jsx    P
        │   ├── TitleBar.jsx                                                   Q
        │   ├── StatusBadge.jsx                                                S
        │   └── ExportMenu.jsx                                                 E/I
        ├── utils/        colors.js throttle.js names.js (P)
        │                 exportMarkdown.js (E)  exportPng.js (I)
        ├── stubs/        fakeUsers.js  fakeStrokes.js                         L  (temporary)
        ├── styles/       theme.css                                            Q
        └── __tests__/                                                         Q
```

## Page layout (how text and ink stack)

```
<Page>                         fixed size, position: relative
  <Editor />                   TipTap, z-index 1
  <InkCanvas />                transparent <canvas>, position: absolute, inset: 0, z-index 2
  <CursorOverlay />            remote carets, z-index 3 (pointer-events: none)
  <PenCursors />               remote pen positions, z-index 3 (pointer-events: none)
</Page>
```

- Mode `type`: the canvas has `pointer-events: none`, so clicks reach the editor.
- Mode `pen` / `highlighter` / `eraser`: the canvas has `pointer-events: auto` and the editor is read-only.

## Data flow

**Typing:** TipTap edit → Yjs `Y.XmlFragment` (applied locally at once) → provider → server → other clients' Yjs → their TipTap updates.

**Drawing:** pointer events → `useCanvasDraw` builds a stroke → pushed to `Y.Array('strokes')` (drawn locally at once) → synced → other clients observe the array and redraw.

**Presence:** local user, mode and pen position are written to Yjs awareness (throttled) → other clients read awareness through `usePresence` → pushed into `CollabContext` → rendered by `PresenceList`, `CursorOverlay`, `PenCursors`.

**Title:** typed into `TitleBar` → `useOptimisticTitle` shows it at once, writes to `Y.Map('meta')`, tracks `pending | confirmed | failed`.

## React patterns, and where they live

| Pattern | Where | Why it earns its place |
|---|---|---|
| Props + callbacks | Toolbar, ModeToggle, TitleBar | UI emits intent upward; owners decide what to do |
| Context + lifting | `CollabContext` | One place for user, room, mode, status, presence |
| Custom hooks | `hooks/` | Collaboration logic reusable and testable on its own |
| Render props | `<Presence>` | Provides remote users to `CursorOverlay`, `PresenceList` and `PenCursors` without each re-subscribing |
| HOC | `withCollaboration(DocPage)` | Creates the Yjs doc and provider for a room, injects `ydoc`, `provider`, `status`, `user` |
| Refs | `InkCanvas`, `Page`, `CursorOverlay` | Canvas drawing and overlay positioning |

## Server

A small Node server that relays Yjs updates and awareness between clients in the same room, using the `y-websocket` family of packages. The Sync owner must read the README of the **installed** version before writing `server/index.js`, since the standalone server setup differs between versions. Do not copy server code from old tutorials or LLM memory.

Rooms: the URL `/doc/:roomId` maps to a Yjs room name equal to `roomId`.

## Known risks

- **Contract drift.** Two LLMs quietly assuming different data shapes. Mitigation: `CONTRACTS.md` plus the PR rule.
- **Ink vs reflow.** Ink does not follow text when text reflows. Mitigation: fixed-size pages and page-relative coordinates.
- **Cursor overlay drift** on scroll or resize. Recompute from editor coordinates.
- **Awareness flood.** Throttle pen and caret updates (about 30-50 ms).
- **TipTap/Yjs version differences.** Check the docs of the installed major version for extension names and options.
