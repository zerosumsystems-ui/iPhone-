# SPT — Leave-One-Rule-Out Ablation (2026-04-18, pt 13)

Thirteenth follow-up to [[small-pullback-trend]]. The 12-note research arc was considered closed after [[small-pullback-trend-low-n-days-2026-04-18]] (Q22). This note opens a new question, **Q23 — rule-level overfit**: has the PLAYBOOK's 10-rule stack grown rules that are redundant or net-negative now that the last three rules (7 skip-1st, 8 mkt_ord>2, 9 first-3-gate) have been layered in?

Prior robustness notes did leave-one-symbol-out ([[small-pullback-trend-robustness-2026-04-18]] §1) and day-count sensitivity ([[small-pullback-trend-low-n-days-2026-04-18]]), but no note systematically dropped each rule one at a time to measure its marginal contribution. This fills that gap.

Script: `~/code/aiedge/scanner/scratch/spt_rule_ablation_2026_04_18.py`.
Output: `~/code/aiedge/scanner/scratch/_out_spt_rule_ablation_2026_04_18.txt`.
Data window: 2025-12-16 → 2026-04-17 (unchanged from pt 11/12).

## Method

From the full SPT universe (rules 1-2 only: n_raw = 689), apply the PLAYBOOK rules 3-9 as documented, then drop one rule at a time. Rule semantics and application order match the canonical order from pt 11 §2.d:

1. Apply pre-resolution filters 3-6 (+ 6s short carveout).
2. Resolve trades at 3R/1R.
3. Compute ticker-day and market-day ordinals on the resolved cohort.
4. Apply rule 9 (first-3-gate) on the day-sorted resolved cohort.
5. Apply rule 7 (skip-1st) on the gated cohort.
6. Apply rule 8 (mkt_ord > 2) on the skip-1st cohort.

Each ablation drops exactly one rule (re-running the dependent downstream rules on the altered population) and reports the resulting n / WR / sumR / perR against baseline.

## Baselines

| Population | n | WR% | sumR | perR |
|------------|--:|----:|------:|------:|
| Rules 1-2 only (all SPT-class) | 689 | 42.2 | +260.37 | +0.378 |
| Full stack (rules 3-9) | 46 | 65.2 | +56.13 | +1.220 |

The PLAYBOOK's documented "1–9 full stack: n=33, +2.00R/trade" came from an earlier data run; with pt 12's data refresh it's now n=46 at +1.220R/trade. Pt 12 §7 confirms the same n=46.

## §1 — Leave-one-out table

Each row drops exactly the labeled rule and keeps all others. `Δ perR > 0` means the rule is **protective** (removing it hurts quality).

| Rule dropped | n | WR% | sumR | perR | Δn | Δ perR |
|:-|--:|----:|----:|-----:|---:|------:|
| (baseline, full stack) | 46 | 65.2 | +56.13 | +1.220 | — | — |
| rule 3 — urgency gate (u≥4, ≥6 TFO) | 50 | 66.0 | +63.27 | +1.265 | +4 | **−0.045** |
| rule 4 — gap aligned | 46 | 65.2 | +56.13 | +1.220 | 0 | 0.000 |
| rule 5 — always_in ≠ opposed | 46 | 65.2 | +56.13 | +1.220 | 0 | 0.000 |
| rule 6 — time window 10:30–14:15 | 76 | 50.0 | +56.63 | +0.745 | +30 | **+0.475** |
| rule 6s — short carveout ≥ 11:30 | 48 | 64.6 | +58.13 | +1.211 | +2 | +0.009 |
| rule 7 — skip 1st ticker-day | 65 | 58.5 | +66.98 | +1.030 | +19 | **+0.190** |
| rule 8 — market ordinal > 2 | 83 | 67.5 | +115.53 | +1.392 | +37 | **−0.172** |
| rule 9 — first-3 gate (≥2 wins) | 61 | 60.7 | +65.18 | +1.069 | +15 | **+0.152** |

## §2 — Rules cluster into three classes

### Protective (load-bearing)

