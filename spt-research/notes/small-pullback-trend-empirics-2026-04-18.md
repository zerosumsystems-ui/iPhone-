# SPT Empirics — Time-of-Day, MAE Shape, and Countertrend Traps (2026-04-18)

Follow-up research to [[small-pullback-trend]]. Same Pattern Lab snapshot (1,606 resolved WIN/LOSS, 2026-01-22 → 2026-04-17, 2R cap). This note answers a concrete Brooks claim and quantifies two of the "open research questions" in the concept note.

## 1. Brooks' late-session pullback claim — confirmed in data

**Brooks (Trends, ch. 57)**: *~2/3 of SPT days produce a larger pullback after 11 PST (~14:00 ET) that is ~2× the biggest prior pullback.*

With-trend (H1/H2/L1/L2) WR by detection hour, in SPT-adjacent day types (`trend_from_open` + `spike_and_channel`):

| Hour UTC | ≈ ET  | n  | WR    | Avg urgency |
|---------:|:-----:|---:|------:|------------:|
|       14 | 10:00 | 47 | 36.2% |        3.63 |
|       15 | 11:00 | 83 | 25.3% |        4.11 |
|       16 | 12:00 | 66 | 37.9% |        3.30 |
|       17 | 13:00 | 54 | 35.2% |        2.99 |
|   **18** | **14:00 (= 11 PST)** | **35** | **14.3%** | **4.58** |
|       19 | 15:00 | 17 | 23.5% |        4.95 |
|       20 | 16:00 |  5 |  0.0% |        5.54 |

