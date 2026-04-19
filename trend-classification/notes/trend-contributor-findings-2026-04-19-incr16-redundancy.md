---
title: Trend contributor redundancy study — increment 16
date: 2026-04-19
slice: pairwise Pearson r across the canonical fixture bank, 12-contributor uniqueness ranking, candidate redundancies
---

# Increment 16 — empirical contributor redundancy

## What this increment is

The inventory closed at increment 15 (12 of 12 direction-voting
classifiers wired into a single `TrendState`). The deferred question
is **weighting**: equal weights for all 12 was the safe default, but is
it the right default?

This pass measures, on the canonical 5-fixture bank, how much each
contributor moves with the rest of the stack. The metric is
Pearson r between every pair of contributors over a per-bar sample
bank (5 fixtures × ~26 bar-prefixes each = ~130 rows).

**No production code changed.** Pure addition: one new function in
`tools/visualize_trend_contributors.py`, one new figure
(`figures/contributor_agreement.png`), this note. The aggregator,
schema, and tests are unmodified. Scanner ranking is unmodified.

## How to read the figure

`figures/contributor_agreement.png` has two panels.

**Left — 12×12 correlation heatmap.** Contributors are reordered to
put the five families in contiguous blocks (directional, magnitude,
memory, structural, regime). Diagonal blocks visualise *within-family*
agreement; off-diagonal blocks visualise *between-family* agreement.
Family separator lines are drawn in white. Red = positive correlation,
blue = negative.

**Right — per-contributor uniqueness score.** For each contributor,
the bar height is the mean |r| with the other 11 contributors. Lower
bars = the contributor carries more independent signal (less reducible
by removing one of its peers). Higher bars = the contributor moves
with the rest of the stack — equal weighting is silently
double-counting it. Bar colour = family. The vertical dotted line is
the bank average.

## Findings — at a glance

| Finding | Headline |
|---|---|
| **F1** | `trending_swings` carries **2× more unique signal** than any other contributor (|r|_avg = 0.335 vs ~0.77 for the next-best). |
| **F2** | `small_pullback_trend` ↔ `bpa_trend_bar_density` correlate at **r = +0.997** — near-duplicate votes inside the magnitude family. |
| **F3** | Most contributors sit at |r|_avg ≈ 0.77–0.88 — **equal weighting is double-counting** the bulk of the stack. |
| **F4** | Several cross-family near-perfect correlations are **sample-bank artifacts** (synthetic, polarized fixtures), not production properties. |
| **F5** | `trending_swings` is the **only direction-voter that fires on pullback sessions and not on monotonic** — its sign pattern is opposite the rest of the bank, which is why uniqueness is so high. |

## F1 — `trending_swings` is the standout independent signal

**Numbers.** On the per-bar bank, mean |r| with the other 11
contributors:

```
trending_swings        |r|_avg = 0.335   ← standout, by 2× margin
spike_duration         |r|_avg = 0.770
always_in              |r|_avg = 0.775
session_shape          |r|_avg = 0.776
majority_trend_bars    |r|_avg = 0.780
day_type               |r|_avg = 0.811
htf_alignment          |r|_avg = 0.831
trending_everything    |r|_avg = 0.836
spike_quality          |r|_avg = 0.848
cycle_phase            |r|_avg = 0.870
small_pullback_trend   |r|_avg = 0.876
bpa_trend_bar_density  |r|_avg = 0.879
```

**Why.** `trending_swings` is the only direction-voter that requires
detectable swing structure (HH/HL or LH/LL). Strictly monotonic
sessions have no detectable swings, so on `bull_trend` /
`bear_trend` it scores 0 — the *opposite* of what every other
direction voter does on those fixtures. On `bull_w_pullbacks` /
`bear_w_pullbacks`, the alternating bars create real swings, so
`trending_swings` fires while four other contributors go silent
(the structural blind-spot trio plus `day_type`). The sign pattern
is the negative of most contributors, which deflates its mean |r|.

