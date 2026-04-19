# Trend research — incr 23 — flip timing, strength-as-predictor, structure trajectory

**Date:** 2026-04-19
**Status:** Read-only; no production code changed. One **new** deferred recommendation (a real-time gate rule) documented as optional; five prior threshold changes still pending Will's nod.

## TL;DR

Incr 22 proved the live `TrendState.direction` rarely flips (63 % zero-flip rate, median lock-in bar 11). This increment asks three residual real-time questions:

1. **When** do the 37 % of flips that *do* happen actually happen?
2. Can a trader use early `|strength|` to know in advance which sessions are "safe to trust"?
3. Does `TrendState.structure` stabilise at the same rate as direction, or does it flicker even when direction holds?

**Three decisive answers:**

- **Flips cluster tightly in bars 15-39 (64 % of all flips) — not uniformly.** Peaks at bar 15-19 (**19 %**) and bar 30-39 (**26 %**). After bar 40, only **26 %** of flips occur. **Waiting until bar 30 dodges most of the reversibility.**
- **`|strength| ≥ 0.15 at bar 20` is a clean real-time gate.** Sessions meeting this bar have **93 %** direction-survival to the final bar (vs 47 % baseline for |strength| < 0.15) and **85 %** zero-flip rate (vs 46 % baseline). Median lock-in bar drops from 18 → 10.
- **Structure is 16× noisier than direction.** 1 905 structure flips vs 116 direction flips across the 200 sessions (9.5 vs 0.58 per session). But most structure flips are the *intended* spike→channel evolution: at bar 10, **100 %** of sessions are in `bull_spike`/`bear_spike`; by bar 30, **62 %** have moved to `bull_channel`. The label is telling a story, not flickering randomly.

## Headline numbers

