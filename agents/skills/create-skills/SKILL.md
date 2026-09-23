---
name: create-skills
description: >
  Use when someone asks for a new skill or slash command, or describes a repeatable procedure
  they want captured. Validates that the thing should exist at all and that a skill is the
  right shape for it, inventories what already does the job, grills the requirement until the
  build is unambiguous, checks outside prior art only where the approach is genuinely unknown,
  fixes an integration contract, writes it, verifies it triggers, and registers it.
  Rejecting the request is a complete and frequent outcome.
  Invoked as /create-skills <description | "from this session">.
---

You are creating a new skill. **The default outcome is rejection**, and a well-argued rejection
is worth more than a skill nobody invokes.

## This skill orchestrates. It does not teach.

Three documents already own the craft. Read the one you need, at the step that needs it:

| Layer | Where | Read it at |
|---|---|---|
| Is a skill even the right shape? | `learn-and-blend` — the four-category taxonomy | step 1 |
| Mechanics: frontmatter, structure, what makes a `description` actually trigger | `superpowers:writing-skills` | step 6 |
| Prose quality for an agent reader: hierarchy, progressive disclosure, co-location, positive phrasing | `writing-for-agents` | step 6 |

**Never restate their content here.** One home per fact (`AGENTS.md` §0). If one of them is wrong,
fix it there.

---

## 1 — FRAME & SHAPE

**Frame.** One sentence: *what will be different after this exists?* Not "we will have a skill" —
name the behaviour that changes, and who or what changes it. If the sentence cannot be written,
stop here and say so.

**Shape.** Run the four-category test (`learn-and-blend`). The deciding question, in order:

```
Must it work even when the agent does not want it to?     → guardrail (a hook). Not a skill.
Multi-step, needs judgement, accumulates context worth
  nurturing across runs?                                  → agent profile (/create-agents)
Will someone deliberately invoke it as a named procedure?  → skill. Continue.
Deterministic computation, no judgement?                   → script in brain/scripts/
```

A procedure written as a skill that should have been a guardrail will be skipped exactly when it
matters, because an agent about to break a rule is an agent that has already stopped reading it.

**Reject out loud** when the answer is not "skill". Name which of the four it is and where it
should be built instead. That is a finished deliverable.

---

## 2 — INVENTORY

Before reading anything external, list what already does part of this job:

```bash
ls .agents/skills/ ~/.agents/skills/           # skills, ours and vendored
ls .claude/commands/                           # project slash commands
cat docs/conventions/commands.md               # the registry — what is supposed to exist
ls .agents/agents/                              # profiles that might already own this work
grep -rl "<the capability>" brain/scripts/     # a script may already do it
```

For each overlap: **absorb it, extend it, or say precisely why the new thing is different.**
Two skills that partly overlap is the worst outcome — the caller picks by coin-flip and neither
gets improved.

---

## 3 — GRILL

Ask only questions whose answers change the build. Settle everything else yourself from the code
and say that you did.

- Vague requirement, or an unclear multi-session shape → `/wayfinder`.
- Otherwise → one round of `/decision`, then build.

Four questions that expose most bad skill requests:

1. **Who invokes it, and at what moment?** A skill nobody has a reason to type is dead on arrival.
2. **What does it read, and what does it write?** If it writes nothing and decides nothing, it is
   a document, not a skill.
3. **What must it never do?** A skill with no prohibitions has not been thought about.
4. **How will we know it worked?** Name the observable difference.

---

## 4 — PRIOR ART (only when genuinely uncertain)

Skip this when the skill is writing down something we already do — outside examples cannot improve
a procedure we invented and run daily. Search when the approach itself is unknown to us.

When you do search: `/deepapi` — `POST /v1/search/web` (several phrasings) and
`POST /v1/scrape/github/search`, ranked by stars. Then **adjudicate item by item**: adopt / rewrite
/ reject, each with a reason (`learn-and-blend` step 2). Never import a structure wholesale; a
skill shaped for someone else's repo carries their constraints into ours.

State plainly when you searched and found nothing worth taking. That is a result.

---

## 5 — CONTRACT

Fix these before writing a line. They are what makes the skill reusable rather than a one-off:

**Judgment seam (mandatory).** Does any part of this skill classify, score, route, or
confidence-gate text or evidence? If yes, the contract's `reads` names the `jev` skill and
says which step consults it — then the five adoption criteria in
`docs/policy/verdicts-and-gates.md` § Machine judgments decide whether it is wired in or
noted as a seam for later. A skill with a judgment-shaped step and no jev decision recorded
is incomplete.

```
name          kebab-case, matches the directory
invocation    /name <args>, and what each argument does
audience      human | unattended — see below. Decides what may be depended on
callers       which pipelines, commands or agent profiles will invoke it
reads         paths and tools it consumes
writes        paths it creates or edits
never         the prohibitions
scope         repo .agents/skills/ (default) or elsewhere — see below
```

**Audience: ask who this runs for, and do not guess.** The answer changes what the skill is
allowed to depend on, and it is the one contract field that cannot be recovered from the code.

| audience | what it means | consequence |
|---|---|---|
| **human** | invoked in a session where a person is present and can answer | may depend on a skill that only a human can invoke; may stop and ask |
| **unattended** | reached from an agent profile, a sprint lane, a cron, or any run with nobody watching | **every dependency must be model-invocable end to end.** A skill that stops to ask is a skill that hangs until morning |

A skill can serve both; then the stricter rule applies — write it unattended-safe and let the
human path benefit.

### Dependency check — mandatory, and mechanical

A skill whose body tells an agent to call another skill inherits that skill's invocability.
`disable-model-invocation: true` in a skill's frontmatter is **a hard refusal, not a degrade**:

