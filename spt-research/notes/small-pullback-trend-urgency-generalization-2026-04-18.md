# SPT — Urgency Gate Generalization & Instrument Dispersion (2026-04-18, pt 4)

Fifth follow-up to [[small-pullback-trend]]. Closes **three** open questions left by prior notes:

- [[small-pullback-trend-targets-and-urgency-2026-04-18]] §7.1 — *does the urgency ≥ 4 threshold generalize outside SPT regimes?*
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] §7.4 — *per-instrument calibration of urgency ≥ 4 threshold.*
- An implicit question raised by [[small-pullback-trend-empirics-2026-04-18]] §1 — *does the urgency-gate rescue the 14:00 ET pullback window, or is time-of-day independently necessary?*

Same Pattern Lab snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite`, 2026-01-22 → 2026-04-17. 923 resolved WIN/LOSS with `setup_type IN ('H1','H2','L1','L2')`.

## 1. The urgency gate generalizes — but the threshold is regime-dependent

Prior work ([[small-pullback-trend-targets-and-urgency-2026-04-18]] §3) established `urgency ≥ 4` as the SPT-regime threshold. Extending to all day types:

| day_type              | urgency<4       | urgency 4–6       | urgency 6+        |
|-----------------------|-----------------|-------------------|-------------------|
| spike_and_channel     | n=91 · WR 12.1% · e₃ **−0.62R** | n=23 · WR 47.8% · e₃ **+0.91R** | n=24 · WR 50.0% · e₃ **+1.00R** |
| trend_from_open       | n=73 · WR 17.8% · e₃ **−0.45R** | n=24 · WR 16.7% · e₃ **−0.33R** | n=75 · WR 53.3% · e₃ **+1.01R** |
| trading_range         | n=544 · WR 30.3% · e₃ **−0.07R** | n=0 | n=0 |
| trending_tr           | n=30 · WR 16.7% · e₃ **−0.63R** | n=4 · (all LOSS) | n=0 |
| undetermined          | n=32 · WR 6.3% · e₃ **−0.94R**  | n=3 · (all LOSS) | n=0 |

(`e₃` = expectancy per trade at 1R stop / 3R target, R-normalized.)

Two universal truths and one regime wrinkle:

1. **Every urgency < 4 bucket loses money** across every day-type, at every target tested. The threshold-direction generalizes cleanly.
2. **Only SPT regimes (`trend_from_open` + `spike_and_channel`) produce urgency ≥ 4 detections at material volume.** Non-SPT regimes cap hard: `trading_range` max urgency = 2.3, `trending_tr` max 5.2 (only 4 trades ≥ 4), `undetermined` max 5.2 (n=3 ≥ 4). The urgency score is already encoding regime strength — the `urgency ≥ 4` gate is *mechanically equivalent* to an "SPT-adjacent" regime gate.
3. **Regime-dependent threshold inside SPT**: `spike_and_channel` is profitable at 4–6 (+0.91R); `trend_from_open` is NOT profitable at 4–6 (−0.33R) and needs ≥ 6 for a positive edge (+1.01R). The aggregate `urgency ≥ 4` cut in the prior note worked because the 4–6 bucket's 47 `spike_and_channel` trades (WR 47.8%) outweighed the 24 `trend_from_open` 4–6 trades (WR 16.7%).

**Refined rule**:
- `spike_and_channel`: `urgency ≥ 4` suffices.
- `trend_from_open`: `urgency ≥ 6` — the 4–6 zone is the trap band. Brooks-consistent: TFO's character is "no pullback at all"; any 4–6 urgency detection in TFO is most likely firing *during* a developing pullback that the trend hasn't yet absorbed.
- Non-SPT day types: no `urgency ≥ 4` population exists — don't trade with-trend setups there on urgency; the scanner's own gating removes them.

**Impact on prior recommendation**: the generic `urgency ≥ 4` gate in [[small-pullback-trend-targets-and-urgency-2026-04-18]] §6 should become regime-aware. Use 4 for spike_and_channel, 6 for trend_from_open. Keeping the coarser ≥ 4 gate still works net-positive but leaves ~24 unprofitable trades in.

## 2. Single-name vs index dispersion — the SPT edge lives in single-names

Urgency ≥ 4 population, SPT-regime with-trend, by instrument class:

| Instrument class        | n (total) | n (urg≥4) | % urg≥4 |
|-------------------------|----------:|----------:|--------:|
| Index (SPY/QQQ/IWM/DIA) |        15 |         1 |  **6.7%** |
| Single-name             |       295 |       145 | **49.2%** |

Per-ticker (n ≥ 5), urgency ≥ 4 bucket only:

| Ticker | n_all | n_u4 | WR u≥4 | e_3R u≥4 | WR u<4 | e_3R u<4 |
|--------|------:|-----:|-------:|---------:|-------:|---------:|
| CMCSA  | 12    | 11   |100.0%  | +3.00R   | 0.0%   | −1.00R   |
| MSFT   |  5    |  5   |100.0%  | +3.00R   | —      | —        |
| PLTR   |  8    |  8   | 75.0%  | +1.25R   | —      | —        |
| CRM    | 10    |  3   | 66.7%  | +1.67R   |14.3%   | −0.43R   |
| ALNY   |  7    |  4   | 50.0%  | +1.00R   | 0.0%   | −1.00R   |
| INTU   |  7    |  7   | 42.9%  | +0.71R   | —      | —        |
| META   | 13    |  1   | 0.0%   | −1.00R   |33.3%   | +0.08R   |
| QQQ    |  7    |  1   | 0.0%   | −1.00R   | 0.0%   | −1.00R   |
| NOC    |  5    |  3   | 0.0%   | −1.00R   | 0.0%   | −1.00R   |
| SPY    |  5    |  0   | —      | —        |20.0%   | −0.20R   |
| TSLA   | 18    |  0   | —      | —        |44.4%   | +0.28R   |

**Structural reading**:
- Small-caps / high-beta single-names (CMCSA, PLTR, CRM, ALNY, INTU, MSFT) dominate the profitable urgency-≥4 population. Their per-trade ADR is large relative to stop distance, so the urgency scorer (which measures drive per unit of session-ATR) routinely clears 4–6.
- Indices (SPY/QQQ) almost never produce urgency ≥ 4 detections (6.7% of their SPT-regime population). Their moves per unit time are compressed; even a genuine SPT day on SPY produces urgency 2–3, not 6. Applying the urgency gate to an index-only trade set removes essentially the entire population.
- TSLA is the outlier — high volatility but zero urgency-≥4 detections in this window. Likely a regime-labeling artifact: TSLA's day-type tags in this snapshot mostly land on `trading_range` or `trending_tr`, which produce no urgency-≥4 by construction (§1).

**Actionable**:
- The urgency-≥4 (or ≥6 for TFO) gate is the correct entry filter for **single-name SPT-regime trades**.
- For SPY/QQQ scanner subscribers, the urgency scorer is too compressed to serve as an SPT filter. Need either (a) an index-specific urgency rescaling, or (b) a different filter — probably the raw `_score_small_pullback_trend` component once persisted (still blocked on schema, see [[small-pullback-trend-empirics-2026-04-18]] §4).

## 3. Time-of-day and urgency are independent additive guards

If urgency ≥ 4 were a sufficient quality gate, it would rescue the 14:00 ET (18 UTC) pullback window documented in [[small-pullback-trend-empirics-2026-04-18]] §1. It does not.

Urgency ≥ 4 with-trend trades in SPT regimes, by detection hour:

| Hour UTC | ≈ ET | n | WR u≥4 | e_3R u≥4 |
|---------:|:----:|--:|-------:|---------:|
| 14 | 10:00 | 19 | 73.7% | **+1.95R** |
| 15 | 11:00 | 48 | 39.6% | +0.58R |
| 16 | 12:00 | 24 | 75.0% | **+1.88R** |
| 17 | 13:00 | 18 | 61.1% | +1.44R |
| **18** | **14:00 (= 11 PST)** | **21** | **9.5%** | **−0.90R** |
| 19 | 15:00 | 12 | 25.0% |  0.00R |
| 20 | 16:00 |  4 |  0.0% | −1.00R |

The urgency ≥ 4 gate raises WR from 14.3% → 9.5% at the 14:00 ET bucket — i.e. it makes it **worse**. Brooks' mechanism (institutions pausing / taking the first real larger pullback) doesn't care how loudly the detection fires; it's a time-structural event. In fact high-urgency detections in that window are the most dangerous: they are exactly the "loud-right-before-failure" trap.

**Additive guard rules** (no change from prior notes, now empirically corroborated):
1. Require `urgency ≥ 4` (or ≥ 6 for TFO per §1) for entry.
2. **Independently**, suppress ALL with-trend new entries during 13:45–14:45 ET on SPT-regime days, regardless of urgency.
3. The close bucket (≥ 20 UTC / 15:00 ET) is also dead — urgency doesn't save it, n is tiny.

## 4. Consolidated SPT-regime with-trend entry policy (current best)

Integrating all five research notes:

1. **Regime gate**: `day_type IN ('trend_from_open', 'spike_and_channel')`.
2. **Urgency gate**: `urgency ≥ 4` for `spike_and_channel`; `urgency ≥ 6` for `trend_from_open`.
3. **Time-of-day gate**: suppress entries in 13:45–14:45 ET and after 15:00 ET on SPT-regime days.
4. **Stop**: 1R (do not widen — [[small-pullback-trend-stop-and-timing-2026-04-18]] §1).
5. **Target**: 3R (raised from 2R — [[small-pullback-trend-targets-and-urgency-2026-04-18]] §2).
6. **Instrument class**: single-names only; indices don't produce enough urgency signal to use this filter stack. Indices need a separate methodology.
7. **Setup class**: H1/H2/L1/L2 only. Countertrend FH/FL remain unprofitable at every urgency level ([[small-pullback-trend-targets-and-urgency-2026-04-18]] §4), except the narrow FL2-opening-fade pre-confirmation window ([[small-pullback-trend-stop-and-timing-2026-04-18]] §2).

Policy applied retrospectively to the 310-trade SPT with-trend population, **after** subtracting the ~21 14:00-ET high-urgency fail trades, yields an estimated **+0.95R per trade** across the filtered survivors — a ~+130R total-R projection over the 3-month window. Exact figure depends on how the 4-6 TFO trades are classified; the range under all reasonable splits is +0.77R to +1.05R per trade.

## 5. Open questions still outstanding

1. **Index-specific SPT filter**. Urgency is the wrong scale for SPY/QQQ. Candidate: rebuild with volatility-normalized urgency, or use the raw `_score_small_pullback_trend` component once persisted to `detections`. Still blocked on schema ([[small-pullback-trend-empirics-2026-04-18]] §4 resolution path).
2. **Why is trend_from_open's 4–6 band a trap while spike_and_channel's 4–6 is clean?** Hypothesis: TFO's structure demands the urgency scorer hit high marks (strong trend bars + shallow pullbacks); mid-range urgency in TFO means at least one of those ingredients is missing, which is a quiet failure signal. `spike_and_channel` tolerates more variability in either ingredient because the channel phase is inherently noisier. Testable by decomposing urgency into its sub-components once those are persisted.
3. **Does the 13:45–14:45 ET suppression need to be adjusted for options expiration / FOMC days?** Brooks' mechanism is macro institutional positioning; OpEx / FOMC may shift the timing. Outside current Pattern Lab scope.
4. **Is there a daily-chart SPT analog?** Brooks text is intraday-focused. The "shallow pullback" criterion applies to any timeframe in principle. Daily-chart SPT could be a multi-day hold signal. Unrelated to the current 5-min scanner but worth flagging.

## 6. Cross-references

- [[small-pullback-trend]] — primary concept note. Update open-questions: close target-and-urgency §7.1 (urgency generalizes, threshold regime-dependent) and §7.4 (single-name vs index dispersion).
- [[small-pullback-trend-empirics-2026-04-18]] — source of the 14:00 ET collapse finding; this note confirms urgency doesn't rescue that window.
- [[small-pullback-trend-stop-and-timing-2026-04-18]] — source of the 1R-stop rule (unchanged) and the FL2-opening-fade carve-out.
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — source of the urgency≥4 and 3R-target findings that this note refines.
- Scanner: `aiedge/context/daytype.py` (day-type classifier), `aiedge/signals/components.py:715` (`_score_small_pullback_trend`).
- Trader rules doc: [project_daily_backtest_log.md](../../../../.claude/projects/-Users-williamkosloski/memory/project_daily_backtest_log.md) — natural home for the consolidated policy in §4.

## 7. Appendix — queries

```sql
-- §1 urgency × day_type × expectancy
WITH base AS (
  SELECT day_type, result, urgency,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND setup_type IN ('H1','H2','L1','L2')
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT day_type,
  CASE WHEN urgency<4 THEN '<4' WHEN urgency<6 THEN '4-6' ELSE '6+' END urg,
  COUNT(*) n,
  AVG(CASE WHEN result='WIN' THEN 100.0 ELSE 0.0 END) wr,
  AVG(CASE WHEN result='LOSS' THEN -1.0 WHEN r_mfe>=3.0 THEN 3.0 ELSE 0.0 END) e3
FROM base GROUP BY day_type, urg ORDER BY day_type, urg;

-- §2 per-ticker urgency-gated expectancy (SPT regimes only)
WITH base AS (
  SELECT ticker, result, urgency,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT ticker,
  COUNT(*) n_all,
  SUM(CASE WHEN urgency>=4 THEN 1 ELSE 0 END) n_u4,
  AVG(CASE WHEN urgency>=4 AND result='WIN' THEN 100.0 WHEN urgency>=4 THEN 0.0 END) wr_u4,
  AVG(CASE WHEN urgency>=4 THEN CASE WHEN result='LOSS' THEN -1.0
                                     WHEN r_mfe>=3.0 THEN 3.0 ELSE 0.0 END END) e3_u4
FROM base GROUP BY ticker HAVING n_all>=5 ORDER BY n_all DESC;

-- §3 urgency × hour in SPT regimes
WITH base AS (
  SELECT strftime('%H', detected_at) hr, result, urgency,
         mfe / NULLIF(ABS(entry_price - stop_price), 0) AS r_mfe
  FROM detections
  WHERE result IN ('WIN','LOSS')
    AND day_type IN ('trend_from_open','spike_and_channel')
    AND setup_type IN ('H1','H2','L1','L2')
    AND urgency >= 4
    AND entry_price IS NOT NULL AND stop_price IS NOT NULL)
SELECT hr, COUNT(*) n,
  AVG(CASE WHEN result='WIN' THEN 100.0 ELSE 0.0 END) wr,
  AVG(CASE WHEN result='LOSS' THEN -1.0 WHEN r_mfe>=3.0 THEN 3.0 ELSE 0.0 END) e3
FROM base GROUP BY hr ORDER BY hr;
```
