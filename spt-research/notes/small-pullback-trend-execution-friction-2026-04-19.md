# SPT — Execution friction sensitivity (2026-04-19, pt 18)

Opens a new question introduced by [[small-pullback-trend-target-walkforward-2026-04-18]] closing its own: **does the hybrid rule-9 edge survive realistic slippage?** Pt 17 ran on a perfect-fill resolver (fills at `post[0].o`, exact target/stop triggers). Before Will adopts the hybrid rule, a stress test on execution friction is owed.

Script: `~/code/aiedge/scanner/scratch/spt_execution_friction_2026_04_19.py`. Output: `_out_spt_execution_friction_2026_04_19.txt`.

## Method

- **Friction model.** Parameterize adverse slippage as a fraction of per-trade risk (`k * |entry − stop|`). Asset-agnostic; the scanner is already unitless.
- **Four slippage events per trade:** entry, stop-hit exit, target-hit exit, chart-end exit. Each pays its own `k`.
- **Symmetric grid.** `k ∈ {0, 2, 5, 10, 20}%` applied identically to all four events.
- **Asymmetric scenario.** Stops bite 2× target slip (fast-market reality).
- **Variants tested.** `global_3R` (current PLAYBOOK rule 9), `global_5R` (pt 17 co-winner), `hybrid` (pt 17 proposed).
- **Stack.** PLAYBOOK-REVISED `{3, 4, 5, 6, 6s, 7, 9}` — same as pt 17.
- **Rule-9 (first-3-gate) evaluation.** The gate uses pre-friction R to decide the "first 3 won" condition. This matches how Will would evaluate it live (the gate decision doesn't know whether friction ate part of R).

## 0. Population note — DB grew by 1 trade

The DB added 1 raw SPT candidate overnight (709 → 710; revised-stack trades 83 → 84). Pt 17's post-adoption headline shifts slightly:

| variant | pt 17 (n=83) | pt 18 (n=84) | Δ perR |
|:--------|:------------:|:------------:|:------:|
| global_3R | +1.269 | +1.242 | −0.027 |
| global_5R | +1.632 | +1.601 | −0.031 |
| hybrid    | +1.630 | +1.599 | −0.031 |

Max DD unchanged (−4.00R for 3R and hybrid, −4.35R for 5R). WR drops ~1pp in each variant. The new trade is a small loss. All pt 17 conclusions (hybrid dominance on DD, 25/25 LOO-week over 3R) still hold — this is one trade inside bootstrap CI, not a regime break.

## 1. Symmetric friction grid

| variant   | k=0%  | k=2%  | k=5%  | k=10% | k=20% |
|:----------|------:|------:|------:|------:|------:|
| global_3R | +1.242 | +1.202 | +1.142 | +1.042 | +0.842 |
| global_5R | +1.601 | +1.561 | +1.501 | +1.401 | +1.201 |
| hybrid    | +1.599 | +1.559 | +1.499 | +1.399 | +1.199 |

Per-trade cost of friction is **≈ 2k × R** (each trade pays entry + exit friction). This explains the near-linear degradation: +0.04R/trade lost per 2pp of `k`.

Max drawdowns scale the same way:

| variant   | k=0%  | k=5%  | k=10% | k=20% |
|:----------|------:|------:|------:|------:|
| global_3R | −4.00 | −4.40 | −4.80 | −5.65 |
| global_5R | −4.35 | −4.95 | −5.55 | −6.80 |
| hybrid    | −4.00 | −4.40 | −4.80 | −5.65 |

**Hybrid's DD advantage over global_5R widens under friction** — from 0.35R at no friction to 1.15R at 20%. At any realistic friction level, hybrid's max-DD is strictly better.

## 2. Breakeven friction

| variant   | breakeven `k` (perR → 0) |
|:----------|:------------------------:|
| global_3R | ≈ 62% |
| global_5R | ≈ 80% |
| hybrid    | ≈ 80% |

Interpretation: to wipe the edge on a single fill, slippage would need to be ~80% of per-trade risk. For a $1 risk/share trade (e.g. a $100 stock, $1 stop), that's $0.80 of adverse fill on *each* of entry and exit. In 5-minute US equities execution with reasonable discretion, realistic friction is <10% per event (often <3%). **Edge is friction-robust.**

## 3. Asymmetric friction — stops bite 2× target

Stops trigger as market orders in fast conditions (the very bars that fire a stop are usually wide). Targets are limit exits, more controlled.

| scenario                       | variant   | perR   | DD     |
|:-------------------------------|:----------|-------:|-------:|
| low  (e=2, s=4, t=2, c=2%)     | global_3R | +1.196 | −4.24 |
| low  (e=2, s=4, t=2, c=2%)     | global_5R | +1.554 | −4.69 |
| low  (e=2, s=4, t=2, c=2%)     | **hybrid** | **+1.552** | **−4.24** |
| mid  (e=5, s=10, t=5, c=5%)    | global_3R | +1.126 | −4.60 |
| mid  (e=5, s=10, t=5, c=5%)    | global_5R | +1.484 | −5.20 |
| mid  (e=5, s=10, t=5, c=5%)    | **hybrid** | **+1.482** | **−4.60** |
| high (e=10, s=20, t=10, c=10%) | global_3R | +1.009 | −5.20 |
| high (e=10, s=20, t=10, c=10%) | global_5R | +1.367 | −6.05 |
| high (e=10, s=20, t=10, c=10%) | **hybrid** | **+1.365** | **−5.20** |

Under the "mid" scenario — a reasonable real-world baseline (5% entry/target/end, 10% stop) — hybrid retains +1.48R/trade at −4.60R DD. The original cost assumption for pt 17's hybrid adoption decision is unchanged.

## 4. Variant ordering stability

| k   | hybrid − 3R | hybrid − 5R | 5R − 3R | ranking                                |
|:---:|:-----------:|:-----------:|:-------:|:---------------------------------------|
|  0% | +0.356 | −0.002 | +0.359 | 5R (+1.601) > hyb (+1.599) > 3R (+1.242) |
|  2% | +0.356 | −0.002 | +0.359 | 5R (+1.561) > hyb (+1.559) > 3R (+1.202) |
|  5% | +0.356 | −0.002 | +0.359 | 5R (+1.501) > hyb (+1.499) > 3R (+1.142) |
| 10% | +0.356 | −0.002 | +0.359 | 5R (+1.401) > hyb (+1.399) > 3R (+1.042) |
| 20% | +0.356 | −0.002 | +0.359 | 5R (+1.201) > hyb (+1.199) > 3R (+0.842) |

**Ordering is invariant to friction.** All three pairwise perR gaps stay constant because friction is a per-trade additive cost and the trade populations are identical across variants (same filter stack; only the target_r differs). The 0.002R edge of 5R over hybrid on expectancy is preserved, but hybrid's DD advantage compounds. **Pt 17's adoption recommendation stands under any realistic friction.**

## 5. Reason-code breakdown at 5% symmetric friction (hybrid)

| reason     | n  | WR%    | perR    | share |
|:-----------|---:|-------:|--------:|------:|
| target     | 20 | 100.0% | +4.100 | 24% |
| stop       | 28 |   0.0% | −1.100 | 33% |
| chart_end  | 36 |  97.2% | +2.075 | 43% |

**Caveat on chart_end.** 43% of trades resolve as chart-end exits — they never hit target or stop within the chart window. With 97% of these green (+2.075R each, capped at target), the honest-pessimistic resolver is almost certainly **underestimating** true edge: most of these runners would likely have continued to target in live trading, turning +2R-ish resolutions into +3–5R target hits.

This caveat applies equally to all variants; it is not a hybrid-specific bias.

## 6. Conclusion

**Hybrid rule-9 adoption is safe under any realistic execution-friction model.**

- Breakeven friction is ~80% of risk per fill — 10–20× worse than realistic execution.
- At realistic levels (5% symmetric or mid-asymmetric), hybrid still delivers +1.48 to +1.50R per trade.
- Hybrid's DD advantage over global_5R widens under friction (0.35R → 1.15R at 20%).
- Variant ranking is invariant to friction: 5R ≈ hybrid >> 3R at every `k`.

The 0.002R/trade point-estimate gap between 5R and hybrid remains inside bootstrap noise, as in pt 17 §3. The DD advantage tips the decision to hybrid.

## Open items (low-priority)

- **Chart_end resolver pessimism.** 43% of the hybrid's 84 trades resolve as chart_end (no explicit target/stop hit within chart window). A follow-up could re-simulate these trades on bars beyond the chart_json window (requires stitching additional 5m bars from pattern_lab's underlying Databento cache). Likely lifts all variants' perR by 0.3–0.6R, not a ranking-changing effect.
- **Entry-mechanic model.** The resolver uses `post[0].o` as entry, which approximates a market-on-open-of-next-bar. A stop-entry at signal-bar-high + 1 tick would fill earlier (better price on slow bars, worse price on fast bars). Not tested here.
- **Slippage non-stationarity.** All tests assume constant `k` across the window. Friction is likely higher in fast-market windows (rule 3 urgency ≥ 6 regime). Not a near-term concern; the 80% breakeven gives ample headroom.

## Related

- [[small-pullback-trend-PLAYBOOK]] — hybrid rule 9 proposed, pending Will's adoption.
- [[small-pullback-trend-target-walkforward-2026-04-18]] — pt 17, walk-forward + LOO-week validation.
- [[small-pullback-trend-INDEX]] — full research arc.