```
Skill wayfinder cannot be used with Skill tool due to disable-model-invocation.
```

The calling procedure stops dead at that line. Twenty-two of the installed skills carry that
flag, including `/wayfinder`, `/handoff`, `/triage` and every `grill-*` — so this is common, not
exotic. Measured 2026-09-18: a session following its own plan hit exactly this and could only
hand the step back to the operator.

Run this before writing the body, and again in step 6:

```bash
brain/scripts/skill_dependency_check.py <skill-name>
```

For every skill your new one names, it reports whether that skill is model-invocable. **Then
bring what it found to the operator** — the fix is a decision, not a default:

- **write a fallback** — keep the dependency and state in the body what to do when the call is
  refused ("this step is the operator's: ask them to run it"). Correct when the step genuinely
  needs a human.
- **drop the dependency** — reach the same outcome another way. Often available: `/wayfinder`'s
  research tickets are already AFK and resolve through `Skill("research")`, which carries no flag.

Choosing silently is the failure this check exists to prevent — an `audience: unattended` skill
with a locked dependency looks finished and hangs at 3am.

**Scope: `.agents/skills/` in this repository, by default.** That directory is symlinked into
`~/.claude/skills/` and `~/.config/opencode/skills/`, so a skill there is invocable from any
directory on the machine *and* travels to the rest of the fleet through git. A skill written only
into `~/.claude/skills/` exists on one machine and reaches no other — measured 2026-09-14: two of
four fleet machines had 1 of 41 global skills.
► `brain/scripts/install-skills.sh`

**If it targets named pipelines, stop and add an adapter instead.** A skill that hardcodes
`alpha-research` must be edited for every future pipeline. Put the per-pipeline facts in
`brain/config/pipelines/<id>.yaml` and let the skill read the contract.
► `brain/config/pipelines/README.md`

---

## 6 — WRITE + VERIFY

Write it following `writing-for-agents` (prose) and `superpowers:writing-skills` (structure).
Density over length: the reader is an agent that pays tokens for every line, so a line earns its
place by changing what the agent does.

Then verify three things, by running them:

1. **It triggers.** The `description` is the only thing a client matches on. State the situation
   that should invoke it and confirm the description actually covers that phrasing.
2. **It runs.** Walk one real case end to end — a case that already happened, not a hypothetical.
3. **Its paths resolve.** Every file the skill cites exists:
   ```bash
   grep -oE '`[a-zA-Z_.][a-zA-Z0-9_./-]+\.(md|py|sh|yaml|json)`' <SKILL.md> | tr -d '`' \
     | sort -u | while read p; do [ -e "$p" ] || echo "MISSING: $p"; done
   ```
   A pointer to a file that does not exist is worse than no pointer: it ends the search.

---

## 7 — REGISTER + REPORT

```bash
uv run brain/scripts/registry_check.py --fix    # the row; then read what it wrote
brain/scripts/install-skills.sh                 # link it into both clients on this machine
```

**File it under a category in the same edit.** `docs/conventions/commands.md` groups every
command and skill under one of six categories, and the gate refuses a row that sits under none.
Choose by **what the caller wants when they reach for it**, not by what the skill does inside —
the internals are almost never the reason it is invoked.
► `docs/conventions/commands.md` § Categories

`pre-commit` guard 3 refuses the commit while the registry disagrees with the disk, so this step
is not optional. If the skill changes what `AGENTS.md` routes, run `/documentation` as well.

**Report, in this order:**

1. **What was built** — path, scope, and the one-sentence frame from step 1.
2. **Trigger scenario** — the situation that invokes it, in the words someone would actually use.
3. **Before → after** — what happened previously, what happens now.
4. **Strengths and weaknesses** — both. A report with no weakness column is not finished.
5. `SKILL GAP:` — one line on what this run needed that *this* skill did not provide.

---

## Never

- **Never write a fourth document about how to write skills.** Point at the three that exist.
- **Never build on "it might be useful".** Name the occasion (`AGENTS.md` §0).
- **Never leave a new skill only in `~/.claude/skills/`** — it reaches one machine.
- **Never ship without the trigger test.** A skill that never fires is indistinguishable from one
  that was never written, and costs more.
- **Never skip the rejection option.** It is the most common correct answer.

---

## Additive — AXI-shaped CLI skills (does not change steps 1–7)

Steps 1–7 above are unchanged. This section only fires when the thing you are creating is a
**thin wrapper around a CLI that agents will call via shell** (especially an
[AXI](https://github.com/kunchenguid/axi)-shaped tool).

### What this adds that we did not have before

| Gap we had | What AXI supplies |
|---|---|
| CLI-wrapper skills that paste full flag lists into `SKILL.md`, then rot | **Thin skill**: trigger + `npx -y <tool> --help` as the source of truth |
| No shared checklist for "agent will burn tokens on every byte of stdout" | Principles: minimal list schemas, truncation + `--full`, definitive empty states, next-step hints |
| Ambiguous empty CLI output → agent re-runs with different flags | Explicit zero-result lines |

It does **not** replace rejection, inventory, grill, register, or `install-skills.sh`.

### Extra checks when the skill wraps a CLI

After the contract in step 5 is fixed, also confirm:

1. `SKILL.md` body does **not** duplicate command flags or workflows — it points at
   `npx -y <tool> --help` / `<tool> <cmd> --help`.
2. `description` is outcome-shaped (when to load), not a flag catalogue.
3. Examples use `npx -y <tool>…` so a machine without a global install still works.
4. If the CLI is AXI-shaped, read `/axi` for the full principle list; do not invent a parallel
   doc.

House AXI skill: `.agents/skills/axi/SKILL.md`.
