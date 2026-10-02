# Experiment: why not plain `contentEditable` over WebSocket?

**Owner:** QA/Docs
**Status:** TODO. Run this experiment, then fill in the results. Do not write results you have not actually observed.

## Goal

Show, with evidence, why we use a CRDT (Yjs) and an editor framework (TipTap) instead of building our own sync on top of a raw `contentEditable` element.

## Setup (about 2-3 hours)

1. Create a throwaway folder outside the repo, or a branch you will not merge, named `experiment/naive-editor`.
2. A page with one `<div contenteditable>`.
3. A tiny Node WebSocket server that rebroadcasts messages to all other clients.
4. On every `input` event, send the full `innerHTML` (or text). On receipt, set it back into the div.
5. Open the page in two browser windows.

## Tests to run

| # | Test | What to look for |
|---|---|---|
| 1 | Both windows type in the same paragraph at the same time | Lost or overwritten characters |
| 2 | One window types while the other is mid-sentence | Caret jumps to the start or end |
| 3 | Type in one window, then edit the same spot in the other | Cursor reset, duplicate text |
| 4 | Disconnect one window (devtools offline), edit both, reconnect | Whose edits survive? |
| 5 | Try bold and lists with `document.execCommand` in two browsers (Chrome, Safari or Firefox) | Inconsistent HTML, deprecated API |

## Results

_Fill in after running. Add screenshots or a short screen recording in this folder (`docs/experiments/`)._

| # | What happened | Evidence (file name) |
|---|---|---|
| 1 | | |
| 2 | | |
| 3 | | |
| 4 | | |
| 5 | | |

## Conclusion

_Write 3-4 sentences based on what you observed. Reference the failures above and explain how Yjs (merges edits per character, no overwrites, relative cursor positions, offline merge) and TipTap (consistent rich-text model) address them._
