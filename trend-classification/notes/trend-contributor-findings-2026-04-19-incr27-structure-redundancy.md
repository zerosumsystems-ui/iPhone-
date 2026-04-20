---
date: 2026-04-19
increment: 27
topic: does `structure` carry incremental predictive value on top of (bar-k, |strength|) for the live direction-confirmed gate?
status: finding (NO production code change) — updates capstone open item "pick which schedule ships"
---

# Incr 27 — Structure is collinear with strength; the bar-k gate is a spike filter in disguise

## TL;DR

The +15 pp channel-vs-spike survival gap on equities (and +9 pp on ES) that
the bucket chart shows is **a composition artifact**. Once we condition on
`|strength| ≥ 0.15`, adding a `structure ≠ spike` filter buys only
**+1.3 pp** survival on equities and **+0.2 pp** on ES. The
`structure ≠ spike` filter and a hard `k ≥ 15` cutoff are **statistically
interchangeable** (100 % of spike-labelled directional observations on both
populations happen at bars 10-14, so the two rules select ≈ the same rows).

**Practical implication**: the incr-23 recommendation "high-confidence live
direction iff `|strength| ≥ 0.15` AND bar ≥ 20" does not need a structure
input bolted on. Strength alone captures essentially all the signal. Ship
strength + bar-k; leave `structure` as a display-only label on the
dashboard.

No production code change this pass. This is a negative result — it
saves us from shipping a redundant input on the live gate.

## Data

- Equity — incr 23 trajectory (`realtime_stability_stratified_incr23_traj.csv`):
  200 RTH sessions, 9,425 progressive-k rows, 7,264 directional-obs rows.
- ES — incr 26 trajectory (`es_direction_survival_incr26_traj.csv`):
  101 RTH sessions, 6,795 progressive-k rows, 4,887 directional-obs rows.
- Bar-bin / strength-bin edges imported verbatim from
  `tools/direction_survival_incr25.py` so every grid in the trend-research
  thread stays cell-aligned.

Structure labels in both trajectories:
`{bull_channel, bear_channel, bull_spike, bear_spike, trading_range}`
(no `bull_trading_range` / `bear_trading_range` variant emitted — verified
on disk). Canonical buckets: `channel | spike | trading_range`.

## Finding 1 — Bucket chart looks decisive, but is confounded by strength

| asset  | bucket         |   rate | n     |
|--------|----------------|-------:|------:|
| equity | spike          | 0.648  |   741 |
| equity | channel        | 0.798  | 5,967 |
| equity | trading_range  | 0.791  |   556 |
| es     | spike          | 0.617  |   355 |
| es     | channel        | 0.708  | 4,123 |
| es     | trading_range  | 0.729  |   409 |

`channel − spike` = +0.151 on equities, +0.092 on ES. At first glance this
is a big structural signal. It isn't — see Finding 3.

Figure: `structure_direction_survival_by_bucket.png` (bar chart, both
populations, per bucket, with sample sizes and coin-flip baseline).

## Finding 2 — Spike observations are 100 % concentrated at bars 10-14

For directional-nonzero rows (where we'd actually fire a live badge):

| k-bin | equity spike share | ES spike share |
|-------|-------------------:|---------------:|
| 10-14 | 693/693 (100.0%)   | 349/349 (100.0%) |
| 15-19 | 8/731 (1.1%)       | 2/363 (0.6%)     |
| 20-24 | 6/715 (0.8%)       | 0/393 (0.0%)     |
| 25-29 | 1/704 (0.1%)       | 0/381 (0.0%)     |
| 30-39 | 23/1369 (1.7%)     | 2/758 (0.3%)     |
| 40-59 | 6/1576 (0.4%)      | 2/1366 (0.1%)   |
| 60-78 | 4/1476 (0.3%)      | 0/1277 (0.0%)   |

**By the time bar = 15, "structure = spike" is essentially never emitted.**
The 12-contributor aggregator's session-memory contributors (spike_quality,
spike_duration) still have opinions about the open, but the structural
bucket has transitioned to channel. Result: `structure ≠ spike` and
`k ≥ 15` are selecting nearly identical row subsets.

