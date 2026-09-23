---
name: software-architecture
description: Design-first workflow for non-trivial changes to the workspace's own machinery — sprint runtime, hooks and guards, brain/scripts, OMS, agent/skills plumbing. Use when a change will touch several files, redesign an existing component, or produce a diff too large to review at once; also when pipeline-iteration or create-skills needs a full pre-design for something it is building. Not for alpha cards or research-pipeline work (the 8-step funnel owns those), not inside sprint lanes or any unattended run, and not for trivial tweaks.
---

# Software Architecture

Make every important decision about a machinery change **before the code exists**, where
changing it costs a sentence. Work through four gates in order — problem → structure →
detailed design → slice plan — with operator approval at each, then build in thin verified
slices.

House rewrite of Dex Horthy's 4-gate workflow (gist `Maciejdziuba/88890d7e`), imported via
learn-and-blend on 2026-09-17. Divergences from the reference are deliberate and listed at
the bottom; the decision record is `docs/plans/software-architecture/00-status.md`.

## When to run — and when never

Run the gates when ALL of these hold:

- The target is the workspace's own machinery: `brain/scripts/`, hooks and guards, sprint
  runtime, OMS, agent profiles/skills plumbing, client configs.
- The change is non-trivial: several files, a redesigned component, or a diff the operator
  would hate to review at once (~100+ lines is the rough line).
- A human can answer questions this session.

Never run the gates:

- Inside a sprint lane or any unattended context (a session working under
  `.sprint-worktrees/`, or anywhere no human will answer). Approval stops freeze unattended
  work — a measured failure class here. Record the need in the lane's decisions log (the
  DECISIONS.md convention in `docs/sprint-loop/README.md`) and surface it at close.
- For alpha cards or research-pipeline steps — the 8-step funnel owns those
  (`docs/pipelines/alpha-pipeline.md`).
- For trivial tweaks: renames, copy edits, one-line config changes.
- When the operator says skip ("fast version", "just vibe it") — note the skip in
  `00-status.md` and proceed without gates.

Unsure whether the task qualifies → ask once: "This looks big enough for the design gates —
run them, or fast version?" Respect the answer.

## Files and state

All design state lives in `docs/plans/<slug>/` (convention owner: `docs/plans/README.md`):

```
docs/plans/<slug>/
  00-status.md      gate approvals + slice checklist + notes for a fresh session
  01-problem.md
  02-structure.md
  03-design.md
  04-slices.md
```

Create `00-status.md` first. Update it at every approval and every completed slice.
Template:

```markdown
# Status: <feature>

- Gate 1 — Problem: pending | in progress | APPROVED <date>
- Gate 2 — Structure: pending | in progress | APPROVED <date>
- Gate 3 — Detailed design: pending | in progress | APPROVED <date>
- Gate 4 — Slice plan: pending | in progress | APPROVED <date>

## Slices
- [ ] Slice 1 — tracer bullet: <one line>
- [ ] Slice 2 — <one line>

## Notes for a fresh session
<decisions made in chat that a new session must know>
```

**Resume rule.** At the start of any session touching `<slug>`: if
`docs/plans/<slug>/00-status.md` exists, read every doc in that folder, then continue from
the first unapproved gate or first unchecked slice. Never redo an approved gate unless the
operator asks or a later gate invalidated it.

## The approval protocol (every gate)

1. Write the gate doc to disk.
2. Present ≤10 bullet decisions + the doc path. Never paste the whole doc into chat. (If
   the decision load exceeds what bullets carry, hand it to the `decision` skill —
   `.agents/skills/decision/SKILL.md`.)
3. Ask exactly: "Approve Gate N, or what should change?"
4. Approval = a clear yes. Anything else → revise the doc, re-ask.
5. On approval, mark APPROVED in `00-status.md` and move on.
6. Backtracking: later work reveals an approved decision is wrong → stop, update the
   earlier doc, set that gate back to in progress, re-approve.

## Gate 1 — Problem

Write `01-problem.md`:

```markdown
# Problem: <feature>

## Occasion
<the observed pain: an incident with date and cost, a measurement, or the explicit
operator instruction that commissions this. Per AGENTS.md §0, "it might be useful"
builds nothing.>

## Success measurement
<the number or observable that says this worked, and how it is read>

## Boundaries
<what this change must not touch, and what is explicitly out of scope>
```

