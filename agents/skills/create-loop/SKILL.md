---
name: create-loop
description: Design and stand up a new unattended agentic loop workflow (a new /sprint-loop) from a problem statement — a 12-section definition questionnaire, then gated slice-wise build with a tracer bullet and a canary. Use when a repeated, multi-session, committing agent workflow is wanted; reject when a script, a skill, or one attended session suffices.
---

# Create-loop

audience human — the operator frames the loop; once defined, the loop itself runs unattended.

Harness decisions move outcomes as much as model selection (same model, GAIA 30.9% → 74.6%
on harness alone; same model + forced pre-completion verification: 52.8 → 66.5 Terminal-Bench).
This skill is how that control plane gets designed here, once, instead of re-derived per loop.
Harnesses also go stale as models improve (one generation's resets are the next's dead weight):
design stable interfaces that outlast implementations, and re-check every throttle when the
model generation changes.

## 0 — Is it a loop at all?

A loop is: repeated cycles · model-judged steps · unattended stretches · work that commits or
decides. Runs once → a task. Needs no judgment → a script in `brain/scripts/`. One session
with a person present → a plain skill. Say which, and stop if it is not a loop.

## 1 — The questionnaire

Write answers to `docs/plans/<slug>/00-loop-definition.md` before any code. "Inherited"
means: point at the house mechanism, never rebuild it.

**Section-scoring accelerator (2026-09-21, advisory).** After drafting, `/jev` may score each
of the 12 sections `sufficiently-specified / ambiguous` in one request; the operator then
answers only the flagged sections. The scores never unblock a section — "build with an
unanswered questionnaire section" remains forbidden below — and the step is skipped when the
key or API is absent. A component that is a controlled process rather than a model-judged
cycle is designed by `/create-pipeline`, which this skill may call for its pipeline-shaped
components.

1. **Goal & completion predicate.** One sentence that says done, as an objective-specific
   predicate — never "the model stopped calling tools" (a terminal message is not a satisfied
   goal). A machine-readable goal list agents may only flip (`passes: false → true`), never
   rewrite or delete. JSON over Markdown for agent-owned state.
2. **Metrics.** What the loop maximizes, per cycle and per run, and the ledger each number
   lands in. Separate productivity / efficiency / effectiveness — house pattern
   `docs/sprint-loop/metrics-audit.md`. No metric, no loop. Economics: history grows
   linearly, token cost quadratically — hold system prompt, tool order, and working set
   static so the prefix cache holds (up to 90% input-cost cut; a rotating ID or timestamp
   in the prompt silently breaks it). Report **model+harness pairs**, never model names
   alone — harness deltas are the size of model deltas.
3. **Components & I/O.** Table: component → reads → writes → owns. Two writers on one path is
   a design bug. Ledgers are append-only under `merge=union` (`.gitattributes`). Tool set:
   broad (shell, code exec) where the model is strong and the contract is loose; narrow
   where outputs are noisy or policies are strict (a constrained interface measurably beat
   raw shell ~2×); a sprawling endpoint list confuses the model and inflates cost.
4. **Memory & context.** Compaction alone does not bridge sessions: progress file + git
   history + goal list do (initializer writes them; every later session starts by reading
   them and smoke-testing before new work). Mark each read programmatic (always loaded:
   history, task, frontier) or agent-triggered (search, expansion). Prefer just-in-time
   loading via light search over preloading indexes. A context file is a **budget line, not
   a free prior**: LLM-generated ones measured −3% success / +20% cost; keep only
   non-obvious deviations. Guard the three memory failures: noisy retrieval, stale memory (date volatile facts), tool/schema overload
   (route via the registry — never enumerate everything).
5. **Agent roles.** Roles, ownership, and decision rights split by evidence depth, not rank —
   `docs/conventions/agent-hierarchy.md`. Each profile exists or is created via
   `/create-agents` (duties, prohibitions, authority rung, growth record). One unit of work
   per agent at a time; clean state at every boundary. Multi-agent rule: **one active
   writer, many read-only intelligence agents** — parallel writers diverge silently; every
   handoff starts a clean context with a precise goal. Training: a profile is trained, not
   rewritten — every close appends what the role missed or mishandled to its growth record
   (`/create-agents`), and the next run's profile is the trained version.
