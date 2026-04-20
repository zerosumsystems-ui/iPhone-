---
date: 2026-04-19
increment: 24
topic: Structure flip-type breakdown — intended evolution vs genuine reversal
status: finding (NO production code change)
---

# Incr 24 — What kind of flips are the 1,905 structure flips?

## TL;DR

Incr 23 said direction flips are rare (0.58/session) but structure flips are
**16× noisier** (9.53/session) and speculated that "most are the intended
Brooks spike → channel evolution." **That speculation was wrong.** Re-reading
the 9,425-row trajectory and classifying every flip:

- **Only 8 %** of structure flips are `intended_evolution` (spike → channel,
  same direction). The end-of-session distribution (94 % in channels at bar
  78) looks tidy because sessions _eventually_ land in a channel — not
  because the path there was smooth.
- **30 %** are `cross_reversal` — the structure label literally crossed the
  bull/bear boundary (e.g. `bull_channel → bear_spike`).
- **30 %** are `consolidation` (any direction → `trading_range`) and
  **30 %** are `resumption` (`trading_range` → any direction). Sessions
  enter and leave ranges at roughly equal rates.
- Despite cross_reversal being common (median **3/session**, p95 **7/session**),
  the Pearson correlation with *direction* flips is only **r = 0.144**.
  **Aggregate direction damping does its job** — structure can flicker across
  the bull/bear boundary without the 12-contributor mean crossing the ±0.05
  direction threshold.

Practical implication: the front-end `TrendState` panel (if/when it lands)
can treat **direction** as a trustworthy early read, but must treat
**structure** as a discrete label with no self-consistency guarantee. Don't
tie latching / colour-change behaviour to structure.

## Method (pure read-only analysis)

- Source: `realtime_stability_stratified_incr23_traj.csv` (9,425 bar-by-bar
  rows across 200 real RTH sessions × up to 68 progressive calls each,
  warmup k=10).
- Per session, walked the k-ordered bars and emitted a transition row every
  time `structure[k] != structure[k-1]`.
- Classified every transition into exactly one of 6 mutually-exclusive
  buckets:
    1. `intended_evolution` — `bull_spike → bull_channel` or
       `bear_spike → bear_channel`. The Brooks canonical progression.
    2. `reverse_evolution` — `bull_channel → bull_spike` or
       `bear_channel → bear_spike`. A re-energising within the same side.
    3. `consolidation` — `bull_* → trading_range` or `bear_* → trading_range`.
    4. `resumption` — `trading_range → bull_*` or `trading_range → bear_*`.
    5. `cross_reversal` — `bull_* → bear_*` or `bear_* → bull_*`.
       The informative kind.
    6. `other` — safety net, 0 rows observed.
- Aggregator and schema untouched. Test suite untouched (602 / 905).

## The numbers

```
category              count    pct
intended_evolution      158   8.29 %
reverse_evolution        28   1.47 %
consolidation           582  30.55 %
resumption              562  29.50 %
cross_reversal          575  30.18 %
other                     0   0.00 %
total                 1,905
```

### Timing profile (flips per 10-bar bin)

Bins map to within-session bar index; warmup floor k=10, RTH length 78 bars.

| category             | 10–19 | 20–29 | 30–39 | 40–49 | 50–59 | 60–78 |
|----------------------|------:|------:|------:|------:|------:|------:|
| intended_evolution   | **126** | 7 | 12 | 7 | 2 | 4 |
| reverse_evolution    | 16 | 4 | 4 | 2 | 0 | 2 |
| consolidation        | 185 | 103 | 106 | 21 | 46 | 121 |
| resumption           | 167 | 101 | 113 | 24 | 47 | 110 |
| cross_reversal       | 271 | 81 | 76 | 17 | 36 | 94 |

Two sharp observations:

1. **`intended_evolution` is front-loaded**: 80 % (126/158) occur in bars
   10-19 — which is exactly when the opening spike ends. This is the one
   well-behaved category.