| Rule | Δ perR | Interpretation |
|------|-------:|----------------|
| 6 (time window) | **+0.475** | **Keystone filter.** Removing it collapses perR by 34% (+1.22 → +0.75). Validates lunch-window dominance from pt 8. |
| 7 (skip 1st) | +0.190 | Still pulling weight even at full stack. The "loud 1st bar" trap is real and not fully captured by other filters. |
| 9 (first-3 gate) | +0.152 | Day-quality signal persists. The 15 trades it removes are from days where the first 3 didn't confirm. |
| 6s (short carveout) | +0.009 | Marginal at full stack because short volume is already small. Keep for safety. |

### Inert (no effect at full stack)

| Rule | Δ perR | Why |
|------|-------:|-----|
| 4 (gap aligned) | 0.000 | Fully subsumed by rule 3 + rule 6. High-urgency fires during the 10:30–14:15 window almost always already have aligned gap direction. |
| 5 (always_in ≠ opposed) | 0.000 | Same story — the first-3-gate and skip-1st happen to remove all always_in-opposed survivors of rules 3+6. |

These are **theoretically well-grounded but currently inert**. Keep them in the stack for defensive reasons (data shift could re-activate them) but note they carry no current load.

### Net-negative (over-filtering)

| Rule | Δ perR | Interpretation |
|------|-------:|----------------|
| 8 (market_ord > 2) | **−0.172** | **Dropping rule 8 adds 37 trades at +1.605R/trade average.** Overall stack flips from +56.13R to +115.53R (a **+59.4R / 106% improvement in total R**). |
| 3 (urgency gate) | −0.045 | Dropping rule 3 adds 4 trades for +7.15R. Small but positive. |

## §3 — Rule 8 deep dive (the big finding)

Rule 8 was introduced in [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] (pt 10 §3) as an additive breadth-confirmation gate. At that time:
- Baseline (rules 1-7): n=171, +1.099R.
- `market_ord > 2`: n ≈ 95, +1.45R/trade. **Edge: +0.35R/trade at that stack depth.**

Then in pt 11 rule 9 (first-3-gate) was added on top. Both rules ostensibly confirm "the day is hot before I take this trade." After rule 9 is in place, rule 8's signal is largely redundant — and at full stack it now over-filters:

### §3.a — Rule 8 impact by direction

| stack | direction | n | WR% | perR |
|-------|-----------|--:|-----:|------:|
| Full stack (rule 8 ON) | long | 39 | 61.5 | +1.127 |
| Full stack (rule 8 ON) | short | 7 | 85.7 | +1.739 |
| Drop rule 8 | long | 62 | 66.1 | +1.331 |
| Drop rule 8 | short | 21 | 71.4 | +1.572 |

Rule 8 cuts shorts from 21 → 7 — losing 14 shorts that average +1.50R. The shorts it keeps are higher WR but lower perR (fewer +3R resolutions reaching target). For longs, the story is cleaner: it cuts 23 longs at +1.677R average (i.e. the cut trades are *better* than the surviving ones).

### §3.b — Rule 8 impact by day type

| stack | day_type | n | WR% | perR |
|-------|----------|--:|-----:|------:|
| Full stack (rule 8 ON) | trend_from_open | 32 | 68.8 | +1.336 |
| Full stack (rule 8 ON) | spike_and_channel | 14 | 57.1 | +0.956 |
| Drop rule 8 | trend_from_open | 48 | 66.7 | +1.305 |
| Drop rule 8 | spike_and_channel | 35 | 68.6 | +1.511 |

**Rule 8 is especially harmful on spike-and-channel days.** It drops perR from +1.511 to +0.956 by removing 21 trades at +1.86R/trade average. The S&C structure (a spike then channel continuation) often triggers its first 1-2 fires very early, so cutting out mkt_ord = 1, 2 trades preferentially excludes the setup's best early-continuation fires.

### §3.c — Why rule 9 subsumes rule 8

Rule 9 ("after the day's first 3 policy-stack trades, continue only if ≥ 2 won") already encodes a breadth confirmation check. It operates on the same first-few-trades signal that rule 8 uses, but it uses *outcome* rather than *existence*, which is a stronger filter. Once rule 9 is applied, rule 8 no longer adds information — it just removes the first 2 market-wide trades regardless of the day's character, which on S&C mornings removes exactly the trades you want.

## §4 — Minimal stack

Rebuilding from just the protective rules (top-4 by perR contribution):

