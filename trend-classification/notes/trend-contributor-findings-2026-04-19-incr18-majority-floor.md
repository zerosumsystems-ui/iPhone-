---
title: Trend research — incr 18 · `majority_trend_bars` is gated by the 40 % majority floor, not the body-ratio threshold
date: 2026-04-19
slice: 800-session real-RTH-equity sweep across body-ratio threshold {0.30…0.60} and majority floor {0.20…0.50}; surfaces the floor as the binding constraint and quantifies directional accuracy when the classifier fires
---

# Increment 18 — `majority_trend_bars` floor recalibration study

## What this increment is

Incr 17 surfaced that `majority_trend_bars` was constant-0 across 593 cached
ES.c.0 overnight bars and recommended either dropping it or recalibrating
its threshold. This increment runs that recalibration study on a much
larger real-data sample — **800 RTH 5-min sessions across 387 US equity
symbols** drawn from the full `cache/databento/` parquet store — and
finds the right knob.

Two-stage sweep:

1. **Body-ratio sweep** — vary `MAJORITY_TREND_BAR_BODY_RATIO` ∈ {0.30 … 0.60}.
2. **Majority-floor sweep** — vary the 40 % "trend-bar majority" gate ∈ {0.20 … 0.50} while holding body-ratio at production (0.50).

Read-only with respect to the scanner. No production code modified, no
flags flipped, no thresholds touched. Sweep results are research
artifacts in the vault.

## TL;DR

- The body-ratio threshold is **not** the constraint. Even at body=0.30
  the classifier fires on only **1.9 %** of sessions.
- The **40 % majority floor** is the constraint. At the production
  floor of 0.40, `majority_trend_bars` fires in **1 of 800** real RTH
  equity sessions (0.12 %). On that single firing, the vote was wrong.
- When the floor is lowered to **0.25**, the classifier fires on 78 of
  800 sessions (**9.75 %**) with **~90 % directional accuracy** on the
  realised open → close move (vs a 66 % base-rate-of-up baseline).