2. **`cross_reversal` is bimodal**: heavy at the open (271 in bars 10-19)
   AND heavy at end-of-day (94 in bars 60-78). The middle of the session is
   quieter. Matches the incr 23 finding that direction flips are bimodal in
   bars 15-39; the end-of-day spike here is **new** — direction flips don't
   have it, because by bar 60 aggregate strength usually has enough
   commitment to absorb late-session structure chop.

### Cross-reversal vs direction flips, per session

- Pearson **r = 0.144** (n = 200).
- Median cross_reversals / session = **3**. Median direction flips / session
  = **0** (from incr 23).
- The mismatch is the point: roughly 5 cross_reversal structure events per
  every 1 direction flip. The aggregator absorbs 80 % of them.

## Why this matters

Two places this changes how we should think about live output:

1. **Latching behaviour** — if the front-end ever emits a latched "trend
   confirmed" ribbon, it must latch on **direction** (rare flips, known
   cadence) not **structure** (noisy regardless of direction stability).
2. **Narrative output** — when we tell the trader "the session structure
   is `bull_channel`", we should emit it as an instantaneous label with a
   small confidence-of-current-label indicator (how long has structure held
   this label?), not as a property with stability semantics.

This also retroactively corrects incr 23's prose. The claim was "The flips
are the intended Brooks spike→channel evolution, not random flicker." The
correct claim is: **the end-of-day distribution is tidy (94 % in channels at
bar 78) but the path there is not** — sessions visit ranges and cross the
bull/bear structure line a median of 3 times on the way.

## Five mistakes to avoid — explicit, so future-me doesn't repeat

1. **Don't infer flip mechanics from distribution snapshots.** "94 % in a
   channel at bar 78" ≠ "the path was spike → channel". Always re-check
   transitions when asking about flips.
2. **Don't treat structure as a stability signal.** Direction is the
   stability signal (0.58 flips/session, deterministic bimodal timing).
   Structure is a descriptor, not a stability property.
3. **Don't conflate structure cross-reversals with direction flips.** They
   correlate at r = 0.14. The aggregator eats most of the structure
   noise — a structure cross-reversal without a direction flip is the
   expected common case, not a bug.
4. **Don't overweight the 60-78 bar cross-reversal spike** in any real-time
   rule. It's end-of-day chop; the more useful gate is the incr 23 rule
   ("`|strength| ≥ 0.15` at bar 20 → 93 % direction survival").
5. **Don't re-run the progressive-k sweep to iterate on this analysis.**
   The 9,425-row trajectory CSV is the durable artifact. All flip-type /
   transition questions should be answered off it.

## What's next — needs Will's nod

No code change proposed by this pass. The analysis only sharpens what we
already knew directionally. Open items unchanged from incr 23's list:

- Emit `trend_state` into dashboard payload (partially landed in incr 17).
- Pattern Lab WR-driven weighting decision.
- Optional front-end rule: "show high-confidence live direction when
  `|strength| ≥ 0.15` AND bar ≥ 20" (incr 23). This pass reinforces that
  rule — direction is the right thing to gate on, not structure.

## Artifacts

- `structure_flip_breakdown_incr24.csv` — per-flip rows.
- `structure_flip_category_counts_incr24.csv` — 6-row category rollup.
- `structure_transition_matrix_incr24.csv` — 5×5 from→to counts.
- `structure_flip_timing_incr24.csv` — category × bar-bin timing.
- `structure_flip_per_session_incr24.csv` — per-session category counts
  joined to incr 23 direction-flip numbers.
- `structure_flip_types_incr24.json` — summary.
- Figures in `vault/Scanner/methodology/figures/`:
  `structure_flip_categories.png`, `structure_flip_timing.png`,
  `structure_transition_matrix.png`, `structure_flip_vs_direction_flip.png`.
- Tool: `~/code/aiedge/scanner/tools/structure_flip_types_incr24.py`.