Across all other hours WR averages ~30–38%. The 14:00 ET bucket (exactly Brooks' "11 PST larger pullback" window) **collapses to 14.3% WR**. Urgency in that bucket is actually the highest of the day (4.58) — i.e. detections fire loudly right before they disproportionately fail.

Countertrend in the same window is no better: FH/FL in day types `trend_from_open` + `spike_and_channel` at 18 UTC → 8.3% WR (n=24), avg MFE 0.25R. The 14:00 ET pullback is a pullback *within* the trend, not a reversal — fading still loses.

**Actionable reading** (already proposed in the concept's open-question #2): suppress or de-urgency both directions of new entry in the 13:45–14:45 ET window on days tagged `trend_from_open` / `spike_and_channel`. Brooks' prescription for that window is trim / hold, not new-entry.

## 2. MAE shape on winners — Brooks' "don't BE too early" in data

MAE distribution, WINNING with-trend trades in SPT day types (n=90):

| MAE bucket (R) | n  | cumulative % |
|---------------:|---:|-------------:|
|          ≤ 0.3 | 53 |         58.9 |
|        0.3–0.7 | 23 |         84.4 |
|        0.8–1.0 |  7 |         92.2 |
|            >1R | 12 |         100  |

Summary stats: WIN avg MAE 0.67R (max 7.51R, n=90); LOSS avg MAE 2.48R (max 40.54R, n=216).

Two observations:
1. **Majority of winners draw down < 0.3R** — the classic SPT "barely retraces" tape.
2. **~13% of winners would have been stopped by a 1R stop** (12/90). Twelve trades that did eventually run to 2R but tested further back first. Brooks' "don't move stops to BE too early — price tests entry before extending" maps onto this long tail.

A looser stop (1.5R? 2R?) would retain those 12 wins at the cost of letting more losers run to the larger stop. Whether that's net-positive requires an R-multiple backtest — **candidate experiment**: rerun urgency-gated backtest with fixed 2R stop (vs current 1R stop, 2R target), restrict to SPT day types, compare portfolio expectancy.

## 3. Countertrend trap, setup-by-setup

WR and avg MAE for FH/FL setups in SPT-adjacent day types:

| day_type / setup          |  n | WR    | Avg MAE | Max MAE |
|---------------------------|---:|------:|--------:|--------:|
| trend_from_open / FH1     | 13 |  0.0% |    1.83 |    5.74 |
| spike_and_channel / FH2   | 32 |  3.1% |    0.69 |    2.46 |
| spike_and_channel / FH1   | 16 |  6.3% |    0.76 |    2.46 |
| trend_from_open / FL1     | 26 |  7.7% |    0.94 |    2.63 |
| spike_and_channel / FL1   | 21 |  9.5% |    1.85 |   25.78 |
| spike_and_channel / FL2   | 32 | 15.6% |    1.65 |   25.78 |
| trend_from_open / FH2     | 28 | 17.9% |    1.16 |    5.74 |
| trend_from_open / FL2     | 49 | 22.4% |    0.96 |    7.61 |

Three things stand out:
- **FH1 in `trend_from_open` is a perfect trap** — 0/13. "Shorting below bars in a strong bull trend is a loser's strategy" (Brooks, Trends ch. 47) literally shows up as zero wins.
- **Max MAE of 25.78R in spike_and_channel FL1/FL2** — catastrophic outliers. Fading a channel mid-leg can produce runaway losers.
- **FL2 in `trend_from_open`** is the only countertrend setup in this regime set with >20% WR (22.4%, n=49) — worth isolating to see whether it's the "first real pullback after a trend leg" L2-on-the-short-side case rather than a pure countertrend short. Follow-up task.

## 4. Data-schema limitation blocking the main open question

Open-question #1 in the concept note — "does `_score_small_pullback_trend` add signal beyond day-type classification?" — cannot be answered directly from `pattern_lab.sqlite`. The `detections` table stores `day_type` but **not** the individual BPA component scores (no `spt_component_raw`, `spt_component_weighted`, nor the other `_score_*` outputs).

Two paths forward:
1. **Add component-score columns** to `detections` — requires touching the write path in the scanner pipeline (`aiedge/signals/` + writer). One-time migration, then future detections are queryable.
2. **Rebuild from `chart_json`** — each detection row stores a snapshot. In principle the SPT scorer could be re-run off `chart_json` to backfill the score. Cheaper than schema change but slower per-row.

Given scanner refactor is fresh (2026-04-17) and the column list is already being revised, (1) is the cleaner follow-up.

## 5. Cross-references

- Concept note: [[small-pullback-trend]] — primary definition and Brooks-grounded checklist.
- Follow-up: [[small-pullback-trend-stop-and-timing-2026-04-18]] — closes the stop-width hypothesis raised in §2 (widening hurts) and explains the FL2/TFO anomaly raised in §3 (it's an opening-fade, not countertrend).
- Related: [[always-in-direction]], [[failed-breakout]].
- Scanner: `aiedge/signals/components.py:715` (`_score_small_pullback_trend`), `aiedge/context/daytype.py` (DAY_TYPE_WEIGHTS).
- Day-type WR source file: [small-pullback-trend.md](small-pullback-trend.md) §"Empirical WR from Pattern Lab".
- Hourly + MAE queries archived below for reproducibility.

## Appendix — queries used

```sql
-- §1 time-of-day
SELECT strftime('%H', detected_at) AS hour_utc, COUNT(*) n,
       AVG(CASE WHEN result='WIN' THEN 1.0 ELSE 0.0 END)*100 wr_pct,
       AVG(urgency) avg_urg
FROM detections
WHERE result IN ('WIN','LOSS')
  AND day_type IN ('trend_from_open','spike_and_channel')
  AND setup_type IN ('H1','H2','L1','L2')
GROUP BY hour_utc;

-- §2 WIN MAE histogram
SELECT ROUND(mae, 1) mae_R, COUNT(*) n
FROM detections
WHERE result='WIN'
  AND day_type IN ('trend_from_open','spike_and_channel')
  AND setup_type IN ('H1','H2','L1','L2')
GROUP BY mae_R;

-- §3 countertrend
SELECT day_type, setup_type, COUNT(*) n,
       AVG(CASE WHEN result='WIN' THEN 1.0 ELSE 0.0 END)*100 wr_pct,
       AVG(mae) avg_mae, MAX(mae) max_mae
FROM detections
WHERE result IN ('WIN','LOSS')
  AND setup_type IN ('FH1','FH2','FL1','FL2')
  AND day_type IN ('trend_from_open','spike_and_channel')
GROUP BY day_type, setup_type;
```
