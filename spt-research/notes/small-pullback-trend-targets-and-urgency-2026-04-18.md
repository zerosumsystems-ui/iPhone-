# SPT — Target-Side Optimization & Urgency Gating (2026-04-18, pt 3)

Fourth follow-up to [[small-pullback-trend]]. Closes the "target-side / trailing" experiment explicitly flagged as the next candidate in [[small-pullback-trend-stop-and-timing-2026-04-18]] §1, and surfaces a previously-unreported urgency threshold.

Same Pattern Lab snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite`, 2026-01-22 → 2026-04-17. N = 310 resolved WIN/LOSS with `day_type IN ('trend_from_open','spike_and_channel') AND setup_type IN ('H1','H2','L1','L2')`.

Note: MFE/MAE in the `detections` table are stored in **price units**, not R. Every figure below is R-normalized via `r_mfe = mfe / |entry - stop|` and analogously for MAE. Prior notes reported R-values directly; the numbers are consistent but the SQL differs.

## 1. The WIN-side leaves massive room

R-normalized MFE distribution for resolved WIN trades (n=91):

| MFE bucket | n  | cum % | avg MFE (R) |
|------------|---:|------:|------------:|
| 2.0–2.5R   |  4 |  4.4  | 2.24 |
| 2.5–3.0R   |  5 |  9.9  | 2.65 |
| 3.0–4.0R   | 21 | 33.0  | 3.51 |
| 4.0–5.0R   | 11 | 45.0  | 4.42 |
| 5.0–7.0R   | 23 | 70.3  | 6.01 |
| 7.0R+      | 26 |100.0  |12.53 (max 18.6) |

**90% of SPT with-trend WINs ran past 3R. 91% ran past 2.5R. Only 9/91 stalled between the 2R target and 3R.** The current 2R target is exiting essentially every runner prematurely.

LOSS-side MFE: of 219 losses, 178 (81%) never reached 2R favorable — genuine no-trade-worked-out. The remaining 41 (19%) went ≥2R favorable before reversing to the 1R stop.

## 2. Target-side expectancy model

Model assumptions (path-independent, R-multiple units):
- Losses always exit at -1R (mae ≥ 1R triggered stop).
- WINs at their target if mfe ≥ target; otherwise a scenario choice (BE vs -1R vs partial) below.
- Partials assume fractional exits at fixed R-levels with the remainder running to a runner target. Ambiguous runners default to BE scratch (conservative).

Over all 310 resolved SPT with-trend trades:

| Policy | Expectancy / trade | Total R (3 mo) |
|--------|-------------------:|---------------:|
| A. 1R stop, **2R target** (current)          | **−0.12R** | −37R |
| B. 1R stop, 3R target                        | +0.08R     | +24R |
| C. 1R stop, 4R target                        | +0.07R     | +21R |
| D. 50% at 2R, 50% runner → 3R (BE after 2R)  | −0.02R     | −7R  |
| E. 50% at 2R, 50% runner → 4R (BE after 2R)  | −0.03R     | −8R  |
| F. 50% at 2R, 50% runner → 5R (BE after 2R)  | −0.02R     | −6R  |
| G. 33% at 1R, 67% runner → 2R                | −0.04R     | −11R |
| H. 33% at 1R, 33% at 2R, 34% runner → 4R     | **+0.16R** | +49R |

Two structural takeaways:
- **Simple target raise (2R → 3R) flips the population from −0.12R to +0.08R** with no change to stop or entry rules.
- **Policies that scale down after 2R (D/E/F) under-perform** because the 2R partial exit prints just as the price begins its real runner move. BE-trailing doesn't catch the 3R+ extensions; a static higher target does.
- **Three-tier partial (H) is best overall** (+0.16R) because it pays for the BE-scratch cost of runners that fail with the harvested 1R and 2R partials.

### 2a. Robustness — WIN-with-mfe-below-new-target assumption

Of the 9 WINs in the 2–3R bucket, the model assumes they scratch at BE when the target is raised to 3R. Under a pessimistic assumption (they retrace to −1R instead of BE), Policy B's expectancy drops from +0.077R to +0.045R per trade — still positive, still materially better than current. Under a midway assumption (+0.5R partial via trailing), +0.094R. The 3R-target result is robust across the assumption band.

## 3. Urgency threshold — the hidden filter

The far bigger finding: 53% of the population (164/310) carries **all** of the negative expectancy. Stratifying on `urgency`:

| Urgency | n   | WR    | Exp @ 2R | Exp @ 3R | Exp @ 4R | Exp @ 3R (pess.) |
|--------:|----:|------:|---------:|---------:|---------:|-----------------:|
|   < 4   | 164 | 14.6% |  −0.56R  |  −0.54R  |  −0.51R  |  −0.59R |
|  4–5    |  20 | 30.0% |  −0.10R  |  +0.20R  |  +0.30R  |  +0.20R |
|  5–6    |  27 | 33.3% |   0.00R  |  +0.33R  |  +0.22R  |  +0.33R |
|  6–7    |  29 | 51.7% |  +0.55R  |  +1.07R  |  +1.17R  |  +1.07R |
|   7+    |  70 | 52.9% |  +0.59R  |  +0.99R  |  +0.84R  |  +0.94R |

**Every urgency < 4 bucket loses money under every target policy.** Every urgency ≥ 4 bucket is profitable at the 3R target, and the pessimistic-assumption column confirms that's not an artifact of the BE-scratch model.

Combined filter + target policy:

| Gate                     | n   | WR    | Exp @ 2R (total) | Exp @ 3R (total) |
|--------------------------|----:|------:|-----------------:|-----------------:|
| ALL (current)            | 310 | 29.4% | −0.12R (−37R)    | +0.08R (+24R)    |
| urgency ≥ 4              | 146 | 45.9% | +0.38R (+55R)    | **+0.77R (+113R)** |
| urgency ≥ 5              | 126 | 48.4% | +0.45R (+57R)    | +0.87R (+109R)   |
| urgency ≥ 6              |  99 | 52.5% | +0.58R (+57R)    | +1.01R (+100R)   |

**Headline**: `urgency ≥ 4 AND target = 3R` converts a −37R 3-month run into a +113R run — a ~150R swing on the same trade population, with no change to entry logic.

The urgency-6+ cut produces the highest per-trade expectancy (+1.01R) but on fewer trades (99 vs 146). The monotonic pattern — more selective → higher per-trade, diminishing in total — suggests **urgency ≥ 4 is the maximum-total-R cut**. Urgency ≥ 5 is a close second with a cleaner WR (48.4%).

## 4. Countertrend traps do not respond to urgency gating

Same framework applied to FH/FL setups in the same day types (n = 217):

| Urgency | n   | WR    | Exp @ 2R | Exp @ 3R |
|--------:|----:|------:|---------:|---------:|
|   < 4   | 131 |  6.9% |  −0.79R  |  −0.93R  |
|  4–5    |   9 |  0.0% |  −1.00R  |  −1.00R  |
|  5–6    |  19 | 15.8% |  −0.53R  |  −0.53R  |
|   6+    |  58 | 25.9% |  −0.22R  |  −0.02R  |

Countertrend in SPT remains unprofitable at every urgency level. High-urgency countertrend (6+) approaches breakeven at a 3R target but never becomes genuinely positive. Brooks' teaching ("shorting below bars in a strong bull trend is a loser's strategy") holds regardless of urgency selectivity. **Do not generalize the urgency-4 gate to FH/FL setups.**

## 5. Interaction with prior empirics — time-of-day & FL2

- **14:00 ET window** ([[small-pullback-trend-empirics-2026-04-18]] §1 — WR collapses to 14.3%): unchanged. The urgency filter and target raise don't rescue the 14:00 pullback window because urgency is highest in that bucket (4.58 avg) — the filter lets those trades through. The time-of-day gate is additive to the urgency gate, not replaced by it.
- **FL2 / trend_from_open opening fade** ([[small-pullback-trend-stop-and-timing-2026-04-18]] §2 — 50% WR in session bars 7–18): the urgency-4 gate for with-trend setups does NOT apply. FL2/TFO is a separate pre-confirmation pattern, not an SPT with-trend signal. Keep the existing session-bar-18 window for that setup.

## 6. Actionable changes

Ranked by impact, easiest first:

1. **Raise the SPT-regime with-trend target from 2R to 3R.** Direction-confirmed across every urgency bucket, robust to the WIN<3R assumption, single-parameter change. Expected impact: +0.20R per trade on the total population.
2. **Filter SPT-regime with-trend entries at urgency < 4.** 53% of the population, 100% of the negative expectancy. Expected impact: another +0.40R per remaining trade, at the cost of halving trade count. If total R is the metric, urg ≥ 4 is the optimum.
3. **Consider the three-tier partial (Policy H) for highest-expectancy trades (urgency ≥ 6).** Adds ~+0.10R per trade over Policy B in the high-urgency bucket by harvesting the 33% that don't reach 4R. Worth simulating in the backtest log before adopting — introduces operational complexity.
4. **Retain existing guards**: 1R stop (no widening), 13:45–14:45 ET suppression, FL2/TFO 18-bar window.

## 7. Open questions created by this note

1. **Does the urgency threshold generalize outside SPT regimes?** Only tested on `trend_from_open + spike_and_channel` × H1/H2/L1/L2. Trading-range / trending-tr may have their own urgency-WR curves. Worth testing — possible answer to the 25.5% aggregate WR on `trend_from_open` all-urgency vs 45.9% at urgency ≥ 4.
2. **Post-2R path distribution for ambiguous runners.** The 9 WIN trades with mfe ∈ [2R, 3R) are modeled as scratch; what actually happens after peak? Testable via `chart_json` replay — compute post-`hit_target_bar` price path. Would let us choose definitively between Policies B, D, H.
3. **Interaction with `_score_small_pullback_trend` component.** Same structural question as concept-note open-question #1: does the SPT component score correlate with urgency, or with expectancy *beyond* urgency? Still blocked by the same schema issue (component scores not persisted).
4. **Per-instrument calibration of urgency ≥ 4 threshold.** This snapshot is SPY/QQQ/TSLA-heavy; the cutoff might shift for other tickers as they're added.

## 8. Cross-references

- [[small-pullback-trend]] — primary concept note. Update §"Open research questions": close #4 target-side flag, add urgency-gating as a new finding.
- [[small-pullback-trend-empirics-2026-04-18]] §2 — MAE shape (source of target-side hypothesis).
- [[small-pullback-trend-stop-and-timing-2026-04-18]] §1 — stop-width expectancy (this note is that note's "target-side optimization" follow-up).
- Scanner: `aiedge/context/daytype.py` (day-type weights include SPT×urgency coupling). `aiedge/signals/components.py:715` (`_score_small_pullback_trend`).
- Daily backtest log: [project_daily_backtest_log.md](../../../../.claude/projects/-Users-williamkosloski/memory/project_daily_backtest_log.md) — candidate place to run this policy against the actual urgency-gated day-by-day trade set.

## Appendix — queries

```sql
-- §1 WIN MFE distribution (R-normalized)
WITH base AS (
  SELECT result,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result='WIN'
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT
  CASE
    WHEN r_mfe < 2.5 THEN '2.0-2.5R'
    WHEN r_mfe < 3.0 THEN '2.5-3.0R'
    WHEN r_mfe < 4.0 THEN '3.0-4.0R'
    WHEN r_mfe < 5.0 THEN '4.0-5.0R'
    WHEN r_mfe < 7.0 THEN '5.0-7.0R'
    ELSE '7.0R+' END AS bucket,
  COUNT(*) n, AVG(r_mfe) avg_mfe
FROM base GROUP BY bucket;

-- §2 target-side expectancy model (Policy B: 1R stop, 3R target)
WITH base AS (
  SELECT result,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT COUNT(*) n,
  SUM(CASE WHEN result='LOSS' THEN -1.0
           WHEN r_mfe >= 3.0 THEN 3.0
           ELSE 0.0 END)*1.0/COUNT(*) exp_3R
FROM base;

-- §3 urgency × target stratification
WITH base AS (
  SELECT result, urgency,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT
  CASE WHEN urgency<4 THEN '<4'
       WHEN urgency<5 THEN '4-5'
       WHEN urgency<6 THEN '5-6'
       WHEN urgency<7 THEN '6-7'
       ELSE '7+' END bucket,
  COUNT(*) n,
  AVG(CASE WHEN result='WIN' THEN 100.0 ELSE 0.0 END) wr,
  AVG(CASE WHEN result='WIN' THEN 2.0 ELSE -1.0 END) e_2R,
  AVG(CASE WHEN result='LOSS' THEN -1.0
           WHEN r_mfe >= 3.0 THEN 3.0 ELSE 0.0 END) e_3R
FROM base GROUP BY bucket;
```