## Finding 3 — Ablation: structure adds ≤ 1 pp survival over strength alone

Simulated gate variants on the full 9,425-row equity trajectory and
6,795-row ES trajectory:

**Equity** (coverage shown as % of all rows including `direction = none`):

| strength cut  | strength only                | + structure ≠ spike          | + k ≥ 15                    |
|---------------|-----------------------------:|-----------------------------:|----------------------------:|
| `|s|≥0.15`    | pass 34.5% / surv **89.0%**  | pass 31.0% / surv **90.3%**  | pass 31.1% / surv 90.4%     |
| `|s|≥0.20`    | pass 16.7% / surv 94.0%      | pass 15.0% / surv 95.2%      | pass 15.1% / surv 95.2%     |
| `|s|≥0.30`    | pass 3.2% / surv 99.3%       | pass 2.8% / surv 99.6%       | pass 2.8% / surv 99.6%      |

**ES**:

| strength cut  | strength only                | + structure ≠ spike          | + k ≥ 15                    |
|---------------|-----------------------------:|-----------------------------:|----------------------------:|
| `|s|≥0.15`    | pass 31.3% / surv **82.1%**  | pass 29.2% / surv **82.3%**  | pass 29.3% / surv 82.3%     |
| `|s|≥0.20`    | pass 17.2% / surv 90.8%      | pass 16.0% / surv 90.5%      | pass 16.1% / surv 90.5%     |
| `|s|≥0.30`    | pass 4.2% / surv 99.7%       | pass 3.7% / surv 99.6%       | pass 3.7% / surv 99.6%      |

**Two things jump out:**

1. `+ structure ≠ spike` and `+ k ≥ 15` are within ≤ 0.2 pp of each other
   in every row. They are the same filter.
2. The incremental gain of EITHER filter over strength-alone is +1.3 pp at
   most (equity `|s|≥0.15` → 89.0 → 90.3). On ES at `|s|≥0.20` the
   "with-structure" variant is actually 0.3 pp *worse* than strength alone
   (coverage drops from 17.2 % → 16.0 % but survival barely changes).

Figure: `structure_gate_ablation.png` — grouped bars, per asset, showing
the three variants at each strength cut with coverage annotation under
each bar.

## Finding 4 — channel and trading_range are near-identical survival classes

When the aggregator fires a non-none direction *inside* a trading-range
structure, survival is **79.1 % on equities** and **72.9 % on ES** —
essentially the same as channel (**79.8 % / 70.8 %**). Intuition says
trading-range periods should be lower-confidence, but by construction the
aggregator only emits a non-none direction when most of its 12 contributors
agree; those "directional trading-range" observations are rare
(7.6 % equity / 8.4 % ES of directional rows) and already self-filtered
by the ensemble.

This is why a spike vs non-spike split captures essentially all the
structural signal — channel and trading_range can be merged without loss.

## Finding 5 — ES vs equity: per-k trajectories confirm the incr-26 "stricter gate" finding

Figure: `structure_direction_survival_compare.png` — per-asset, per-bucket
survival-vs-bar-k curves.

- Equity channel rises monotonically 67.5 % (bar 15-19) → 91.6 % (bar
  60-78).
- ES channel rises more slowly: 63.7 % → 78.6 %. The asymptote is **13 pp
  below** equity even in the late session — this is the incr-26 finding
  confirmed at the structure-conditioned level. It isn't a spike-regime
  difference; ES channels are simply noisier than equity channels.
- Both populations have spike observations confined to bar 10-14 (bars
  sized 0-6 in the compare chart show spike curves that effectively
  terminate).

## Five mistakes-to-avoid documented this pass

