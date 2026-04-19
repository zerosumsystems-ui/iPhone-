# SPT — Stop-Width & Setup-Timing Empirics (2026-04-18, pt 2)

Third follow-up to [[small-pullback-trend]] / [[small-pullback-trend-empirics-2026-04-18]]. This note answers open-question #4 (stop-width trap) and the flagged FL2-in-`trend_from_open` anomaly. Open-question #3 (next-day follow-through) is **not testable with current data** — documented in §3.

Same Pattern Lab snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite`, 2026-01-22 → 2026-04-17, 306 resolved WIN/LOSS with `day_type IN ('trend_from_open','spike_and_channel') AND setup_type IN ('H1','H2','L1','L2')`.

## 1. Stop-width expectancy — widening HURTS

**Hypothesis from [[small-pullback-trend-empirics-2026-04-18]] §2**: MAE shape shows 12/90 winners (13%) drew down > 1R before running to 2R. Brooks' "don't move stops to BE too early" suggests a wider stop might net-capture these.

**Test**: compute portfolio expectancy under three stop/target policies, conservatively assuming (a) wins survive wider stops if `mae < new_S`, and (b) losses only convert to wins if `mae < new_S` AND `mfe ≥ 2.0`; otherwise losses grow to `new_S`.

| Policy      | Wins | Losses | Expectancy/trade |
|-------------|-----:|-------:|-----------------:|
| 1R stop, 2R target (current) | 90 | 216 | **-0.12R** |
| 1.5R stop, 2R target | 83 | 223 | -0.55R |
| 2R stop, 2R target   | 87 | 219 | -0.86R |

**Result**: widening stops makes expectancy strictly worse. The ~5 winners reclaimed at 1.5R / ~8 at 2R are overwhelmed by the extra -0.5R / -1R cost paid across 200+ losers whose `mae` already exceeded the new stop.

**Interpretation**: Brooks' teaching ("don't move stops to BE too early") is *directionally* correct — the current 1R stop already beats every narrower stop. But the empirics note's implied hope that *wider* stops would rescue the 12 premature stop-outs was wrong. The loss-side cost dominates in the current population. The right optimization is **not** stop-side; it is either:
- Filter the loser pool (day-type gating, time-of-day gating per empirics §1), or
- Target-side (trail past 2R on strong SPT tape) — unexplored.

**Actionable**: do not widen the SPT-regime stop. Keep 1R. Next candidate experiment: trailing-stop or scale-out at 1R+T+ to capture the ~12 winners that ran past 2R without giving back.

## 2. FL2 / `trend_from_open` anomaly — it's an opening-fade, not a countertrend

**From [[small-pullback-trend-empirics-2026-04-18]] §3**: FL2 in `trend_from_open` posted 22.4% WR (n=49) — the only FH/FL setup in SPT regimes with >20% WR. Flagged as "worth isolating to see whether it's the 'first real pullback after a trend leg' L2-on-the-short-side case."

**Test**: bucket FL2/TFO detections by `session_bar_number` (bar number within the trading session).

| Session phase | n | WR | avg urgency | avg MFE | avg MAE |
|---|---:|---:|---:|---:|---:|
| Early (bars 7–18, first ~90 min) | 8 | **50.0%** | 5.35 | 2.72 | 0.76 |
| Mid (bars 19–36, ~2–3 hrs in) | 20 | 25.0% | 5.49 | 0.62 | 1.13 |
| Late (bars 37–54, afternoon) | 16 | 12.5% | 3.00 | 0.16 | 0.61 |
| Close (bars 55+) | 5 | 0.0% | 6.50 | 0.14 | 1.71 |

**Result**: the 22.4% aggregate WR is driven entirely by the early-session bucket (50% WR, n=8, avg MFE 2.72R). After bar 18, FL2/TFO collapses to 12–25% WR and below-1R MFE. The "close" bucket (n=5, 0% WR, 1.71 avg MAE) is the classic Brooks trap — high urgency, zero wins.

**Interpretation**: early-session FL2 in a nominal `trend_from_open` day is **not** a countertrend short; it's a fade of an opening bull spike before the trend's direction is confirmed. The day_type classifier can't distinguish "opening climax that fails" from "already-established trend" in the first 90 min — both get labeled `trend_from_open` in retrospect. Brooks' teaching ("shorting below bars in a confirmed bull trend is a loser's strategy") is confirmed by the bar-36+ data: 8/41 wins = 19.5% there. The edge lives only in the pre-confirmation window.

**Actionable**: FL2/TFO is tradeable only before session bar 18 (~90 min). After that it's part of the countertrend trap band. This aligns with Brooks' general pattern: the best Brooks-book countertrend entries are at the EARLY edge of a suspected climax, before the trend has had time to prove itself.

## 3. Next-day follow-through — NOT testable with current Pattern Lab data

**From concept note open-question #3**: Brooks claims SPT days leak into the first 1–2 hours of the next session. Proposed test: tag historical SPT days, check next-day opens.

**Why it fails now**:
- **Candidate scarcity**: only 12 ticker-days in Pattern Lab meet a strict SPT threshold (`trend_from_open` AND ≥3 same-direction with-trend setups AND zero opposite). At a looser threshold (≥2 same-direction, zero opposite): 44 candidates.
- **First-hour blind spot**: across the entire 1,606-resolved Pattern Lab snapshot, only 30 detections landed in `session_bar_number ≤ 12` (the first hour). The scanner's warm-up and urgency gates push almost all detections past bar 12. So "next-day first-hour continuation" is essentially unobservable from `detections`.
- **Relaxed version tried anyway** (next-day ALL detections, ≥2 threshold): 4 continuation / 6 reversal / 6 other resolved trades, WR 0% / 67% / 33% respectively. Directionally opposite Brooks' claim, but n is too small to conclude anything.

**Deferred**. Revisit when:
- Per-day urgency-gated backtest log ([project_daily_backtest_log.md](../../../../.claude/projects/-Users-williamkosloski/memory/project_daily_backtest_log.md)) extends far enough back to yield ≥100 clean SPT days.
- Or: drop first-hour filter and test next-day **entire session** continuation — still needs SPT-day sample size that doesn't yet exist.
- Or: swap data source to `chart_json` replay with looser filters than the live scanner applies.

## 4. Summary — what changed in SPT ops guidance

1. **Keep 1R stop on SPT-regime with-trend setups.** Widening to 1.5R or 2R strictly decreases expectancy on current data. The 13% of winners that draw >1R are a real cost but smaller than the cost of wider losses. Target-side (trailing / partial at 2R+) is the right optimization axis.
2. **FL2 / `trend_from_open` is a session-timing play, not a regime play.** Only tradeable before session bar ~18 (~90 min in). After that, it collapses into the same countertrend trap as FH1/FH2/FL1. The aggregate 22.4% WR reported in the prior empirics note overstates the edge; the true edge is ~50% WR confined to n=8 opening entries.
3. **Next-day follow-through remains unproven.** Not a contradiction of Brooks — just not testable with a 3-month scanner dataset. Deferred.

## Cross-references

- [[small-pullback-trend]] §"Open research questions" — this note closes #4, partially closes FL2 flag, defers #3.
- [[small-pullback-trend-empirics-2026-04-18]] §2 (MAE shape) — source of the stop-width hypothesis tested here.
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — the target-side follow-up explicitly flagged in §1 of this note; 3R target + urgency ≥ 4 filter flips expectancy from −0.12R to +0.77R per trade.
- [[failed-breakout]] — FL1/FL2/FH1/FH2 semantic framing.
- Scanner day-type classifier: `aiedge/context/daytype.py` (the source of the `trend_from_open` label that proves too coarse in §2).

## Appendix — queries

```sql
-- §1 stop-width expectancy (conservative model)
WITH base AS (
  SELECT result, mae, mfe FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2'))
SELECT 'wider_2R_2R',
  SUM(CASE WHEN result='WIN' AND mae<2.0 THEN 1
           WHEN result='LOSS' AND mae<2.0 AND mfe>=2.0 THEN 1 ELSE 0 END) wins,
  SUM(CASE WHEN result='WIN' AND mae>=2.0 THEN 1
           WHEN result='LOSS' AND (mae>=2.0 OR mfe<2.0) THEN 1 ELSE 0 END) losses
FROM base;

-- §2 FL2 timing
SELECT
  CASE WHEN session_bar_number<=6 THEN 'opening'
       WHEN session_bar_number<=18 THEN 'early'
       WHEN session_bar_number<=36 THEN 'mid'
       WHEN session_bar_number<=54 THEN 'late'
       ELSE 'close' END bucket,
  COUNT(*) n,
  AVG(CASE WHEN result='WIN' THEN 100.0 ELSE 0.0 END) wr
FROM detections
WHERE result IN ('WIN','LOSS') AND day_type='trend_from_open' AND setup_type='FL2'
GROUP BY bucket;

-- §3 next-day follow-through (relaxed threshold, sample size confirmed too small)
WITH spt AS (
  SELECT ticker, detection_date,
    CASE WHEN SUM(CASE WHEN setup_type IN ('H1','H2') THEN 1 ELSE 0 END)>=2
              AND SUM(CASE WHEN setup_type IN ('L1','L2') THEN 1 ELSE 0 END)=0 THEN 'bull'
         WHEN SUM(CASE WHEN setup_type IN ('L1','L2') THEN 1 ELSE 0 END)>=2
              AND SUM(CASE WHEN setup_type IN ('H1','H2') THEN 1 ELSE 0 END)=0 THEN 'bear'
         ELSE NULL END dir
  FROM detections WHERE day_type='trend_from_open' GROUP BY ticker,detection_date)
SELECT d.setup_type, v.dir prior, d.result
FROM detections d
JOIN spt v ON d.ticker=v.ticker
  AND julianday(d.detection_date)-julianday(v.detection_date) BETWEEN 1 AND 3
WHERE v.dir IS NOT NULL AND d.result IN ('WIN','LOSS');
```
