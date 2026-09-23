---
name: jev
description: Run a bounded TypeSafe Jev judgment when text must be classified, scored, routed, or confidence-gated. Use for CRO proposal triage and other closed-option decisions; keep workflow actions in code. Delegates to the `typesafe-ai` skill when a way of asking is new, and ledgers every call so unattended runs stay auditable.
---

# Jev

audience unattended

Use Jev as a typed judgment inside an ordinary workflow. It returns choices, scores,
yes-probabilities, and probability distributions; it does not perform the work.

## Procedure

1. **Name the bounded decision.** Keep exact calculations, policy checks, file changes,
   execution, and side effects in code.

2. **Decide whether this is a NEW way of asking — then read the docs only if it is.**

   ```bash
   uv run brain/scripts/jev.py REQUEST.json --fingerprint
   ```

   It prints the sha256 of the canonicalised `questions` object and whether this workspace has
   asked it before (key order and whitespace do not change it; changing an instruction or a
   criterion does).

   - `seen_before: false` → **load the `typesafe-ai` skill** (or `/typesafe`) and read the
     current primitive and API pages before building the request. A way of asking is designed
     once, against live guidance.
   - `seen_before: true` → do not read the docs. Re-running a proven question against new
     state needs no refresh.

   Measured 2026-09-21: the docs index is ~2,300 tokens and one primitive page 6,400–10,700,
   against 2,000–4,000 for a typical CRO triage request. Refreshing on every call costs 4–6×
   the call itself, on four machines, mostly unattended. The fingerprint is what makes
   "always be current" affordable.

   **The gap this leaves, and it is real.** The fingerprint sees text, not meaning. A question
   whose wording is unchanged but whose *intent* has moved — the same words pointed at a new
   kind of proposal — reads as `seen_before: true` and skips the docs. When you know the
   meaning shifted, load `typesafe-ai` anyway and say why.

3. **Build one JSON request** with current `state`, `model: "jev-latest"`, and narrow
   independent `questions`. **Batch every independent question into that one request** —
   measured 12.2× cheaper and 10× faster than one request per question, with identical
   answers (parallel-questions cookbook, docs.typesafe.ai). A battery asked one-call-per-
   question is a bug, not a style choice:
   - `choice`: one option from a complete closed set, including `other` or `unclear` when needed.
   - `noul`: probability that one yes/no condition holds.
   - `score`: degree across concrete ordered levels.

4. **Show the request before its first live run.** Remove outcomes, future knowledge, and
   secrets. A historical evaluation must be blind to its recorded settlement.

5. **Run it:**

   ```bash
   uv run brain/scripts/jev.py REQUEST.json
   ```

   Human sessions may keep the key at `~/.typesafe/api_key` with mode `600`. Unattended callers
   inject `TYPESAFE_API_KEY`; a missing credential is a hard failure.

   Every run appends one row to `brain/ledger/jev_calls.jsonl`: timestamp, device, caller,
   interactive or not, fingerprint, question ids, status, latency, and each answer with its
   confidence. **The row never contains `state`** — state carries alpha research and this
   ledger is committed and pulled onto every machine in the pool. Set `JEV_CALLER` when
   invoking from a lane or profile, so the row names what asked.

   **The stop switch.** `~/.typesafe/DISABLED` halts every Jev call on that machine before
   it is made. `zsh brain/scripts/jev-disable.sh "<reason>"` places it across the fleet and
   `--off` lifts it; `--status` says who is stopped. It is fail-safe — present means stop,
   manual calls included — because `isatty()` cannot tell an operator's own run from a lane's
   (measured 2026-09-21), and a switch that silently fails to stop what it names is worse
   than none. A single deliberate call overrides with `JEV_FORCE=1`. Blocked calls still
   ledger, with status `disabled` and the reason, so a quiet night is distinguishable from a
   stopped one.

6. **Report the raw answer first:** model, selected answer, full probabilities or Noul value,
   confidence, and token usage. Then state what ordinary code would do with it.

