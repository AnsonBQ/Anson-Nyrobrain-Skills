---
name: decision
description: Use when a discussion has produced more information than the user can act on - long answers, dense jargon, or a tangle where it is unclear what needs deciding versus what is just a statement. Converts everything on the table into a decision questionnaire of multiple-choice questions the user can answer one by one, written for a non-engineer, so the work can move forward instead of stalling on a wall of text.
---

# Decision

The user has just been handed more than they can act on. Your job is to compress it into the
smallest set of choices that, once answered, unblocks the work — and to write those choices
so that someone who does not code can pick correctly.

This is not a summary. A summary tells them what happened. A questionnaire tells them what
they must decide, and what changes depending on how they decide it.

## The rule that makes this skill work

**Only ask what the user actually owns.**

Before writing a question, ask yourself: *can I settle this myself from the code, the
measurements, or a sensible default?* If yes, settle it, state that you did, and do not ask.
Every question you ask costs the user attention they wanted back.

A question belongs in the list when, and only when, different answers lead to genuinely
different work — and the difference turns on the user's taste, risk appetite, priorities or
business context rather than on a fact you could go and look up.

Facts are your job. Decisions are theirs. If a question needs a fact you do not have, go get
the fact first (read the code, run the thing, dispatch a subagent), then ask the question
with the fact already in it.

## Who you are writing for

Write as if the reader has never opened a terminal and does not know what a shell, a
process, a pane, a daemon, a seam, or a token is.

- **No jargon without a plain-language gloss.** The first time a technical term is
  unavoidable, explain it in one middle-school sentence, inline, right there. Do not send
  them to a glossary.
- **Name things by what they do, not what they are called.** "The thing that watches whether
  a job is still alive" beats "the supervisor poll".
- **No file paths, no flags, no function names in the question or the options.** Those belong
  in your own notes, not in a decision aid. If a path is genuinely the subject of the
  decision, describe it: "the folder where finished work gets filed".
- **Concrete over abstract.** "It would cost about 35% more per helper" beats "there is a
  token overhead".

Keep the technical precision in the artifacts you write to disk. Keep it out of the question.

## Surface the edge cases

The most valuable thing this skill does is drag the small, easy-to-miss consequence into the
light — the one that looks like a footnote and turns out to decide the whole thing.

When you know of one, it goes in the option's description in plain words, even if it is
unglamorous. "If you pick this, a single permission popup from a helper freezes the whole
job until someone clicks it — measured at 7 minutes one night" is worth more than three
paragraphs of architecture.

If an edge case is important but does not attach to any one option, state it in one or two
lines of prose immediately before the questions. Do not bury it.

## Question format

Use the **AskUserQuestion** tool so the user can click rather than type. Up to 4 questions
per round; 2-4 options each. Ask only the questions that are answerable *now* — a question
whose answer depends on another question in the same round belongs to the next round.

Put your recommended option first and mark its label `(Recommended)`.

**Every option's description must carry all three of these, in this order:**

1. **好处 / Benefit** — what you get, stated concretely.
2. **代价 / Cost and risk** — what it costs, what could go wrong, what you give up.
3. **前后差异 / What visibly changes** — what is different after choosing this versus not.
   The thing they would actually notice.

Do not write an option whose description is only its benefit. An option with no stated cost
reads as the obvious answer and corrupts the choice.

Where a real number exists, use it. Measured beats estimated; estimated beats adjectival.

## Around the questions

**Before:** two or three sentences at most. What was decided since last time (so they can see
progress), and any edge case that does not fit inside an option. Not a recap of the whole
discussion — they asked for this skill precisely because the recap was the problem.

**After each round:** record the answers where they belong (the ticket, the map, the spec —
whatever this effort uses), then recompute what is now answerable and ask the next round.
Tell them roughly how many rounds are left so the end is visible.

**When the user's answer contradicts something binding** — an existing rule, a measurement, a
constraint they themselves set earlier — say so once, plainly, in one or two sentences, and
offer to re-ask. Then respect their decision. Do not re-litigate an answer they have
reaffirmed.

**When an answer is a question back at you**, that is a signal the option descriptions were
not concrete enough. Go get the missing fact and re-ask with it included, rather than
explaining harder.

## What this skill is not

- Not a status report. If they wanted to know what happened, they would have asked.
- Not a way to offload work. Questions you could answer yourself do not belong here.
- Not a plan. The output is decisions; the plan follows from them.

##FINAL WORD
While working on this, which important decisions / choices did you make, that you are not confident about?

Think about this deeply, reason about all the important decisions made, and think whether these decisions have any other great alternatives that we have not considered.

DO NOT list out the choices / decisions where we already have the best possible solution.

Only list out the decisions you are really unsure about.

answer in short. be very concise.