Rules: no solution shape yet — a problem named as a missing solution ("we need a caching
layer") is sent back to be named as a pain. If the occasion cannot be written, stop and
say so; that is a valid outcome. Run the approval protocol.

## Gate 2 — Structure

Read the real code first — never design against an imagined codebase. Write
`02-structure.md`:

```markdown
# Structure: <feature>

## Fit
<existing components this touches — scripts, hooks, roles, runtimes — and how>

## State
<files, configs, registries, ledgers created or changed>

## Flow
<the end-to-end order of the main path: what calls what>

## External
<fleet machines, clients, hooks, schedulers, quotas, third-party APIs affected — or "none">
```

Run the approval protocol.

## Gate 3 — Detailed design

The step everyone skips: the decisions otherwise made silently mid-implementation. Write
`03-design.md`:

**Judgment seam (mandatory).** If any component classifies, scores, routes, or
confidence-gates text or evidence, the design names the seam: which question class, who
owns the label source, and whether `jev` is wired now or noted for later. A design that
leaves a judgment-shaped hole for "the model will figure it out" fails Gate 3 — that is
exactly the seam the five criteria in `docs/policy/verdicts-and-gates.md` § Machine
judgments exist to settle.

```markdown
# Detailed design: <feature>

## Files
<every file created or changed, one line each on why it lives there>

## Types & signatures
<code blocks: types and signatures, NO bodies — readable in seconds, judgeable right/wrong>

## Call stack
<per main flow: what calls what, top to bottom>

## Test plan
<test names and what each asserts — before any of them exist.
Every test must fail against the pre-change code; a test that cannot fail tests nothing.>

## Least-confident decisions
<numbered list of the calls most worth challenging now, while changing them is free>
```

Test conventions: stdlib `unittest` for anything that runs unattended (it must run with no
environment); plain-assert `uv run` scripts elsewhere. No new test framework enters the
project environment — recorded house decision, 2026-07-02. Run the approval protocol.

## Gate 4 — Slice plan, then build

Write `04-slices.md` — one line per slice, in build order — and approve it like any gate.
Then build one slice at a time:

- **Slice 1 is the tracer bullet**: the thinnest end-to-end version that runs and can be
  shown. It does almost nothing — but it runs.
- Slice 2: real logic for the single happy path.
- Slice 3+: one capability per slice, each ending in a working, tested state.
- Horizontal building is banned: never all-of-one-layer then all-of-the-next with nothing
  testable until the end.

After every slice:

1. Prove the effect — run it, and read the result back (house rule: verify the effect,
   never the invocation).
2. Check the slice off in `00-status.md`.
3. Ask: "Continue to slice N+1, or re-steer?" A wrong trajectory is fixed before more code
   exists.

## Standing rules

- **Compact at every boundary.** At each gate approval and each slice completion, the docs
  hold everything decided — nothing important exists only in chat. Say so when the folder
  is a safe resume point.
- **Real tests only.** Never weaken, skip, or comment out a test to reach green.
- **Register what you build.** Any new script, skill, command, or profile produced by the
  slices is registered in the same session (`docs/conventions/commands.md`).
- **Pipelines adopt this skill by adapter, never by name.** A pipeline that wants a design
  gate gets one through `brain/config/pipelines/README.md`'s adapter mechanism; this skill
  never hardcodes a pipeline.

## Divergences from the reference (deliberate)

Recorded so a future diff against upstream reads as decisions, not drift:

1. Gate 1 rewritten: incident/measurement/instruction framing replaces "blog post + HTML
   mockups" — no end-user product here; AGENTS.md §0 is the stronger product gate.
2. Approvals bind only in operator-present sessions; stop-at-every-gate everywhere would
   freeze unattended lanes (measured popup-freeze class).
3. Never-unattended prohibition added (`.sprint-worktrees/` detection).
4. Alpha cards explicitly excluded — the 8-step funnel is the heavier machinery.
5. Test rule strengthened: names + assertions before code, must-fail-pre-change,
   stdlib-only frameworks (recorded house decision, 2026-07-02).
6. `docs/plans/` is committed and registry-covered (`docs/plans/README.md`), not informal.
7. Pipelines integrate by adapter (`brain/config/pipelines/README.md`), never by hardcoded
   pipeline name.
8. The reference's ADR and `docs/external/` folders are rejected — policy files and the
   DECISIONS.md convention (`docs/sprint-loop/README.md`) already carry decisions with
   counterargument + reversal trigger.
9. Fast escape valve ("fast version") formalised and logged, not just implied.
