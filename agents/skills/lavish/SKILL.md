---
name: lavish
description: >
  Open an HTML artifact in Lavish Editor so a human can click elements or select text,
  annotate, and send that feedback back to the agent. Use when a system map, plan,
  comparison, architecture, or report is easier to point at than to describe in prose;
  when the user says /lavish; or after writing an HTML canvas the operator should mark up.
  Not the old lavish-design brand kit.
---

# lavish — HTML as the common canvas

Upstream CLI: [kunchenguid/lavish-axi](https://github.com/kunchenguid/lavish-axi).
Occasion: 2026-09-19 operator instruction — the common canvas for this workspace is a
file-anchored system map rendered as HTML, then marked up in the browser, not a longer
markdown description of a diagram.

## When to use

- The user needs to point at a box, arrow, or sentence on a picture of the system
- `/flowchart` has an accurate map and a human one-pager should be reviewable
- A plan, comparison, or architecture is about to be discussed

## When NOT to use

- Do not install session-start hooks (`lavish-axi setup hooks`) — every session pays tokens
- Do not publish with `lavish-axi share` unless the operator asks (shares are public by default)
- Do not pull in FirstMate, Treehouse, or No Mistakes from the same author
- Do not restyle nyrobrain product UI with a brand kit; this skill is the **editor**, not ink/brass tokens

## Current guidance lives in the CLI

Do not copy flags or playbook HTML into this file — they go stale.

```bash
npx -y lavish-axi --help
npx -y lavish-axi playbook diagram
npx -y lavish-axi playbook explanation
npx -y lavish-axi design
npx -y lavish-axi <html-file>          # open / resume review
npx -y lavish-axi poll <html-file>     # wait for annotated feedback; leave running
npx -y lavish-axi end <html-file>
```

Default artifact directory if the caller does not name a path: `.lavish/`.
Durable canvases for this repo live at `docs/system-map/canvas-<subject>.html`
(e.g. `canvas-alpha-research.html`) so they travel with the maps, not only in a
gitignored folder. **One subject, one file — never share a fixed canvas path across
sessions.** Occasion: 2026-09-20, two parallel sessions wrote the same
`canvas.html` within an hour and one live review silently replaced the other; the
provenance header told us whose it was, but only after the damage.

## How a turn runs here

1. Prefer an existing accurate map (`docs/system-map/`) over inventing a new picture.
2. Write or refresh the HTML. State which design source you used (project tokens, or
   the CLI default CDN when the subject has none).
3. `npx -y lavish-axi <html-file>` so the operator can click and comment.
4. `npx -y lavish-axi poll <html-file>` in the foreground (or a harness-tracked
   background job). Apply the returned comments to the named elements; do not guess.

## Provenance header (required)

Occasion: 2026-09-20 operator instruction — many sessions run in parallel, and an
editor page with no origin label cannot be traced back to the window/tab/session that
made it. Every artifact this skill produces carries a **provenance line as the first
element of `<body>`** (above the header), and every `lavish-axi <file>` open names the
same origin in the chat reply.

Collect, in this order:

1. **herdr identity** — when `HERDR_TAB_ID` is set, resolve human labels:
   `herdr workspace get "$HERDR_WORKSPACE_ID"` and `herdr tab get "$HERDR_TAB_ID"`.
   Record label + raw id, e.g. `15Sep2026 Sprint / alpha-idea-pass (w32:t2)`.
   When unset, record `TERM_PROGRAM` + tty instead.
2. **Host** (`hostname -s`), **repo** (basename of the working directory), **timestamp**
   (`date '+%Y-%m-%d %H:%M %Z'`).
3. **Session self-description** — one short clause you write yourself: the CLI and what
   this session is doing, e.g. `kimi session — alpha-input study`. This is the field
   that distinguishes two tabs running the same CLI; never skip it.

Render as one small muted line (`text-xs`, ~70% opacity) — findable when you look,
silent when you don't:

```
from: 15Sep2026 Sprint / alpha-idea-pass (w32:t2) · MBP · nyrobrain · 2026-09-20 05:41 +08 · kimi session — alpha-input study
```

When a session edits an artifact it did not create, append `· edited by <same format>`
rather than replacing the original line.

## Divergence from upstream

- Distributed via `.agents/skills/` + git, not only `npx skills add`.
- Session hooks left uninstalled (house AXI decision).
- Brand skill `lavish-design` was removed 2026-09-19: it was the chrome tokens, not the editor.