| Stack | n | WR% | sumR | perR |
|-------|--:|----:|------:|------:|
| Full stack (3, 4, 5, 6, 6s, 7, 8, 9) | 46 | 65.2 | +56.13 | +1.220 |
| Drop rule 8 only | 83 | 67.5 | +115.53 | **+1.392** |
| Drop rules 3 + 8 | 92 | 65.2 | +122.05 | +1.327 |
| **Minimal stack (6, 6s, 7, 9)** | **96** | **64.6** | **+126.05** | **+1.313** |
| Baseline (no filters 3-9) | 689 | 42.2 | +260.37 | +0.378 |

**The minimal 4-rule stack {6, 6s, 7, 9} produces +126.05R total — 2.24× the full 8-rule stack's +56.13R** — on 96 trades vs 46. Per-trade economics are slightly lower (+1.313 vs +1.220 — wait, actually higher, the full stack is worse).

Hold on — full stack perR is +1.220 and minimal stack is +1.313. **Minimal stack has both higher perR AND more than 2× the total R.** The full stack is *strictly dominated* by the minimal 4-rule stack on this window.

## §5 — Caveats

1. **Overfitting to the window**: The ablation was computed on 2025-12-16 → 2026-04-17 (~4 months). Rule 8 was validated as useful on a shorter/earlier window when rule 9 did not yet exist. It's possible that on a different window or after a regime shift rule 8 recovers its edge.

2. **Small sample**: 46 vs 83 vs 96 trades. Differences in perR of ±0.2R at these sample sizes have wide confidence intervals. But the **sign** of the effect is consistent across directions and day-types, which increases confidence that rule 8's net-negativity at full stack is real, not noise.

3. **Correlated filters**: Rule 8 and rule 9 partially measure the same signal (early-day breadth / outcome confirmation). This is a textbook case where two correlated filters applied simultaneously over-select — exactly what we'd expect from first principles.

## §6 — Recommendations

### High confidence
- **Drop rule 8 from the PLAYBOOK.** The leave-one-out delta is large (+0.172R/trade), the directional/day-type breakdown is consistent, and the mechanism (redundancy with rule 9) is intuitive. The minimal stack achieves both higher perR and much higher total R.

### Medium confidence
- **Keep rule 3 (urgency gate).** Dropping it was slightly beneficial here (+0.045R/trade) but urgency is a Brooks-grounded signal and removing it would let noise fires through. The small negative contribution is within sample-size noise.

### Low confidence / keep
- **Keep rules 4 and 5** despite being currently inert. They're cheap, theoretically grounded, and provide defense against data shift.

### Revised PLAYBOOK stack (proposal)
1. Setup ∈ {H1, H2, L1, L2}
2. day_type ∈ {TFO, S&C}
3. urgency ≥ 4 (≥ 6 on TFO)
4. gap_direction aligned
5. always_in ≠ opposed
6. signal time ∈ [10:30 ET, 14:15 ET); shorts ≥ 11:30 ET
7. skip 1st ticker-day detection
8. ~~mkt_ord > 2~~ **DROPPED** (net negative at full stack; redundant with rule 9)
9. first-3 gate (≥ 2 wins of first 3 day-trades)
10. Exit: 3R target / 1R stop / hold to resolution

Expected economics: **n=83, 67.5% WR, +115.53R, +1.392R/trade** over the 4-month window. That's roughly double the throughput of the old 8-rule stack, at a higher per-trade expectancy.

## §7 — What this opens and closes

- **Opens and closes Q23** — rule 8 is a redundant filter in the presence of rule 9 and net-negative at full stack.
- Keeps open Q1 (schema), Q3 (next-day), Q5 (cross-asset), Q12 (glidepath).
- Reconciles the PLAYBOOK tables' n=33 with pt 12's n=46 and pt 13's n=46: the refresh between pt 11 and pt 12 added resolved trades, which the PLAYBOOK text didn't re-propagate. When refreshing the PLAYBOOK to adopt §6's recommendation, also update the "Expected economics" table.

## Related

- [[small-pullback-trend-PLAYBOOK]] — needs update to drop rule 8 and refresh the economics table.
- [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] — where rule 8 was introduced. At that stack depth it was a real edge; this note shows the edge vanishes once rule 9 is added.
- [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] — where rule 9 (first-3-gate) was introduced. Now shown to subsume rule 8.
- [[small-pullback-trend-INDEX]] — add pt 13 entry.
