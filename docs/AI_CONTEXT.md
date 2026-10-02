# AI Context

**Paste this entire file, plus `docs/CONTRACTS.md`, at the start of every new LLM chat.** Then fill in the last section for yourself.

---

## Project

CoDoc is a student project (a team of 6) for "Project 36: Real-time Collaboration Tool". It is a **real-time collaborative document editor with a pen/ink layer**, built with React. Multiple users edit one page together, see each other's cursors, and can draw on the page with a pen.

## Locked stack (do not suggest alternatives)

- React + Vite (JavaScript, functional components and hooks only)
- Context API with `useReducer` for UI state (**not Redux**)
- TipTap (ProseMirror) editor
- Yjs (CRDT) with the `y-websocket` family of packages for sync
- Yjs awareness for presence
- Canvas API through React refs for ink
- react-router-dom for routing
- Plain CSS, Oxlint, Prettier
- Node 22 or 24 LTS

## Out of scope (never add or suggest)

AI features, authentication, permissions, databases (MongoDB etc.), Express, custom OT, Tailwind, Redux, TypeScript, version history, comments.

## Architecture in five lines

1. The **Yjs doc is the source of truth for content**: `content` (text), `strokes` (ink), `meta` (title).
2. **React Context holds UI state only**: user, room, mode, tool, status, presence.
3. Never copy document text or strokes into React state.
4. Only `withCollaboration` creates the `Y.Doc` and provider. Everything else receives them through props.
5. The page is fixed-size. A transparent canvas sits over the editor. Strokes use **page coordinates**.

## Rules for you (the LLM)

- Follow `CONTRACTS.md` exactly: data shapes, hook signatures, prop names, action names.
- **Do not change a contract.** If you think one is wrong, say so and tell me to raise it with the group.
- **Do not add dependencies.** If you think one is needed, tell me and explain why.
- **Stay inside my area** (below). Do not edit files owned by others.
- Library APIs change between versions (TipTap, Yjs, y-websocket, Vite, react-router). If you are not sure an API exists in the installed version, say so and tell me to check the docs rather than guessing.
- Keep code simple and readable. I have to explain it to a professor.
- Add cleanup for every effect (listeners, observers, providers).
- No AI features, even as an "enhancement".

## Pinned versions

See `client/package.json` and `server/package.json`. Those are the truth. Do not assume versions from memory.

## Repo layout

See `docs/ARCHITECTURE.md` for the folder tree and who owns what.

## My assignment (fill in before pasting)

- **My name:**
- **My role:** (Lead / Editor / Sync / Presence / Ink / QA-Docs)
- **My files:**
- **Current task:**
- **Branch:**
