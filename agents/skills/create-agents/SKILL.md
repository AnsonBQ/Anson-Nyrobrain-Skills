---
name: create-agents
description: >
  Use when someone asks for a new agent profile, a specialised sub-agent, or describes a job
  they want an agent to own over time. Decides first whether a profile is warranted at all —
  a fixed procedure is a skill, not an employee — then writes the job description, assembles
  its toolset from skills that already exist, fixes what it may decide alone versus what needs
  the operator, and opens the growth record that lets it be trained rather than rewritten.
  Rejecting the request is a complete and frequent outcome.
  Invoked as /create-agents <description of the job>.
---

You are hiring. **An agent profile is an employee, not a document**, and the difference is
whether anything about it improves after it runs.

The test that separates this skill from `/create-skills`:

```
Fixed procedure, same steps every time            → a skill.   Reject; point at /create-skills.
Must fire whether or not anyone invokes it        → a guardrail (hook). Reject.
Deterministic computation                          → a script. Reject.
Multi-step · needs judgement · long-running ·
  accumulates experience worth nurturing           → an agent profile. Continue.
```

**Reject loudly when it is not a profile.** A profile built for work that is really a procedure
costs a dispatch, a context window, and a file someone must maintain — and delivers exactly what
a skill would have.

---

## 1 — FRAME & SHAPE

Name the job in one sentence: *what does this agent own that nobody currently owns?*

Then four questions. **Any "no" is a rejection**, stated with the reason:

1. **Does it repeat?** A one-off is a task, not a role.
2. **Does it need judgement?** If every decision follows from a rule, write the rule.
3. **Does it run long or across many turns?** A single-turn job belongs in the caller.
4. **Will it get better from feedback?** If nothing about the next run depends on this one, there
   is nothing to nurture and a skill is cheaper.

Check the existing roster first — this job may already belong to someone:

```bash
ls .agents/agents/ && cat .agents/agents/README.md
ls docs/sprint-loop/roles/          # the sprint hierarchy: master, leader, sub-agents, CRO
cat docs/conventions/agent-hierarchy.md
```

Two profiles that overlap is worse than one that is too broad: work gets routed by coin-flip and
neither accumulates enough experience to be worth training.

---

## 2 — JOB DESCRIPTION

Written from the outside in. The prohibitions are as load-bearing as the duties:

```
owns            the decision this agent makes that nobody else makes
reports to      its place in the hierarchy (docs/conventions/agent-hierarchy.md)
invoked         the moment that calls for it
NOT invoked     the moment it must stay out of
must never      the actions that are wrong even when they look helpful
done            the observable that says its run finished, not "it stopped"
```

`docs/sprint-loop/roles/chief-research-officer.md` is the house exemplar: it names *"never judge a
single candidate"*, *"never change anything while a run is live"*, and *"never move a gate's
threshold to make a night look better"* — three prohibitions that took incidents to learn.

---

## 3 — TOOLSET

Assemble from what exists before writing anything new:

- **A skill already does it** → name it in the profile and move on.
- **A capability is missing and would generalise to other profiles** → build it with
  `/create-skills`, then name it here. Generalisable work belongs in a skill even when only one
  agent needs it today, because the second caller arrives without warning.
- **Highly specific to this agent, or likely to change every time we read its feedback** → write
  it inline in the profile's own markdown. A volatile procedure forced into a shared skill makes
  every other caller absorb this agent's churn.

Restrict `tools:` to what the job needs. A read-only reviewer that holds `Write` will eventually
use it.

---

## 4 — AUTHORITY LADDER

State what it may do alone and what stops for the operator. Reuse the ladder already proven on
the CRO rather than inventing a new vocabulary:

| rung | what it may adopt alone |
|---|---|
| **B1** | nothing — every change is the operator's |
| **B1.5** | plumbing (delivery, paths, wiring, templates) on a passing test. Anything that changes a number a gate reads is not plumbing |
| **B2** | research decisions too — only after ≥5 consecutive adoptions with no rollback **and** ≥3 verified in a real run |

New profiles start at B1 unless there is a measured reason not to. Write the promotion criteria
into the profile so a future session can read whether it is time, instead of asking.

**Judgment seam (mandatory).** Any rung whose decision is a closed-option judgment on text
or evidence (classify / score / route / confidence-gate) must say whether the `jev` skill
owns it, and at what confidence it binds (≥0.90 only, and only for question classes whose
blind benchmark band passed — `docs/policy/verdicts-and-gates.md` § Machine judgments).
Machine judgment never raises the rung: a B1.5 agent with a benchmarked classifier is still
B1.5 — what it may *adopt* alone does not grow, only the quality of its *advice* does.

---

## 5 — WRITE

`.agents/agents/<name>.md`, following the shape of the existing roster:

```yaml
---
name: <kebab-case>
description: <when to dispatch it — this is what routes work to it; make it specific>
tools: <only what the job needs>
model: <claude-opus-5 for judgement-heavy roles; a smaller model where the job is mechanical>
---
```

Where a sprint role already owns the full contract in `docs/sprint-loop/roles/`, the profile is a
**thin pointer** at that document, not a second copy. Two descriptions of one role drift, and the
one the agent happens to load wins.

---

## 6 — VERIFY

1. **Dispatch it on a case that already happened** — a past run, a real file, a decision already
   made — and compare its output against what actually occurred. A profile validated on a
   hypothetical has been validated against your own imagination.
2. **Check the routing.** Does its `description` actually cause this job to reach it? Describe the
   task the way a caller would and confirm the match.
3. **Check the prohibitions bite.** Hand it something it must refuse and confirm it refuses.

---

## 7 — REGISTER + OPEN THE GROWTH RECORD

```bash
uv run brain/scripts/registry_check.py --fix    # roster row in .agents/agents/README.md
```

Then create `.agents/agents/growth/<name>.md`. **This is the step that makes it an employee.**

Every change to the profile is recorded *before* the evidence exists, and settled afterwards by
real runs:

```markdown
## 2026-09-14 — <what changed in the profile>
PREDICTION: <what should get better, and the exact observable to read>
SETTLE AFTER: <how many real runs>
VERDICT: PENDING | BETTER | WORSE | NO-CHANGE
EVIDENCE: <what was actually read, with paths>
```

The prediction is written first and **never rewritten to match what was found** — that is the
whole mechanism. A `WORSE` verdict is a normal outcome and opens a rollback: revert the profile to
the previous version (git holds it) and record why.

Same discipline as `brain/pipeline-iteration/WATCH.md`, for the same reason: a change proven on
the material it was designed against has not yet been proven to generalise.

**Report:** what was hired, what it owns, what it must never do, its rung on the ladder, and the
first prediction now open in its growth record.

---

## Never

- **Never hire for a fixed procedure.** That is a skill, and saying so is the deliverable.
- **Never write a profile with no prohibitions.** It means the job was not thought through.
- **Never copy a role contract that already lives in `docs/sprint-loop/roles/`** — point at it.
- **Never start a new profile above B1** without a measured reason.
- **Never ship a profile without a growth record.** Without one it cannot be trained, only
  rewritten from scratch — and the next person rewriting it will not know what was already tried.
