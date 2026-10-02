# Contracts

**Everyone codes against this file.** If you or your LLM think something here is wrong, do **not** change your code to match your idea. Raise it in the group chat. The Lead approves, the change goes into this file through a PR first, and only then does code change.

Contracts version: **0.1.0**

---

## 1. Yjs document shape

One `Y.Doc` per room. Room name = `roomId` from the URL.

| Name | Type | Contents | Owner |
|---|---|---|---|
| `content` | `Y.XmlFragment` (`ydoc.getXmlFragment('content')`) | TipTap document | Editor |
| `strokes` | `Y.Array` (`ydoc.getArray('strokes')`) | `Stroke` objects (see 2) | Ink |
| `meta` | `Y.Map` (`ydoc.getMap('meta')`) | `{ title: string }` | QA |

- TipTap's Collaboration extension must be configured with `field: 'content'`.
- Do not mirror document text or strokes into React state. Read them from Yjs.

---

## 2. Stroke

```js
{
  id: string,            // unique, e.g. crypto.randomUUID()
  userId: string,        // author, matches User.id
  tool: 'pen' | 'highlighter',
  color: string,         // CSS color, e.g. '#e11d48'
  width: number,         // in page pixels
  points: [number, number][],   // [x, y] in PAGE coordinates
  page: number,          // 0-based page index
  createdAt: number      // Date.now()
}
```

Rules:
- **Page coordinates:** x and y are pixels relative to the top-left of the page element, at 100% scale. Never store screen or viewport coordinates.
- **Page size:** `794 x 1123` (A4 at 96 dpi). Constants live in `client/src/utils/constants.js` (Lead creates it).
- **Eraser:** there is no eraser stroke. Erasing **deletes whole strokes by id** from `Y.Array('strokes')`.
- **Immutability:** a stroke is pushed once, when the pointer is released. While drawing, the in-progress stroke is held locally (and optionally as a live preview via awareness, see 4).
- **Highlighter:** same shape, `tool: 'highlighter'`, drawn semi-transparent by the renderer.

---

## 3. User

```js
{
  id: string,      // stable per browser, stored in localStorage
  name: string,    // e.g. 'Guest 42' (utils/names.js)
  color: string    // from the palette in utils/colors.js
}
```

---

## 4. Awareness (presence)

Each client sets its local awareness state to:

```js
{
  user: { id, name, color },   // required, same as User. TipTap's caret extension reads name and color from here
  mode: 'type' | 'pen' | 'highlighter' | 'eraser',
  penPosition: { x: number, y: number, page: number } | null,   // page coords, null when not drawing or hovering
  typing: boolean              // true for ~1 s after a keystroke
}
```

- Text caret and selection are handled by TipTap's collaboration caret extension, which writes its own awareness field. **Do not duplicate caret data** in the shape above.
- Throttle awareness writes to about 30-50 ms.

---

## 5. CollabContext

State:

```js
{
  user: User,
  roomId: string | null,
  status: 'connecting' | 'connected' | 'reconnecting' | 'offline',
  mode: 'type' | 'pen' | 'highlighter' | 'eraser',
  tool: { color: string, width: number },   // current pen settings
  title: { value: string, status: 'confirmed' | 'pending' | 'failed' },
  presence: RemoteUser[]                    // others only, see below
}
```

`RemoteUser` = `{ clientId: number, user: User, mode, penPosition, typing }`.

Actions (all dispatched through `useCollab()`):

| Action | Payload |
|---|---|
| `SET_USER` | `User` |
| `SET_ROOM` | `roomId: string` |
| `SET_STATUS` | `status` |
| `SET_MODE` | `mode` |
| `SET_TOOL` | `{ color?, width? }` |
| `SET_TITLE` | `{ value, status }` |
| `SET_PRESENCE` | `RemoteUser[]` |

Only the Lead edits `collabReducer.js`. To request a new action, raise it in the group chat.

---

## 6. Hook signatures

```js
useCollab() -> { state, dispatch }

useCollaboration(roomId, user) -> { ydoc, provider, status }
  // creates the Y.Doc and provider; destroys both on unmount or roomId change

useConnectionStatus(provider) -> 'connecting' | 'connected' | 'reconnecting' | 'offline'

usePresence(provider, localState) -> { remoteUsers: RemoteUser[], setLocal(partial) }
  // reads awareness, writes local state (throttled)

useCanvasDraw({ canvasRef, ydoc, user, mode, tool, page }) -> void
  // attaches pointer handlers, builds strokes, redraws from Y.Array; cleans up on unmount

useUndoRedo(ydoc) -> { undo, redo, canUndo, canRedo }
  // wraps Y.UndoManager tracking both content and strokes; undoes only local changes

useOptimisticTitle(ydoc, status) -> { title, setTitle, titleStatus }
  // shows the new title at once, writes to meta, marks 'pending' until confirmed
```

---

## 7. Component props

```js
<Toolbar editor onUndo onRedo canUndo canRedo />
<ModeToggle mode tool onModeChange onToolChange />
<TitleBar title titleStatus onTitleChange />
<PresenceList users />                       // users: RemoteUser[]
<StatusBadge status />
<Presence provider>{ (remoteUsers) => ReactNode }</Presence>   // render prop
<InkCanvas ydoc user mode tool page />
<CursorOverlay /> <PenCursors />            // consume <Presence> render prop
withCollaboration(Component)                // injects props: ydoc, provider, status, user
```

Components receive data through props or `useCollab()`. They must **not** create their own Yjs doc or provider. Only `withCollaboration` does that.

---

## 8. Routes

| Path | Page |
|---|---|
| `/` | `Home`: create a room or enter a code |
| `/doc/:roomId` | `DocPage` (wrapped in `withCollaboration`) |

`roomId`: short random string (e.g. 8 lowercase alphanumeric characters).

---

## 9. Undo/redo scope

`Y.UndoManager` tracks `content` and `strokes`, with `trackedOrigins` limited to local changes so each user only undoes their own work. TipTap's built-in history must be disabled when using the Collaboration extension (the option name differs by TipTap version, so check the docs of the installed one).

---

## Change log

| Version | Change |
|---|---|
| 0.1.0 | Initial contracts |
