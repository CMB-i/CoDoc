# Conventions

## Environment

- **Node 22 or 24 LTS** for everyone. Do not use odd-numbered (Current) releases.
- Package manager: **npm**. Commit `package-lock.json`. Never commit `node_modules/`.
- Only the **Lead** adds, removes or upgrades dependencies. Versions are pinned exactly. If you need a new package, ask in the group chat first.

## Code style

- JavaScript (no TypeScript), ES modules, **functional components and hooks only**.
- One component per file. File name = component name (`PresenceList.jsx`). Hooks start with `use` and live in `hooks/`.
- Components: `PascalCase`. Functions and variables: `camelCase`. Constants: `UPPER_SNAKE_CASE`. Action types: `UPPER_SNAKE_CASE` strings.
- Plain CSS in `styles/` (no Tailwind, no CSS-in-JS libraries).
- Lint: **Oxlint** (`npm run lint` in `client/`). Format: **Prettier**. Fix both before opening a PR.
- Keep comments for the "why", not the "what".
- No `console.log` left in merged code.

## Do and don't

- **Do** read content from Yjs, UI state from Context.
- **Do** clean up in `useEffect` (observers, listeners, providers).
- **Don't** create a second `Y.Doc` or provider anywhere except `withCollaboration`.
- **Don't** change files outside your area without asking the owner.
- **Don't** change anything in `CONTRACTS.md` or the reducer without Lead approval.

## Git workflow

- `main` is always runnable. No direct pushes to `main`.
- Branch names: `feature/<area>-<thing>`, `fix/<area>-<thing>`, `docs/<thing>`. Examples: `feature/ink-eraser`, `fix/presence-name-tag`.
- Small PRs, at least every 1-2 days. Rebase or merge `main` into your branch before opening a PR.
- Every PR needs **one review** (the Lead, or the owner of the neighbouring module). Reviewer checks: runs locally, follows contracts, no stray files.
- PR description: what changed, how to test it, any contract impact.
- Delete the branch after merge.

### Commit messages

Format: `type(area): short summary`

Types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `style`.

Examples:
- `feat(ink): add eraser that removes whole strokes`
- `fix(sync): destroy provider on room change`
- `docs(contracts): add typing flag to awareness`

## Working with LLMs

1. Start every new LLM chat by pasting `docs/AI_CONTEXT.md` and `docs/CONTRACTS.md`.
2. LLMs suggest code. **You** are responsible for it. You must be able to explain every line you commit (the professor may ask).
3. LLM memory of library APIs is often out of date (TipTap, Yjs, y-websocket, Vite, react-router change often). Check the official docs of the installed version when the LLM's answer touches an API.
4. If an LLM wants to change a contract, add a package, or move files, stop and ask the group.

## Definition of done (for a PR)

- Runs with `npm run dev` and no console errors
- Works in two browser windows when it touches sync, presence or ink
- Lint and format pass
- Matches `CONTRACTS.md`
- Reviewed and approved