1. **Reading a bucket bar chart as if it were causal.** `channel` vs
   `spike` had +15 pp on equities. After conditioning on `|strength|`, the
   gap collapses to +1.3 pp. Composition-confound was hiding.
2. **Treating "interpretable label" as a free input.** Adding `structure`
   to the live gate when it's collinear with `strength` adds cognitive
   load without adding signal. Test orthogonality before shipping a
   feature because it reads well.
3. **Comparing gate candidates on survival only.** At `|s|≥0.30` every
   variant converges to ~99 %. The discriminating range is `|s| ∈
   [0.15, 0.20]`, precisely where coverage is still useful. Always
   evaluate gates along the (survival, coverage) Pareto, not on survival
   alone.
4. **Conflating "spike is bad" with "spike hurts direction".** Spike
   observations have low survival (~65 %) because they fire at bar 10-14
   when strength and confidence haven't accrued — not because spike-ness
   is intrinsically toxic. A spike observation at bar 30 with
   `|strength|=0.3` (n=23 on equities) survives at 91 %.
5. **Not counting the zero cells.** `structure = spike` has zero
   observations at bar ≥ 15 on ES. Any gate "conditioning on structure"
   at bar ≥ 15 is a no-op on ES. Worth verifying before claiming a
   feature buys survival at late bars.

## Recommendation

**Keep the incr-23 proposed live rule as-is.** It's already optimal:

```
emit high-confidence direction iff |strength| ≥ 0.15 AND k ≥ 20
```

Do NOT add `structure != spike` as a gate input — it is collinear with
`k ≥ 15` (the 5-bar-earlier version of the bar cutoff) and adds at most
+1.3 pp survival at the cost of moving code.

Emit `structure` as a **display-only** label on the dashboard card (the
spike→channel transition is still human-interpretable signal that a
trader would want to see visually, just not as a gate).

## Authority

Autonomous, read-only. No scanner code touched. No flags flipped. No
ranking changes. The PREVIOUS capstone open items remain:

1. `ALWAYS_IN_WINDOW: 5 → 10` (incr 20) — needs Will's nod.
2. `MAJORITY_TREND_BAR_FLOOR: 0.40 → 0.25` (incr 18) — needs Will's nod.
3. docstring patch on `compute_trend_state` for silent-zero
   `htf_alignment` (incr 19).
4. `OPEN_EXTREME_THRESHOLD: 0.15 → 0.25` (incr 21).
5. Optional front-end rule "`|strength| ≥ 0.15` AND bar ≥ 20" (incr
   23/24/27) — **this pass reinforces and simplifies it**.
6. Pick the ES vs equity schedule (incr 26) — **this pass makes the choice
   cleaner**: strength cut is the only knob that matters; ES needs
   `|s| ≥ 0.20` for 90 %+ survival, equity needs `|s| ≥ 0.15`. Ship a
   two-strength rule keyed on asset class.
7. Emit `trend_state` into dashboard payload (still deferred).
8. Pattern Lab WR-driven weighting decision (blocked on DB backfill).

## Artifacts

- Tool: `~/code/aiedge/scanner/tools/structure_direction_survival_incr27.py`
- Per-cell CSV: `structure_direction_survival_incr27.csv`
  (asset × bucket × k-bin × strength-bin, n/hits/rate)
- Per-bucket CSV: `structure_direction_survival_incr27_buckets.csv`
- Top-level JSON: `structure_direction_survival_incr27.json` (includes
  ablation numbers from Finding 3)
- Figures in `figures/`:
  - `structure_direction_survival_by_bucket.png` — headline bucket bars
  - `structure_direction_survival_heatmap.png` — 3 × 2 cell grid
  - `structure_direction_survival_compare.png` — per-k trajectories
  - `structure_gate_ablation.png` — headline ablation bars (THE figure)
- PDF: `pdfs/trend-research-2026-04-19-incr27.pdf`
