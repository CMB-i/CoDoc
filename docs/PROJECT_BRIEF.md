# Project Brief

## Original brief (Project 36: Real-time Collaboration Tool)

Build a collaborative whiteboard or document editor interface using React where multiple users can work simultaneously. Implement component communication through props and callbacks, shared state management using Context API or Redux, real-time cursor position tracking, and collaborative editing features. The interface will demonstrate advanced React patterns including render props, higher-order components, and proper state lifting for managing shared application state.

Required tools: React (components, Context API, hooks), Canvas API (whiteboard drawing), WebSocket (frontend setup), Context API or Redux, React refs (canvas manipulation), custom hooks (reusable collaboration logic), optimistic updates.

## Our decision

We are building a **collaborative document editor with a pen/ink layer**. The document covers real-time collaborative editing and cursors. The ink layer covers the Canvas API and refs requirements from the whiteboard side of the brief.

The professor confirmed that **libraries are allowed**.

## Locked stack

- React + Vite
- Context API with `useReducer` (no Redux)
- TipTap + Yjs for the document, `y-websocket` for sync, Yjs awareness for presence
- Canvas API through refs for ink
- Strokes stored in a shared Yjs array, so there is one sync system

## In scope

1. Collaborative rich-text editing
2. Live cursors, selections and a presence list
3. Pen, highlighter and eraser overlay, synced live
4. Connection status, with offline edits merging on reconnect
5. Rooms by URL
6. Per-user undo/redo
7. Optimistic title with confirm and rollback
8. Export: Markdown and PNG
9. The React patterns: props/callbacks, Context + lifted state, custom hooks, render props, one HOC

## Out of scope (do not add)

These came from an earlier idea and have been **deliberately dropped**. If an LLM suggests any of them, ignore it.

- AI features (summarize, grammar, tone, ghost collaborator, translation)
- Auth, sign-in, permissions
- MongoDB or any database
- Express backend, custom OT implementation
- Tailwind (we use plain CSS)
- Version history, comments

They can be listed under "Future work" in the README.

## Priorities if time runs short

Build in this order so there is a working demo at every stage:

1. Collaborative text + presence
2. Ink overlay + stroke sync
3. Undo/redo, export, polish

Drop from the bottom of the list first.

## Milestones

| Checkpoint | Demo |
|---|---|
| Day 1 | Contracts committed, stubs pushed, roles assigned |
| Day 5 | Two windows editing the same doc |
| Day 9 | Live cursors and presence |
| Day 12 | Ink synced between windows |
| Day 14 | Full demo, README, rehearsal |
