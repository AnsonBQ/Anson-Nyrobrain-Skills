# Local changes to the vendored `deepapi` bundle

Read this before refreshing the bundle from upstream. It lists everything in this directory
that upstream did not put here, so a refresh does not silently undo a decision.

Upstream is `davidondrej/skills` → `skills/research-and-web/deepapi`. Refresh it with
`setup-matt-pocock-skills` or by copying the files in, then re-apply the rows below and run
`brain/scripts/install-skills.sh`.

## The upstream contract, in its own words

Every file under `references/` carries this header:

> "This file is always managed — it is refreshed with the bundle even when `../SKILL.md` has
> been customized."

Two consequences, both load-bearing:

1. **A customised `SKILL.md` survives a refresh.** Our trigger-condition change below is safe.
2. **Anything we add under `references/` does not.** It is overwritten without a word. That is
   why `local-api-reference.md` sits at the top level of this directory under a name no bundle
   file uses, and not in `references/` where it would look tidier and vanish.

## What we changed, and why

| file | change | why |
|---|---|---|
| `SKILL.md` frontmatter `description` | upstream auto-activates on any search / research / scraping task; ours activates **only** when `/deepapi` is called explicitly | matches the standing instruction in the user's global CLAUDE.md. Auto-activation sends ordinary searches through a paid API. **Re-apply after every refresh** |
| `SKILL.md` router table | one row added, pointing at `local-api-reference.md` | without it the fuller reference is in the directory but unreachable from the skill |
| `local-api-reference.md` | added, not from upstream | the full API reference, 83 endpoints against the 58 the bundle's own docs mention — GitHub repo/contents/issues, Amazon, Instagram, Facebook groups, email drafts. Fetched 2026-09-13. Keep it until the bundle covers them |

## Version taken

`1320a0ca895c`, with `references/generate-image.md` and the SKILL.md image-model count taken
from upstream `00b181aa4f09` on 2026-09-14 — that build adds `seedream-4.5`,
`gpt-image-2.5-sunburst` and `gpt-image-2.5-flare`, and revises the cost caps. Everything else
in `00b181aa4f09` was byte-identical to what we already held, except example email addresses in
`send-email.md`, which carry no meaning and were left alone.
