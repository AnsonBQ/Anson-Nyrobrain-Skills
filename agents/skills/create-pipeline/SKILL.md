---
name: create-pipeline
description: >
  Design and stand up a new controlled process pipeline — a repeatable, scheduled or
  on-demand job with typed inputs, deterministic steps, evidence receipts, and an iteration
  adapter — without requiring agentic loops. Use when someone describes a process to
  systematize (nightly mining, data hygiene, report generation, signal refresh, any "controlled
  process with jobs/functionality") that is not fundamentally model-judged cycles. When the
  process turns out to need unattended model judgment, stop and hand it to /create-loop, which
  may call this skill back for its pipeline skeleton.
---

# Create-pipeline

audience human — the operator frames the pipeline; once defined, it runs unattended on code and
schedules, with judgment only at named, bounded boundaries.

`/create-loop` builds workflows whose steps are model-judged. Most controlled processes are not
that: they are code, a schedule, and an evidence contract — and forcing them through loop
machinery buys hallucination risk where a script would do. This skill is the discipline layer
between "just write a script" (undiscoverable, uniterable, unaudited) and "stand up an agentic
loop" (create-loop's territory): the same questionnaire rigor, tracer-first build, and
registration, minus the model-judged-cycle machinery. It ends by registering the iteration
adapter, so `/pipeline-iteration` can improve the pipeline from day one.

## 0 — Pipeline, loop, script, or existing?

Run the ladder, in order:

1. **Already exists?** Check `brain/config/pipelines/` adapters, `docs/pipelines/`,
   `brain/scripts/README.md`. Extending an iterable pipeline = `/pipeline-iteration`, not a new
   build. Signal-generation branches (S/M/ML/OS/TS) follow
   `docs/pipelines/pipeline-extension-contract.md` — not this skill.
2. **Runs once, no judgment?** A script in `brain/scripts/` with a README row. Done.
3. **Repeated model-judged cycles, unattended, committing?** `/create-loop`. It may call this
   skill for its deterministic pipeline components.
4. **Repeated, deterministic or mostly-deterministic, evidence-producing?** → this skill.

Say which rung held, and stop if it is not 4.

## 1 — The questionnaire

Write answers to `docs/plans/<slug>/00-pipeline-definition.md` before any code. Inherited means
point at the house mechanism, never rebuild it. `/jev` may advisory-score the sections
`sufficiently-specified / ambiguous` so the operator answers only the flagged ones.

1. **Goal & completion predicate.** One machine-checkable sentence that says done — a state a
   script can verify, never "the run ended". Name the artifact whose existence + hash is the
   receipt.
2. **Trigger & cadence.** cron / LaunchAgent / GitHub Actions / event / manual. House patterns:
   route-A single-writer (`deribit-chain-archive.yml`), LaunchAgent KeepAlive (kbshim), MINI
   hourly sync. Name the missed-run behaviour (a snapshot-only source that skips a day has lost
   it forever — that decides the scheduler).
3. **Inputs & evidence contract.** Each input: source, point-in-time rule, availability lag,
   vintage discipline. If it touches market data, the four disciplines apply
   (`docs/datasources/README.md`) and daily-stamped inputs carry the 24h lag. Unbackfillable
   inputs are named as such.
4. **Steps & ownership.** Table: step → deterministic code | named bounded judgment | human.
   Judgment steps name their instrument (`/jev` question class, or an agent step — which
   escalates this build to /create-loop for that component). Two writers on one path is a
   design bug. Steps are idempotent under replay.
5. **Outputs, ledgers & receipts.** Append-only ledgers under `merge=union` where multi-machine;
   every output verifiable by hash; reports generated from ledgers, not memory. Name the human
   surface if a person reads anything (≤30-min pack pattern). Derived `.parquet`/plots are
   cache unless committed code reproduces them — never archived for tidiness.
6. **Failure & observability.** Failure is loud: non-zero exit, `brain/scripts/notify-operator.sh` for
   unattended runs (never stdout — the 05:05 lesson), a staleness sentinel where a silent
   no-op is the failure mode (kb-sync pattern). Stop switch named. Retry/idempotency under the
   trigger firing twice.
7. **Iteration & generalisation.** The adapter written at landing
   (`brain/config/pipelines/<id>.yaml`): `evidence:` paths, replay corpus, invariants — the
   contract `/pipeline-iteration` reads. A pipeline without an adapter cannot be iterated, only
   patched.
8. **Routing & discovery.** AGENTS.md router row (if a human or agent must find it by task),
   `brain/scripts/README.md` rows, commands.md row if invocable — all in the landing session.

## 2 — Build order

1. Questionnaire → design doc → operator approval. Non-trivial machinery additionally runs
   `/software-architecture <slug>` gates.
2. **Tracer run first**: one minimal end-to-end execution on real input, negative control
   included, before any breadth. Its receipt is the design's first evidence.
3. Slices land separately, each with a falsifiable before/after test.
4. **Canary period**: the first N scheduled runs are watched (staleness sentinel + one manual
   read-back), then a WATCH row records the generalisation verdict before it is trusted alone.
5. Register adapter + router rows in the same landing session (the registry gate enforces it).

## Never

- Build with an unanswered questionnaire section.
- Put unbounded model judgment inside a pipeline step without escalating to `/create-loop` for
  that component — or without a named `/jev` question class that meets
  `docs/policy/verdicts-and-gates.md` § Machine judgments criteria.
- Ship a pipeline that fails silently (no non-zero exit, no sentinel, no alarm path).
- Land without the iteration adapter — an uniterable pipeline is a script with ceremony.
- Leave it unrouted: a pipeline AGENTS.md cannot name is invisible.
- Duplicate a mechanism the house already owns; inherit it.
