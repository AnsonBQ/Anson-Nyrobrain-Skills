---
name: learn-and-blend
description: >
  Use when taking something from someone else's repository, skill, document or codebase into this
  workspace. Forces an inventory of what we already have BEFORE reading the reference, adjudicates
  the reference item by item into adopt / rewrite / reject with a stated reason for each, chooses
  the right shape for whatever survives (skill, agent profile, guardrail, or script), and closes
  with a before/after report naming every file created and why. Invoked as /learn-and-blend <url or
  path> [what we are hoping to get from it].
---

# Learn & blend

Someone else solved a problem you also have. Their solution encodes **their** constraints, and you
cannot see those constraints from inside their repository. Copying it imports their constraints
along with their code, silently.

This skill is the discipline that stops that. It is not "read and adapt" — it is a specific order
of operations, and the order is what does the work.

## The failure it exists to prevent

A real one, from this workspace:

> A public skill called `git-guardrails` blocks **all** `git push` as a safety measure. In its
> author's workflow that is reasonable — push is publication, and an agent should not publish.
>
> In this workspace push is not publication. It is the **durability mechanism**: a commit that is
> never pushed dies with its worktree, and `brain/scripts/sprint_runtime.py:639-647` records
> fourteen research plots already lost that way. Installing that skill unchanged would have
> mechanically enforced the exact failure we were trying to fix.

Same command, opposite meaning, because the surrounding system is different. You cannot catch that
by reading their code carefully. You catch it by knowing your own system first.

---

## Step 1 — Inventory ours FIRST. Before opening theirs.

**Do this before you read the reference in any depth.** Reading theirs first anchors you: every
gap they name starts to look like a gap you have, and you end up importing solutions to problems
you do not possess.

Write down, from our own tree:

1. **What already addresses this?** Name files and line numbers. Read them.
2. **What did each of them cost?** Most defences here were paid for by a named incident. Find the
   incident. A defence with an incident behind it outranks any external design, because it is
   fitted to a failure that actually happened here.
3. **Where are we genuinely weaker?** State it as a sentence with a mechanism: not "we have no
   guardrails" but "nothing fires before an agent runs a command; every defence we have is a rule
   the agent must choose to remember."

Produce this as a short table before going further. If it is empty, say so explicitly — that is a
finding, and it changes how much of the reference you should take.

## Step 2 — Adjudicate the reference, item by item

Never "adopt the skill". Adopt or reject **each piece**, with the reason attached.

| item | ours today | verdict | why |
|---|---|---|---|
| ... | file:line, or *nothing* | adopt / rewrite / reject | one sentence, concrete |

- **adopt** — take it as-is. Legitimate when the thing is genuinely general.
- **rewrite** — the idea is right, the implementation assumes their world. Say which assumption.
- **reject** — say what about our system makes it wrong *here*. "We already have it" is a reason;
  so is "it assumes npm and we use uv"; so is "it fails open and we cannot tolerate that".

A reference that comes out 100% adopt means you did not understand it. A reference that comes out
100% reject means you should not have spent the time — say that too.

**Reject loudly when the reference would hurt.** The `git push` example above is the template:
name the mechanism, name what it would have cost, then move on.

## Step 3 — Choose the shape for what survives

There are **four**, and picking wrong is the most common way good work becomes invisible.

| shape | English | fires when | use for |
|---|---|---|---|
| 技能 | **skill** (`/command`) | someone invokes it | a fixed repeatable procedure with a name |
| 员工 | **agent profile** | dispatched into a session | multi-step work needing judgement, improved over time |
| 护栏 | **guardrail** (a **hook**) | **automatically, on an event** | things that cannot rely on self-discipline |
| 工具 | **script** | called by a human or a skill | deterministic computation, no judgement |

Decide in this order:

```
 Must it work even when the agent does not want it to?          -> guardrail
   otherwise
 Does it need judgement across many steps, and accumulate
   context worth nurturing over months?                          -> agent profile
   otherwise
 Will someone deliberately invoke it as a named procedure?       -> skill
   otherwise                                                     -> script
```

The first question is the load-bearing one. **An agent about to run a destructive command is, by
definition, an agent that has already stopped following the rule against it.** A rule only binds
the agents that are already behaving; that whole class must be infrastructure that fires before any
agent chooses.

Most non-trivial imports become **two** things: a script that does the work, and a skill or profile
that knows when and why to call it. That is correct, not duplication — the script is deterministic
and testable, the wrapper carries the judgement.

## Step 4 — Build it as ours

- **Write down every divergence from the reference, and why.** A future reader will diff against
  upstream and needs to know which differences are deliberate.
- **Name the incident.** If a defence exists because something went wrong here, put the date and
  the cost in the file. `AGENTS.md` §0: *a defence must be paid for by an observed failure, not an
  imagined one.* If you are importing a defence with no local incident behind it, say so out loud
  and say on whose instruction — do not let the exception become quiet precedent.
- **Verify the effect, never the invocation.** Registering a hook is not evidence the hook fires.
  Send a real dangerous command through it and watch it get refused.
- **Check it does not break what is already running.** Enumerate the commands the existing
  automation actually issues and run every one through the new mechanism. A guard that blocks the
  nightly loop has caused more damage than it prevents.
- **Mirror it everywhere it belongs.** In this workspace: `.claude/commands/` and
  `.opencode/commands/` carry the same set; agent profiles live in `.claude/agents/`; global skills
  live in `~/.claude/skills/` and `~/.config/opencode/skills/`. A one-sided install is a silent
  feature.

## Step 5 — The closing report. Mandatory.

End the run with exactly these four sections. Not a summary of what you did — an account of what
now exists.

**1. What was implemented.** One paragraph, plain language.

**2. What was created, and where.** A table. Every path, absolute or repo-relative, with its shape:

| file | shape | what it does |
|---|---|---|

**3. Why — per item.** The reason each thing exists, and for anything adopted from the reference,
what was changed and why. Include the rejections and their reasons; what you refused to take is
half the value of the exercise.

**4. Before / after.** Concrete, and measured wherever a number exists:

| | before | after |
|---|---|---|

State the residual risk too. What is still exposed, what was deliberately left alone, what needs a
supervised follow-up. A before/after with no "still exposed" row is almost always incomplete.

---

## Never

- **Never read theirs before inventorying ours.** This is the whole skill.
- **Never adopt a whole reference.** Adjudicate pieces.
- **Never import a defence without naming either the local incident or the explicit instruction.**
- **Never install a mechanism without sending a real case through it.**
- **Never ship a one-sided mirror** (Claude but not OpenCode, script but no pointer).
- **Never let a blocking mechanism reach unattended automation untested.** Enumerate what the
  automation runs, and prove each one still passes.