6. **Model pool.** Config is not membership. A model is usable when its capability-ledger row
   exists (`brain/scripts/README.md` — vision / sub-agent spawn / tool-calling, probed at
   onboarding; UNKNOWN = never probed, do not assume) and its quota pool is named in
   `brain/config/sprint-fleet.json`. Gateway profiles stay unassigned until per-machine
   onboarding completes — opt-in by construction.
7. **Stop conditions & stall detection.** Enumerate all: goal predicate · cycle cap ·
   wall-clock · budget · quota · operator STOP · loop's own failure exits (repeated identical
   actions, oscillation, no-progress window). Exit on "work stopped progressing", not only on
   "work finished". Name the drift guard: the characteristic long-run failure is **goal
   drift** — a small early error compounds until the agent debugs a problem it invented.
8. **Self-heal, tiered.** Tier 1 in-process failover (quota → next pool, nyroheal pattern);
   Tier 2 external watcher restarting a dead controller from its run manifest; Tier 3
   detached guardian — with the measured warning that the guardian itself was once the
   outage (`docs/conventions/agentic-loop-uptime.md`). Every healer is smaller and dumber
   than what it heals. Recovery assumes externalized state: append-only event log, idempotent
   tools, snapshot-before-write so a replayed step cannot corrupt the environment.
9. **Unattended safety.** Fleet-wide stop switch (jev-disable pattern) · per-run budget caps ·
   git-checkpoints C1–C5 verbatim (`docs/conventions/git-checkpoints.md`: one workflow one
   worktree, own ref, publish to survive, nothing unattended merges main, stopping is a
   checkpoint) · the only operator alarm path is `notify-operator.sh` — never stdout (a loop
   that died at 05:05 writing only to a log was found 7h32m later). Two measured traps:
   **approval fatigue** (humans approve ~93% of prompts — a loop that asks N times per cycle
   has no review signal; gate by risk, not by asking) and **instructional containment**
   (telling a model not to leak a secret is weaker than never letting the secret into the
   room — credentials live in the control plane, never the sandbox).
10. **Evidence & reporting.** Every verdict carries a receipt — verified by hash, not by
    claim. Reports generate from ledgers, not memory. Verification hierarchy: deterministic
    ground truth (tests, compilers, linters, gates) first, model-based graders second, humans
    last — and **the agent may never edit its own verifier** (reward hacking: agents rewrite
    the test to expect the broken output; lock verifier files against the loop). Keep
    trace-level telemetry: an infra regression can degrade output in ways no prompt review
    catches. Name the human surface (≤30-min read
    pack pattern) and the machine carryover the next run consumes.
11. **Iteration.** How the loop improves itself: `/pipeline-iteration` owns
    drain→diagnose→propose→replay→watch; meta-evaluation (funnel rates, recovery tests) is
    measured per run; a B1/B2 adoption rule decides auto-adopt versus operator decision.
12. **Knowledge capture & discovery.** At close: KB settlement for research notes; and in the
    same session as landing — registry rows, `AGENTS.md` router row, install-skills, system
    map. A loop that exists but is unrouted is invisible (measured 2026-09-21: `/jev` and the
    pi gateway sat unfindable in AGENTS.md until patched).

## 2 — Build order

1. Questionnaire → design doc → operator approval. Non-trivial machinery additionally runs
   `/software-architecture <slug>` gates (problem → structure → design → slices).
2. **Tracer bullet first**: one minimal end-to-end cycle, negative controls included, before
   any breadth (gateway slice-1 pattern).
3. Slices land separately, each with a falsifiable before/after; hermetic tests pin the
   pre-change failure by git-revert replay.
4. First real run is a canary with receipts — a batch smoke alone never validates
   interactive or overnight delivery.
5. Route and register everything in the landing session (the registry gate enforces it).

## Never

- Build with an unanswered questionnaire section.
- Let the loop merge to main, move tags, or edit research gates.
- Trust an unprobed model with a role.
- Let the loop edit its own verifier or tests.
- Automate a judgment gate without calibration on labeled history (the `/jev` rule).
- Ship a healer without enumerating what it could kill.
