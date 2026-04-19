# SPT — Alignment Filters, Directional Asymmetry, Time-to-Target Glidepath (2026-04-18, pt 5)

Sixth follow-up to [[small-pullback-trend]]. Opens **three new filters/observations** not covered by the prior five notes, and one significant asymmetry the consolidated entry policy in [[small-pullback-trend-urgency-generalization-2026-04-18]] §4 didn't yet account for.

Same Pattern Lab snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite`, 2026-01-22 → 2026-04-17. Base population: 310 resolved WIN/LOSS with-trend (H1/H2/L1/L2) detections in SPT-adjacent day types (`trend_from_open` + `spike_and_channel`).

## 1. `always_in` alignment is a weak-to-moderate additive filter

`always_in` is the scanner's bar-by-bar institutional-side classification. Cross-tabbed with setup direction:

| always_in vs setup | n   | WR    | e₃       | avg urgency |
|--------------------|----:|------:|---------:|------------:|
| **unclear** (dominant) | 214 | 27.6% | **0.00R** | 3.93 |
| **aligned**        |  74 | 39.2% | **+0.45R** | 3.23 |
| **opposed**        |  22 | 13.6% | **−0.45R** | 4.74 |

Two observations:

1. The majority bucket (~69%) is `unclear` — the scanner's `always_in` classifier is conservative. Missing this signal does not disqualify.
2. **`always_in` opposed is a genuine trap** — WR 13.6%, negative expectancy even at a 3R target. And its avg urgency (4.74) is the highest of the three buckets: the scanner is firing loudly right when the always-in tape disagrees. Same "loud-right-before-failure" pattern as the 14:00-ET bucket ([[small-pullback-trend-empirics-2026-04-18]] §1).

**Rule addendum**: add `always_in != opposed(setup_direction)` to the filter stack. Removes 22 unprofitable trades.

## 2. Gap direction is a *strong* additive filter — stronger than `always_in`

| gap vs setup | n   | WR    | e₃       | avg urgency |
|--------------|----:|------:|---------:|------------:|
| gap_aligned  | 224 | 33.5% | **+0.23R** | 4.84 |
| gap_opposed  |  86 | 18.6% | **−0.33R** | 1.16 |

Gap-opposed detections cluster hard at low urgency — average 1.16 vs gap-aligned's 4.84. So `urgency ≥ 4` already removes most gap-opposed trades mechanically. But even on a raw-population basis, **gap-aligned has ~1.8× the WR and flips expectancy**.

Brooks' text on SPT mentions gap-opens in trend direction as a frequent opener ("large gap in eventual trend direction" — ch. 57). This shows up: gap-aligned detections fire more urgently, pull back less, and convert more reliably.

**Interaction with `always_in` (urgency-gated, u ≥ 4, SPT regimes)**:

| ai × gap                 |   n | WR    | e₃       |
|--------------------------|----:|------:|---------:|
| aligned + gap_aligned    |  26 | 53.8% | **+1.15R** |
| unclear + gap_aligned    | 100 | 50.0% | **+0.91R** |
| opposed + gap_aligned    |  11 | 27.3% | +0.09R |
| *any* + gap_opposed      |   9 |  0.0% | **−1.00R** |

- Gap_aligned with ai∈{aligned, unclear} is the workhorse: 126 trades at ~50–54% WR.
- **Gap_opposed under urgency ≥ 4 loses 100% of the time** (n=9). The combination is a filter-stack hole the prior notes didn't identify.

**Rule addendum**: require `gap_direction` aligned with setup direction (or, equivalently, reject `gap_opposed`). Eliminates another 9 certain-loss trades from the u≥4 population.

## 3. Short-side SPT is meaningfully weaker than long-side in this window

Separating by direction inside the urgency-gated SPT population:

| direction | day_type          | u≥4 | n  | WR    | e₃       |
|-----------|-------------------|:---:|---:|------:|---------:|
| **long**  | spike_and_channel | yes | 25 | **64.0%** | **+1.56R** |
| **long**  | trend_from_open   | yes | 53 | **52.8%** | **+1.11R** |
| short     | spike_and_channel | yes | 22 | 31.8% | +0.27R |
| short     | trend_from_open   | yes | 46 | 34.8% | +0.20R |

At urgency ≥ 4:
- **Longs**: ~53–64% WR, +1.11R to +1.56R per trade.
- **Shorts**: ~32–35% WR, barely positive (+0.20 to +0.27R).

This is a ~20–30 pp WR gap. Two candidate explanations:

- **Regime bias**: the 2026-01-22 → 2026-04-17 window is majority bull-regime. Short SPT days are less common and may fire during counter-trend pullbacks rather than real bear-trend-from-open days. Until the dataset spans a meaningful bear leg, the long-side may look structurally favored.
- **Asymmetry in market structure**: short-covering rallies in a channel squeeze bear setups more than long setups get squeezed. Brooks does note bulls-buy-at-market behavior is specific to strong up-tapes; the symmetric claim for bears is less emphasized.

**Practical implication**: the consolidated policy in [[small-pullback-trend-urgency-generalization-2026-04-18]] §4 assumed symmetric treatment. It should be direction-aware: longs use the existing u≥4 (S&C) / u≥6 (TFO) thresholds; **shorts should use u≥6 across both regimes** or be suppressed entirely pending more bear-regime data.

Shorts at `urgency < 4`: 10–13% WR, −0.48 to −0.66R. Removing those is already part of the gate. But the 32–35% WR at u≥4 for shorts means the short side is still marginal — one bad run could swing it negative.

## 4. Time-to-target: winners plateau around bar 20–25

For urgency-gated SPT winners (n=67):

| stat | bars to 2R |
|------|-----------:|
| mean | 11.1 |
| median | 10.0 |
| max | 36 |

Average MFE (among winners) is **7.19R** — winners run far past the 3R target. Raising the target further wouldn't obviously pay, though, because targets beyond 3R leak into exit-timing territory (the SPT "grind" bleeds out gradually and late-session exit rules matter).

**CK-based R glidepath** (avg cumulative R at N bars ahead, urgency-gated winners, n=62):

| bars ahead | avg R at close |
|-----------:|---------------:|
|          5 | +1.14R |
|         10 | +1.86R |
|         20 | +3.43R |
|         30 | +3.86R |

Reading:
- Most of the R accumulates **by bar 20** (~100 min on 5-min).
- Bars 20→30 add only +0.43R on average — the runner steepens then plateaus.
- **A time-stop at bar 25** on SPT trades would sacrifice very little expected R and would free up risk budget for later-session setups.

*Caveat*: these are winners only (selection bias). The glidepath describes runner shape, not entry filtering. For the policy stack the value is the bar-25 plateau: confirms that 3R is near the natural MFE ceiling for a typical winner and validates the target-side choice from [[small-pullback-trend-targets-and-urgency-2026-04-18]] §2.

## 5. `cycle_phase` distribution mostly mirrors setup direction

Inside SPT regimes, with-trend:

| cycle_phase    |   n | WR    | e₃       | avg urg |
|----------------|----:|------:|---------:|--------:|
| bull_channel   | 153 | 37.3% | **+0.39R** | 4.08 |
| bear_channel   | 135 | 23.0% | −0.17R | 3.91 |
| trading_range  |  17 |  0.0% | −1.00R | 0.73 |
| bull_spike     |   3 | 66.7% | +1.67R | 5.67 |
| bear_spike     |   2 | 50.0% | −0.50R | 1.30 |

bull_channel ≈ long setups; bear_channel ≈ short setups. So this slice mostly re-expresses §3's long/short asymmetry rather than adding new signal. Two residual observations:

- **trading_range residuals (n=17)**: 0% WR at avg urgency 0.73. These are detections where `day_type` was tagged SPT but `cycle_phase` had not yet transitioned. The urgency gate already removes them; no policy change needed.
- **Spike phases (n=5)**: samples too small to act on but flagging for future data.

## 6. Day-of-week effect — tentative, needs more data

Urgency-gated SPT with-trend (u ≥ 4):

| DOW | n  | WR    | e₃   |
|-----|---:|------:|-----:|
| Mon | 12 | 83.3% | +2.33R |
| Tue | 40 | 25.0% |  0.00R |
| Wed | 32 | 56.3% | +1.06R |
| Thu | 39 | 59.0% | +1.28R |
| Fri | 23 | 26.1% | +0.04R |

Mid-week (Wed/Thu) is meaningfully stronger than Tue/Fri. Monday's n=12 is too small to trust. The Tue weakness is counter to Brooks' general "Tuesday is a trend day" claim, but note Brooks was describing Tuesday strength *of trends writ large*, not SPT specifically.

**Do not** act on this yet — three months of data across 5 buckets leaves each bucket underpowered. Flag for re-check when the backtest window extends. If confirmed over a longer window, Tue/Fri could become a 0.5× position-size regime, not an outright exclusion.

## 7. Consolidated policy — revised after this note

Integrating all six notes:

1. **Regime gate**: `day_type IN ('trend_from_open', 'spike_and_channel')`.
2. **Urgency gate**:
   - **Longs**: `urgency ≥ 4` for `spike_and_channel`; `urgency ≥ 6` for `trend_from_open`.
   - **Shorts**: `urgency ≥ 6` across both regimes (tentative — this note §3); revisit when dataset spans a bear regime.
3. **Always-in gate**: reject `always_in = opposite(setup_direction)`. `unclear` and aligned both acceptable.
4. **Gap gate**: require `gap_direction` = setup_direction (i.e. reject `gap_opposed`). The u≥4 filter already removes most gap_opposed, but the residual 9 are a certain-loss pocket.
5. **Time-of-day gate**: suppress entries 13:45–14:45 ET and after 15:00 ET on SPT-regime days ([[small-pullback-trend-empirics-2026-04-18]] §1).
6. **Instrument class**: single-names only ([[small-pullback-trend-urgency-generalization-2026-04-18]] §2); indices need a separate methodology.
7. **Stop**: 1R, do not widen ([[small-pullback-trend-stop-and-timing-2026-04-18]] §1).
8. **Target**: 3R fixed ([[small-pullback-trend-targets-and-urgency-2026-04-18]] §2). Consider a **bar-25 time-stop** as an experimental addition (this note §4).
9. **Setup class**: H1/H2/L1/L2. Countertrend FH/FL unprofitable except the FL2-opening-fade pre-confirmation window ([[small-pullback-trend-stop-and-timing-2026-04-18]] §2).

## 8. Open questions (updated)

Still open from prior notes:

1. **Index-specific SPT filter** — blocked on schema ([[small-pullback-trend-empirics-2026-04-18]] §4).
2. **TFO 4–6 urgency trap mechanism** — blocked on sub-component persistence.
3. **OpEx / FOMC adjustments to 13:45–14:45 ET suppression**.
4. **Daily-chart SPT analog**.

New / refined from this note:

5. **Short-side asymmetry resolution** — is the 20-pp WR gap structural or bull-regime bias? Re-run the full policy stack when the backtest extends into a bear window.
6. **Day-of-week dispersion** — confirm Tue/Fri weakness on longer data; if real, adjust position sizing rather than exclude.
7. **Bar-25 time-stop** — simulate on Pattern Lab: hold for 25 bars or until 1R/3R hit, whichever first. Expected to be ~neutral but free risk capital for later-session trades.
8. **`always_in = unclear` disaggregation** — the 214-trade `unclear` bucket is by far the largest and has 0.00R expectancy. Is there an internal split (e.g., unclear-due-to-chop vs unclear-due-to-momentum-pause) that separates the +WR trades from the −WR trades?

## 9. Cross-references

- [[small-pullback-trend]] — primary concept note.
- [[small-pullback-trend-empirics-2026-04-18]] — 14:00 ET pullback + MAE + countertrend trap.
- [[small-pullback-trend-stop-and-timing-2026-04-18]] — stop-width, FL2 opening-fade.
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — 3R target, urgency ≥ 4 baseline.
- [[small-pullback-trend-urgency-generalization-2026-04-18]] — regime-dependent urgency threshold.
- Scanner: `aiedge/signals/components.py:715` (`_score_small_pullback_trend`), `aiedge/context/daytype.py`.

## 10. Appendix — queries

```sql
-- §1 always_in alignment
WITH base AS (
  SELECT direction, always_in, urgency, result,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT
  CASE
    WHEN (direction='long' AND always_in='long') OR (direction='short' AND always_in='short') THEN 'aligned'
    WHEN (direction='long' AND always_in='short') OR (direction='short' AND always_in='long') THEN 'opposed'
    WHEN always_in='unclear' THEN 'unclear' ELSE 'null' END ai_status,
  COUNT(*) n,
  AVG(CASE WHEN result='WIN' THEN 100.0 ELSE 0.0 END) wr,
  AVG(CASE WHEN result='LOSS' THEN -1.0 WHEN r_mfe>=3.0 THEN 3.0 ELSE 0.0 END) e3,
  AVG(urgency) avg_urg
FROM base GROUP BY ai_status ORDER BY n DESC;

-- §2 gap alignment
-- (same WHERE clause as §1, replace ai_status with)
-- CASE WHEN (direction='long' AND gap_direction='up') OR (direction='short' AND gap_direction='down') THEN 'aligned' ...

-- §3 bull vs bear asymmetry
-- GROUP BY direction, day_type, urgency>=4

-- §4 time-to-target on winners
WITH base AS (
  SELECT hit_target_bar, entry_price, stop_price, direction,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result='WIN'
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND urgency >= 4)
SELECT AVG(hit_target_bar), MEDIAN(hit_target_bar), MAX(hit_target_bar), AVG(r_mfe) FROM base;

-- §4 CK-based R glidepath
SELECT
  AVG(CASE WHEN direction='long' THEN (ck5_close - entry_price) / NULLIF(ABS(entry_price - stop_price),0)
           ELSE (entry_price - ck5_close) / NULLIF(ABS(entry_price - stop_price),0) END) R_at_5,
  -- repeat for ck10, ck20, ck30
  ...
FROM detections WHERE result='WIN' AND day_type IN ('trend_from_open','spike_and_channel')
  AND setup_type IN ('H1','H2','L1','L2') AND urgency >= 4 AND ck30_close IS NOT NULL;

-- §6 DOW
WITH base AS (
  SELECT strftime('%w', detection_date) dow, urgency, result,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND urgency >= 4
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT dow, COUNT(*) n,
  AVG(CASE WHEN result='WIN' THEN 100.0 ELSE 0.0 END) wr,
  AVG(CASE WHEN result='LOSS' THEN -1.0 WHEN r_mfe>=3.0 THEN 3.0 ELSE 0.0 END) e3
FROM base GROUP BY dow ORDER BY dow;
```