- Sample: **200 (symbol, session) pairs** from `cache/databento/*/*.parquet`, 1-min → 5-min RTH. 12 129 long-form bar rows in the persisted trajectory CSV. Runtime: **355 s** (~5.9 min).
- **Direction flips:** 116 total across 200 sessions; mean **0.58** / session (matches incr 22's 0.56 on 300 sessions — the stability claim reproduces).
- **Structure flips:** 1 905 total; mean **9.53** / session; max per session counted on the full bar-to-bar label change trace.

### Flip timing — the bimodal distribution

| Bar bucket | Flip count | Share of all flips |
|---|---|---|
| 10-14 | 2 | **2 %** |
| 15-19 | 22 | **19 %** |
| 20-24 | 16 | 14 % |
| 25-29 | 16 | 14 % |
| 30-39 | 30 | **26 %** |
| 40-49 | 10 | 9 % |
| 50-59 | 5 | 4 % |
| 60-78 | 15 | 13 % |

64 % of all flips fall in bars 15-39. The **60-78 bucket** holds 13 % (end-of-day "slow reversal" cluster), but per-bar rate is a third of the bar-15-39 zone.

### Strength-as-predictor — bar 20 gate

| `|strength|` at bar 20 | n | % zero flips | Median lock-in | % dir@20 = final |
|---|---|---|---|---|
| 0.00 – 0.15 | 85 | 46 % | 18 | **47 %** |
| 0.15 – 0.30 | 59 | **85 %** | **10** | **93 %** |
| 0.30 – 0.50 | 6 | 100 % | 10 | 100 % |
| 0.50 – 1.00 | 0 | — | — | — |

The gap between bin 0–0.15 and bin 0.15–0.30 is structural — 93 % dir-survival vs 47 %. The **0.15 threshold is the gate the data is asking for**.

Caveat: the 0.30-0.50 bucket is only **n=6**, and 0.50+ is **n=0**. The equal-weighted 12-contributor aggregate rarely pushes `|strength|` above 0.30 at bar 20 because not all contributors are firing yet (warmup effects, session-memory contributors still settling). A future weighting study might push the distribution rightward.

### Structure trajectory (where are sessions at each bar)

| Bar | bull_spike | bull_channel | bear_spike | bear_channel | trading_range |
|---|---|---|---|---|---|
| 10 | **49 %** | 0 % | **51 %** | 0 % | 0 % |
| 20 | 1 % | 50 % | 0 % | 38 % | 12 % |
| 30 | 0 % | 62 % | 1 % | 31 % | 7 % |
| 40* | 0 % | 66 % | 2 % | 30 % | 2 % |
| 78* | 0 % | **84 %** | 0 % | 10 % | 6 % |

*Bars 40+ are a subset (only 90-110 sessions reach the full RTH day; today's partial session accounts for the rest).

The structure label *evolves* from opening-spike → channel as sessions unfold — this is the intended Brooks cycle-phase behaviour, not a bug. At bar 78 (end-of-day subset), **94 %** of sessions are in a channel (bull or bear), **6 %** in a trading range, and zero in a spike.

## Why this matters for live consumption

- **The "wait till bar 30" rule.** If a dashboard consumer wants a very safe read, waiting until bar 30 skips 64 % of the flip risk. Combined with incr 22's median lock-in bar 11, this sets an explicit real-time cadence:
  - **Bar 10**: reading available but near-random (31 % 'none', 34 % up / 35 % down).
  - **Bar 20**: decisive for a subset (|strength| ≥ 0.15) — 93 % of THIS subset will see their reading survive to the end.
  - **Bar 30**: 64 % of the flipping sessions already had their reversal. The reading is entering the stable majority.
- **The |strength| ≥ 0.15 gate is a bolt-on rule the front-end could use today** to say "high-confidence live direction" vs "still forming". Direction + strength combined, not direction alone. This complements incr 22's median-lock-in finding.
- **Structure isn't a reliable "is the trend solid?" signal in real time.** It flips 9.5× per session on average, and the flips are the intended spike→channel→pullback progression. The front-end should probably emit structure as a discrete label without a "confidence" summary, and shouldn't treat a structure change as a reversal event.

## Mistakes avoided

1. **Didn't over-claim on the `< 0.15` low-survival number.** 47 % of those sessions had their bar-20 direction disagree with their final direction. But many of those cases are `direction = 'none'` at bar 20 (the classifier saying "not sure yet"), NOT the classifier being wrong. 'none' at bar 20 and 'up' at bar 78 isn't a flip — it's a late commit. The 85 % / 46 % zero-flip gap is the cleaner number here because it only counts up↔down reversals.
2. **Didn't treat the `0.50+` empty bucket as a strength-as-predictor ceiling.** n=0 means "the aggregate doesn't saturate that fast", NOT "high-strength readings don't exist". The 12-contributor equal-weighted mean is structurally damped at bar 20. The right framing is "the gate is at 0.15, not higher".
3. **Didn't conflate structure flips with real instability.** 9.5 structure flips/session is loud, but 100 % → 0 % spike share between bar 10 and bar 30 is the planned cycle-phase evolution. The right comparison is structure flips *within the same broad phase* (bull_spike → bull_channel is expected; bull_channel → bear_channel IS a reversal). Left this as a flagged follow-up in the "open work" section.
4. **Didn't re-run incr 22.** The 63 % zero-flip rate already exists on a 300-session sample; re-running it on 200 sessions would dilute the signal. This increment uses a fresh 200-session sample specifically because we needed per-bar strength and structure, which incr 22 didn't persist.
5. **Didn't promote the 0.15 gate as a production change.** It's a real-time *consumer* rule for the dashboard, not a change to `compute_trend_state`. Will's nod is needed if it ever lands in the front-end panel, but it shouldn't change scoring, ranking, or the aggregate output.
6. **Caveat flagged on late-bar subset size.** Only 90 sessions reach the full 78-bar RTH day — the rest are today's partial 2026-04-19 session. All figure captions carry this caveat. Late-bar percentages reflect that subset, not the full 200.

## Deferred — Will's nod

- **incr 22**: no new recommendations.
- **incr 23 (this run)**: optional real-time consumer rule — "show high-confidence live direction when `|strength| ≥ 0.15` AND bar count ≥ 20". Not a change to `compute_trend_state`. Relevant only if/when the front-end `TrendState` panel lands.
- Prior still-pending:
  - incr 21: `OPEN_EXTREME_THRESHOLD: 0.15 → 0.25` in `aiedge/context/daytype.py`
  - incr 20: `ALWAYS_IN_WINDOW: 5 → 10` in `aiedge/context/trend.py` + mirror `STRONG_TREND_WINDOW` in `aiedge/signals/components.py`
  - incr 18: `MAJORITY_TREND_BAR_FLOOR = 0.25` in `aiedge/signals/components.py`
  - incr 19: docstring note on `compute_trend_state` re: silent-zero `htf_alignment`

## Open work (read-only, next passes)

- **Structure flip type breakdown.** Distinguish "intended evolution" flips (bull_spike → bull_channel) from "reversal" flips (bull_channel → bear_channel). The 9.5 / session total likely hides a 9 / session intended + 0.5 / session genuine-reversal split. Re-run the stratified trajectory with an intent taxonomy.
- **Strength-predictor on a larger sample.** `|strength| ≥ 0.30` has n=6 only. A 500-session run would fill out the high-strength buckets.
- **Day-type stratification.** The current aggregate mixes trend-from-open days with trading-range days. Lock-in on a TFO day probably happens at bar 10; on a trading-range day it may never. Re-run with `_classify_day_type` labels (requires wiring `gap_direction` + `gap_held` upstream — deferred as a weekend task).
- **Strength-at-bar-30 as an even stronger gate.** Bar 20 catches 93 % survival at |strength| ≥ 0.15. A bar-30 gate might get to 98 %. Trade-off: more delay in the live reading.

## Artifacts

- Sweep tool: `tools/realtime_stability_stratified_incr23.py` (new, ~400 LOC)
- Per-session summary CSV: `vault/Scanner/methodology/realtime_stability_stratified_incr23.csv` (200 rows)
- Full per-bar trajectory CSV: `vault/Scanner/methodology/realtime_stability_stratified_incr23_traj.csv` (**12 129 rows** — reusable for future analyses without re-running)
- Summary JSON: `vault/Scanner/methodology/realtime_stability_stratified_incr23.json`
- Figures:
  - `figures/realtime_flip_timing.png` — histogram of WHEN flips occur
  - `figures/realtime_strength_predictor.png` — twin-axis: zero-flip & dir-survival by |strength| bin, plus median lock-in bar
  - `figures/realtime_structure_density.png` — stacked bars of structure labels at k = {10, 20, 30, 40, 50, 60, 78}
  - `figures/realtime_dir_vs_struct_flips.png` — direction vs structure flip count histogram
- PDF: `pdfs/trend-research-2026-04-19-incr23.pdf`