- **Recommendation (needs Will's nod):** lower the majority floor from
  0.40 → **0.25**. Body-ratio threshold stays at 0.50.

## Sample

- 800 (symbol, session) pairs from `cache/databento/2026-04-09 … 2026-04-19`.
- 387 unique symbols; 9 trading dates.
- 45 437 5-min RTH bars total (median 56 bars per session).
- Realised direction (open → close, 1 bp deadband):
  **494 up · 255 down · 51 flat** — 66 % base rate for "always-predict-up"
  on the non-flat subset.

## Headline figures

### 1 — body-ratio threshold sweep (NOT the binding constraint)

![body-ratio threshold sensitivity](figures/majority_trend_bars_threshold_sensitivity.png)

| body-ratio | session fire rate | mean \|signed score\| |
|:---:|---:|---:|
| 0.30 | 1.9 % | 0.010 |
| 0.40 | 0.1 % | 0.001 |
| 0.50 (PROD) | **0.1 %** | **0.001** |
| 0.60 | 0.0 % | 0.000 |

Per-bar |body|/range distribution sits with median 0.558 and IQR
[0.388, 0.730]. **55 % of real bars** already have body > 0.50, so
"strong-body" bars are common. The classifier failing isn't because
real bars are weak.

![body-ratio distribution](figures/majority_trend_bars_body_ratio_distribution.png)

### 2 — majority-floor sweep (THIS is the lever)

![floor sweep](figures/majority_trend_bars_floor_sweep.png)

| floor | sessions fired | fire rate | directional accuracy |
|:---:|---:|---:|---:|
| 0.20 | 260 | 32.5 % | 87.6 % |
| 0.25 | 78 | 9.75 % | **90.5 %** |
| 0.30 | 10 | 1.25 % | 90.0 % |
| 0.35 | 1 | 0.12 % | 0.0 % |
| 0.40 (PROD) | **1** | **0.12 %** | **0.0 %** |
| 0.45 | 0 | 0.00 % | n/a |
| 0.50 | 0 | 0.00 % | n/a |

Decomposed by predicted direction (held-out base rate of "up" = 65.95 % on non-flat sessions):

| floor | up-vote n | up-pred acc | down-vote n | down-pred acc |
|:---:|---:|---:|---:|---:|
| 0.20 | 165 | 89.7 % | 84 | 83.3 % |
| 0.25 | 51 | 92.2 % | 23 | 87.0 % |
| 0.30 | 7 | 100.0 % | 3 | 66.7 % |

Both "up" and "down" votes outperform their respective base rates by **+15 to +25 pp**. The signal is real on both sides; not a directional-bias artifact.

### 3 — per-threshold session score buckets

![session scores](figures/majority_trend_bars_session_scores.png)

At every body-ratio threshold from 0.40 upward, **almost every session lands in the {-1, 0} buckets** (the failed-majority output). The "+0.5 / +1" buckets that would constitute a real directional vote are barely populated.

## Why the production floor produces dead silence

The score requires **>40 % of all session bars to be (a) in the trend
direction AND (b) body-ratio above the body threshold**. On real RTH
equity sessions:

- Roughly half of bars are bull, half bear (any one session: ~50/50).
- ~55 % of bars have body > 0.50 of range.
- The intersection — strong-body bars in one direction — is therefore
  ~25 % of bars on average.

A 40 % majority floor against a ~25 % per-side base rate means the
classifier rarely clears the gate. The 1/800 fire rate we observe is
exactly what the math predicts.

## Why the recommended floor of 0.25 works

At 0.25, the classifier asks for **just above the per-side base rate**
of strong-trend bars. It triggers when one side gets a meaningful
asymmetric tilt without requiring the session to be a one-way Brooks
trending day. The 90 % directional accuracy across 78 fires shows the
asymmetry is reliably predictive of the realised session direction.

The **+0.15 / +0.30 increments above the floor** for the +1 / +2
buckets stay structurally identical. So under floor=0.25:

- ratio > 0.55 → +2 (extreme)
- ratio > 0.40 → +1 (strong)
- ratio > 0.25 → 0 (passing the majority test)
- ratio ≤ 0.25 → -1 (clear failure of the majority test)

Production today uses 0.40 / 0.55 / 0.70 — the +2 bucket at 0.70 is
only attainable on Brooks textbook trending days, which is part of the
reason production saw zero +2 fires across 800 sessions.

## Action items (gated on Will's nod, per scanner CLAUDE.md)

1. **PRIMARY** — lower `MAJORITY_TREND_BAR_BODY_RATIO` is **NOT** the
   right move (sweep proved it). Instead introduce a new constant
   `MAJORITY_TREND_BAR_FLOOR = 0.25` and have
   `_score_majority_trend_bars` use it.
   - Blast radius: trend.py contributor `majority_trend_bars` becomes
     non-degenerate. Equal-weight aggregator already reads it; output
     becomes more directional.
   - Test impact: existing replay-equivalence tests (`MajorityTrendBarsReplayEquivalence`)
     pin the *current* outputs and would need their expected values
     re-baselined under the new floor.
   - **Do not flip until Will signs off.**
2. **Re-run the incr-17 redundancy and weighting hit-rate studies after
   the change.** The current incr-17 |r|_avg = 0.158 figure for
   `majority_trend_bars` would shift; need to re-measure how the new
   non-degenerate signal correlates with the magnitude family
   (especially `bpa_trend_bar_density`).
3. **Pattern Lab WR study** still blocked on DB backfill.
4. **Larger sample.** 800 sessions across 9 trading dates is enough to
   surface the floor-vs-body-ratio finding. Multi-month / RTH-only ES
   futures sample would let the same study run on overnight too.

## Mistakes avoided / lessons baked in

1. **The "constant 0 → drop it" inference was premature.** Incr 17's
   default response was either drop the contributor or recalibrate the
   body-ratio threshold. Both would have been wrong. The body-ratio
   threshold is fine; the floor is broken. A two-axis sweep was needed
   to surface that.
2. **Decomposed accuracy by predicted side.** With a 66 % base rate for
   "always-up", a single accuracy number could be misleading. The
   decomposition shows both up- and down-votes beat their respective
   base rates — the signal is real, not a directional-bias artifact of
   the period.
3. **Realised-direction definition matters.** Used a 1 bp deadband
   (`|move| < 0.001`) to avoid scoring near-zero-move sessions as
   either up or down. 51 of 800 fell in the flat bucket; excluding
   them from accuracy preserves the asymmetric base-rate caveat above.
4. **Sample bias caveat.** All 800 sessions are 2026-04-09 → 2026-04-19,
   a 9-trading-day window with a 62/32 up/down realised bias. Future
   recalibration validation should pull a longer window.

## Where things live

- **New tools:**
  - `~/code/aiedge/scanner/tools/majority_trend_bars_calibration_incr18.py`
  - `~/code/aiedge/scanner/tools/majority_trend_bars_floor_sweep_incr18.py`
- **New figures:**
  - `figures/majority_trend_bars_body_ratio_distribution.png`
  - `figures/majority_trend_bars_threshold_sensitivity.png`
  - `figures/majority_trend_bars_session_scores.png`
  - `figures/majority_trend_bars_floor_sweep.png`
- **New data artifacts:**
  - `majority_trend_bars_calibration_incr18.csv` (per-session scores at
    every body-ratio sweep value)
  - `majority_trend_bars_floor_sweep_incr18.csv` (per-session scores at
    every floor sweep value + realised direction)
  - `majority_trend_bars_calibration_incr18.json`
  - `majority_trend_bars_floor_sweep_incr18.json`
- **This note:** `trend-contributor-findings-2026-04-19-incr18-majority-floor.md`
- **Read first:**
  - `trend-contributor-findings-2026-04-19-incr17-followups.md`
  - `trend-contributor-findings-2026-04-19-incr15-capstone.md`

## Authority

Read-only research under scanner CLAUDE.md "refactors and internal
carves are fine." Pattern Lab and replay-equivalence tests are byte-stable.
The threshold change recommended above is **not** applied — that requires
Will's explicit nod per the scanner's standing direction on detector
thresholds.