7. **A/B the judgment** by changing the evidence while holding questions fixed. If the route
   does not change appropriately, revise the state or criteria before adding more questions.

## Authority boundary

Jev may route work to the next bounded step. The existing owner still decides and executes it.
For CRO proposals, Jev may classify the proposal, judge whether observed evidence and a
falsifiable test are present, and route to evidence collection, controlled replay, or operator
settlement. It does not adopt, reject, promote, merge, edit policy, or consume OOS.

As of 2026-09-20, every workspace result is advisory. Before any automatic gate or
agent-profile dependency, blind-test labeled historical cases and measure by confidence band.

## Measuring whether it earns automation

**The sample is 25 proposals, and that decides the design.** Counted 2026-09-21: four
`CRO_PROPOSAL.md` files carry 3 proposals each in the current format, four older
`CRO_REPORT.md` files (2026-09-09 to 09-11) carry 3 each in an earlier one, and one proposal
file exists in two identical copies. So roughly 12 + 12.

At that size, splitting confidence into 0.10-wide bands puts 2–3 cases in each, and a band
where every case is correct still only supports "the true rate is between 0.44 and 1.00".
That is not a measurement. What 25 cases CAN answer is a single threshold:

| What to measure | Why it is the one worth having |
|---|---|
| accuracy at confidence ≥ 0.90 vs below it | ~15 cases land above. All correct gives [0.80, 1.00] — a statement strong enough to gate on, and exactly the claim the vendor material makes |
| the new-format 12 and the older 12 **reported separately, then pooled** | agreement across formats earns the doubled sample; disagreement is the more useful finding, because it says past practice changed |

**Five runs per case, averaged.** Measured 2026-09-21: ten runs of one identical request
returned 0.29–0.33, sd 0.0117. One call is a sample, not the model's answer. Five cuts the
mean's noise to ~0.005, which keeps cases off the wrong side of the 0.90 line, and costs
about 600 tokens a call. Group the ledger by `questions_sha256` to recover the spread.

Two label sources, and the path between them is the point:

| `label_source` | What it means | When it is valid |
|---|---|---|
| `operator` | a person read the proposal and stated the right answer | always; the starting point |
| `historical_settlement` | what the workspace actually did with that proposal at the time | only after it is shown to agree with `operator` labels |

Run both over the same cases first and report their disagreement rate. Low disagreement means
past settlements carry no systematic bias and can label future cases without a person — which
is how the human leaves the loop on evidence rather than on intent. High disagreement is the
more valuable finding: it means past practice is the thing to fix, not the benchmark.

**Every band reports its own n and a bootstrap interval.** A report that prints `0.88` for the
0.80–0.90 band without printing `n=4, 95% CI [0.25, 1.00]` beside it is worse than no report:
it reads as a measurement and is noise. The reference article's headline `255/255 above 90%
confidence` came from synthetic data with only 11 examples in the critical 0.80–0.90 band —
treat cookbook thresholds and demo results as examples to evaluate, not as universal rules.

Typed output guarantees the interface, not truth. System One models are trained for calibrated
decisions; validate their performance in the target domain.

## Failure triage

Test representative cases and the resulting application behaviour. For failures, inspect the
exact state, questions, candidates, answers, composition, and observed outcome. Separate
missing evidence, model errors, code errors, and service failures. Preserve a deterministic or
human fallback for service failure and low confidence. Keep API credentials server-side in web
apps.

**Benchmark receipt (2026-09-21).** The 25-case blind CRO-triage benchmark ran:
`brain/pipeline-iteration/2026-09-21_jev-benchmark/REPORT.md` — 21/21 correct at confidence
≥ 0.90 in each format band, bootstrap CI [1.0, 1.0]; the duplicate case voted identically 5/5.
Per the verdict rule above, `proposal_class` at ≥ 0.90 is the one question class carrying
binding routing weight today (`docs/policy/verdicts-and-gates.md` § Machine judgments); every
other question class remains advisory until its own band passes.