**Implication.** In a future weighting scheme, `trending_swings` is
the contributor most worth keeping at full weight. Removing it would
lose information that no other contributor recovers. The 1-bar
swing-confirmation lag (spec rule #2) is the cost of carrying the
only contributor that uniquely lights up pullback sessions — worth
paying.

## F2 — `small_pullback_trend` ↔ `bpa_trend_bar_density` are near-duplicate votes

**Numbers.** Top 5 most-correlated pairs on the bank:

```
small_pullback_trend  ↔ bpa_trend_bar_density   r = +0.997  (same family)
trending_everything   ↔ htf_alignment           r = +0.995  (cross-family — see F4)
always_in             ↔ spike_duration          r = +0.986  (cross-family — see F4)
always_in             ↔ majority_trend_bars     r = +0.979  (cross-family — see F4)
small_pullback_trend  ↔ spike_quality           r = +0.977  (cross-family — see F4)
```

**Why this one is real.** `small_pullback_trend` and
`bpa_trend_bar_density` both belong to the magnitude family. Both
emit a smooth signed magnitude based on per-bar trend-bar character.
They use different *internal* mechanics (5-sub-check weighted sum vs
per-bar Brooks predicate density), but the outputs track each other
across all 5 fixtures: the per-bar trend-bar density mostly
determines what each one says.

**Why this one isn't a sample-bank artifact.** Both contributors have
genuine bar-by-bar variability within fixtures (unlike `htf_alignment`
or `spike_duration` which saturate). The 0.997 correlation reflects
shared mechanics, not shared between-fixture polarity.

**Implication.** Strong candidate to either:

- **Down-weight as a unit**: weight = 0.5 each in the magnitude family.
- **Drop one and reduce the magnitude family from 5 → 4 contributors.**
  `bpa_trend_bar_density` is the simpler, more interpretable read
  (per-bar Brooks predicate, no 5-sub-check). `small_pullback_trend`
  carries the broken-swing gate, which is information neither
  `bpa_trend_bar_density` nor any other contributor has.

**Defer until Pattern Lab WRs.** Don't make this call from the
correlation alone — measure which one (or which combination)
correlates better with backtest outcomes.

## F3 — equal weighting is double-counting most of the stack

**The shape of the distribution.** 11 of 12 contributors sit between
|r|_avg = 0.77 and 0.88. Only `trending_swings` is significantly
below. Bank average = 0.79. That means the typical contributor is
~80% redundant with the rest of the stack on this fixture bank.

**What this means for the aggregator.** Equal weighting silently
treats two near-identical votes (e.g. F2's near-duplicate pair) the
same as two genuinely independent votes (e.g. `trending_swings` vs
anyone else). The net effect on the equal-weighted mean is
overweighting the "consensus" view.

**What this DOESN'T mean.** It doesn't mean equal weighting is wrong
in production. Two reasons:

1. The bank is synthetic and polarized. Real intraday sessions are
   messier; redundancies will be lower.
2. **Redundancy ≠ harmful**. Multiple correlated reads of the same
   underlying signal can be a *robustness* feature: any one
   contributor failing (a NaN, a regression) doesn't tank the
   aggregate. Equal weighting buys robustness at the cost of vote
   efficiency.

**Implication for the deferred weighting question.** The choice is
not "equal vs unequal" — it's "what specifically do we want from the
aggregate?" If we want robustness, equal weighting is justified. If
we want efficiency (treating one independent contributor as worth
more than two duplicates), down-weight the redundant pair from F2.
Pattern Lab WRs answer this empirically.

## F4 — sample-bank artifacts: cross-family near-perfect correlations

Three of the top-5 most-correlated pairs are cross-family:
`trending_everything` ↔ `htf_alignment`, `always_in` ↔ `spike_duration`,
`always_in` ↔ `majority_trend_bars`. None of these reflect shared
contributor mechanics — they're driven by the bank's structure.

**The mechanism.** Inside a single fixture (say `bull_trend`):

- `htf_alignment` is constant (HTF history doesn't change bar-to-bar).
- `trending_everything` saturates at +1.0 within ~5–10 bars on a
  monotonic session and stays there.
- `spike_duration` saturates at the end of the opening sequence.
- `always_in` saturates at +1.0 once the 5-bar streak hits.

Across fixtures the signs are also confluent (bull intraday → bull
HTF, bull → all magnitude scorers fire positive, etc.). So the
between-fixture variance — bull positive, bear negative, choppy zero
— dominates the Pearson denominator. The within-fixture variance is
near zero. Result: r ≈ 1 even when the contributors are
mechanically unrelated.

**The honest read.** These cross-family correlations would not
survive a real-data redundancy pass. The fixture bank was designed
to make every contributor *visible* (which is why HTF is paired
confluently per fixture), not to differentiate them. A production
redundancy measure should run on a backtest's worth of real intraday
sessions where directional polarity is messier and HTF varies
independently of intraday.

**The mistake to avoid.** Don't drop `htf_alignment` because it
correlates 0.995 with `trending_everything` here — it's the wrong
read of a measurement artifact. The correct argument for HTF lives
in the asymmetric-magnitude design (Finding 4 of the capstone), not
in this matrix.

## F5 — `trending_swings` and the structural blind-spot story

The blind-spot table from the capstone:

| Fixture | Firing | Blind contributors |
|---|---|---|
| `bull_trend` | 11/12 | `trending_swings` |
| `bull_w_pullbacks` | 8/12 | `always_in`, `majority_trend_bars`, `spike_duration`, `day_type` |
| `bear_w_pullbacks` | 8/12 | (mirror) |
| `bear_trend` | 11/12 | `trending_swings` |

Read together with F1, this is a coherent picture:

- `trending_swings` is **deliberately blind on monotonic sessions**
  (no detectable swings) but **uniquely lights up pullback sessions**
  (alternating bars create swings).
- The four pullback-blind contributors (the strict-threshold ones)
  are deliberately blind on pullback sessions but light up on
  monotonic.

These two blind-spot patterns are **mirror images**, and the union
covers both regimes. The reason `trending_swings` has the lowest
|r|_avg in the bank is exactly that its sign pattern is the negative
of most of the stack.

**This is a feature, not a bug.** Removing `trending_swings` because
it "doesn't agree" with the consensus would erase the only contributor
that uniquely fires on pullback sessions — exactly the regime where
the strict-threshold contributors silently fail. Keep it at full
weight.

## Mistakes to avoid (forward-looking)

These are the ways this study could be misread, surfaced now so a
future pass doesn't fall into them.

1. **Don't treat Pearson r ≈ 1 on the synthetic bank as proof of
   production redundancy.** F4 explains why several cross-family
   pairs hit r ≈ 1 from bank structure alone. Real-data
   corroboration is required before any weight change. The within-
   family pair from F2 is more credible because both contributors
   have bar-by-bar variability — but even there, Pattern Lab WRs are
   the deciding evidence, not r.

2. **Don't down-weight high-|r|_avg contributors hoping to fix the
   reversal-lag drag.** That drag (capstone Finding 2 — equal-
   weighted mean settles at -0.13 vs recency mean -0.55 after a
   bull→bear flip) is caused by session-memory and regime
   contributors not flipping with recency. It's not caused by
   correlation. Different problem; the right remedy is family-level
   weighting that fades memory + regime when recency disagrees.

3. **Don't add more contributors to "fix" redundancy.** The
   inventory is closed at 12. The right move when an aggregate
   over-counts the consensus is consolidation (down-weight pairs)
   or specialization (different weights per family), not dilution.

4. **Don't drop a high-|r|_avg contributor without a redundancy
   partner.** `bpa_trend_bar_density` has |r|_avg = 0.879 (highest
   in the bank), but its near-duplicate is specifically
   `small_pullback_trend`. Dropping `bpa_trend_bar_density` only
   removes the duplicate, not the magnitude family's representation.
   Dropping it without also recognising the duplicate would leave
   the magnitude family with 4 contributors all bunched at high
   |r|_avg — a worse trade than down-weighting both.

5. **Don't claim this study answers the weighting question.** It
   surfaces candidates and ranks contributors by uniqueness. The
   weighting decision is empirical — Pattern Lab WRs against weight
   variants. Open spec question #1 is still open; this pass is the
   foundation, not the answer.

## What's next (still NOT taken autonomously)

Same three items as the capstone, plus one new one this study
specifically informs:

1. **Emit `trend_state` into the dashboard payload.** Additive but
   visible. Needs Will's nod.
2. **Wire real HTF history in the pipeline** so `htf_alignment` isn't
   dormant in production.
3. **Pattern Lab WR-driven weighting decision.** Open spec question
   #1.
4. **NEW — real-data redundancy validation.** Run the same
   pairwise-Pearson study on a backtest sample of real intraday
   sessions (the per-day urgency-gated backtest log already produces
   the necessary input). Will surface which of the bank-level
   findings (F1, F2 especially) hold on real data, and rule out the
   sample-bank artifacts (F4). Single deliverable: a new
   `contributor_agreement_real.png` to sit beside the synthetic
   one. Useful regardless of whether weighting changes; informs
   future contributor decisions.

## Where things live

- **The new figure**: `vault/Scanner/methodology/figures/contributor_agreement.png`
- **The function**: `tools/visualize_trend_contributors.py::fig10_contributor_agreement`
- **This note**: `vault/Scanner/methodology/trend-contributor-findings-2026-04-19-incr16-redundancy.md`
- **The capstone (read first)**: `vault/Scanner/methodology/trend-contributor-findings-2026-04-19-incr15-capstone.md`
- **Regenerate**: `python3 tools/visualize_trend_contributors.py` from the scanner repo.

## Authority

Autonomous under `~/code/aiedge/scanner/CLAUDE.md` "refactors and
internal carves are fine." This pass is **read-only** with respect
to the scanner: no production code modified, no thresholds touched,
no flags flipped, no payload change. The aggregator, schema, and
test suite are byte-stable. The only artifacts produced are research
deliverables in the vault.
