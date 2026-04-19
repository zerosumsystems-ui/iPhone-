# SPT — Post-3R Runner Analysis (2026-04-18, pt 16)

Follow-up to [[small-pullback-trend-stack-validation-2026-04-18]] (pt 14) and [[small-pullback-trend-targets-and-urgency-2026-04-18]] (pt 3).

Pt 3 closed the 2R → 3R target question on the **raw** SPT population (n=310). The stack then hardened to 9 rules and pt 14 locked down n=83 trades at +1.392R / trade. Open question #2 in pt 3 ("post-2R path distribution for ambiguous runners ... testable via chart_json replay") was never executed on the hardened stack.

This note answers the more general version: **on the default stack (pt 14's REVISED, n=83), is there R left on the table past the 3R target?**

Script: `~/code/aiedge/scanner/scratch/spt_runner_analysis_2026_04_18.py`. Output snapshot: `_out_spt_runner_analysis_2026_04_18.txt`.

## 1. Fixed-target sweep — massive gain between 3R and 5R

Default stack trades resolved against progressively wider targets (same 1R stop, same entry, same filter set):

| Target | n  | WR    | sumR    | perR    | σ(R)  |
|-------:|---:|------:|--------:|--------:|------:|
|   2.0R | 83 | 66.3% | +75.96  | +0.915  | 1.389 |
|   3.0R | 83 | 66.3% | +105.35 | +1.269  | 1.745 |
| **4.0R** | **83** | **65.1%** | **+128.59** | **+1.549** | **2.113** |
| **5.0R** | **83** | **65.1%** | **+135.49** | **+1.632** | **2.292** |
|   6.0R | 83 | 65.1% | +141.77 | +1.708  | 2.437 |
|   8.0R | 83 | 65.1% | +142.28 | +1.714  | 2.480 |
|  10.0R | 83 | 65.1% | +146.28 | +1.762  | 2.619 |

**Raising the target from 3R to 5R gains +0.36R / trade (+29%)** on the same population, same filters, same max DD (−4.00R across every variant). One trade that scratched at 3R flipped back to a −1R loss at 4R+ (WR drops 66.3% → 65.1%), but the 12 additional winners that run to 4R and the 12 that run to 5R swamp it.

The curve saturates by 6R. Going from 5R to 6R gains +0.08R/trade; 6R to 8R gains +0.006R; 8R to 10R gains +0.05R (one 10R+ fat-tail winner). **5R is the practical ceiling.**

## 2. Peak-MFE distribution

Every default-stack trade's maximum favorable excursion over the chart_json horizon, irrespective of actual exit:

| Bucket | n  | cum%  | avg_MFE |
|--------|---:|------:|--------:|
| < 0R   |  0 |  0.0% | —       |
| 0–1R   | 10 | 12.0% | +0.44   |
| 1–2R   | 16 | 31.3% | +1.53   |
| 2–3R   | 17 | 51.8% | +2.46   |
| 3–4R   | 10 | 63.9% | +3.38   |
| 4–5R   | 14 | 80.7% | +4.35   |
| 5–7R   | 13 | 96.4% | +5.96   |
| 7–10R  |  0 | 96.4% | —       |
| 10R+   |  3 |100.0% | +13.39  |

48.2% of default-stack trades reach ≥ 3R favorable at some point. Of the 34 eventual 3R-target WINs, 76.5% also reach ≥ 4R, 58.8% reach ≥ 5R, 35.3% reach ≥ 7R, and 5.9% run past 10R. Pt 3's earlier "90%+ of winners run past 3R MFE" finding on the RAW population is even stronger on the **filtered** stack.

The 7–10R bucket is empty because the three ≥ 10R trades cluster at +13.4R average — these are the genuine TFO / Spike-and-channel outlier days that the stack is designed to catch. They're also why the Sharpe falls as target widens (σ jumps from 1.75 to 2.62); but for expectancy-maximization without leverage constraints, wider wins.

## 3. Scale-out / runner strategies are dominated by simple higher target

Every common scale-out variant tested on the same 83 trades:

| Variant | n  | WR    | sumR    | perR    |
|---------|---:|------:|--------:|--------:|
| B0: baseline 3R single-leg                    | 83 | 66.3% | +105.35 | +1.269 |
| B1: 50% @ 3R, 50% runner → 5R (BE after 3R)   | 83 | 66.3% | +120.92 | +1.457 |
| B2: 50% @ 3R, 50% runner → 6R (BE after 3R)   | 83 | 66.3% | +124.06 | +1.495 |
| B2b: 50% @ 3R, 50% runner → 8R (BE after 3R)  | 83 | 66.3% | +124.32 | +1.498 |
| B3: 33% @ 2R, 33% @ 4R, 34% runner → 6R       | 83 | 66.3% | +109.66 | +1.321 |
| B4: 50% @ 3R, 50% trail 1R giveback from peak | 83 | 66.3% | +106.26 | +1.280 |
| B5: 50% @ 3R, 50% trail 2R giveback from peak | 83 | 66.3% | +118.04 | +1.422 |
| B6: 50% @ 4R, 50% trail 2R giveback from peak | 83 | 65.1% | +133.07 | +1.603 |

Every partial-exit variant loses to a single-leg 5R target (+1.632R). Two reasons:

1. **The 50% exit at 3R is paid for in runner upside.** When the runner runs to 6R, you've banked (0.5 × 3R) + (0.5 × 6R) = 4.5R instead of a straight 6R on full size.
2. **The BE-after-3R protection is illusory value.** In the default stack, the baseline 3R-target WR is 66% — so of 83 trades, only 8 trades out of the 34 that reached 3R failed to continue to 5R. That's 8 × (5R − 3R) × 50% = 8R in "saved losses" by scaling; but the 12 trades that would have reached 5R on full size pay (12 × 2R × 50%) = 12R for that safety. Net cost: 4R.
3. **Pt 3's H-policy (three-tier) was the best of the raw population**, but on the filtered population it under-performs because the filter removes the losers H was designed to offset.

The tightest trailing variant (B4, 1R giveback) barely moves the needle (+0.011R vs baseline) — trails exit on the first normal 1R pullback and give back most of the runner.

B6 (50% at 4R, trail 2R) is the only scale-out that matches the simple 4R target within noise (+1.603R vs +1.549R on pure 4R), but still loses to pure 5R.

**Policy implication: don't scale out. Lift the target.**

## 4. Side asymmetry — longs run further

Per-direction, at candidate targets:

| Direction | target | n  | WR    | sumR   | perR   |
|-----------|-------:|---:|------:|-------:|-------:|
| long      |   3.0R | 62 | 66.1% | +77.47  | +1.250 |
| long      |   4.0R | 62 | 66.1% | +98.47  | +1.588 |
| long      | **5.0R** | **62** | **66.1%** | **+107.41** | **+1.732** |
| long      |   6.0R | 62 | 66.1% | +111.69 | +1.801 |
| short     |   3.0R | 21 | 66.7% | +27.88  | +1.328 |
| short     | **4.0R** | **21** | **61.9%** | **+30.12**  | **+1.434** |
| short     |   5.0R | 21 | 61.9% | +28.09  | +1.338 |
| short     |   6.0R | 21 | 61.9% | +30.09  | +1.433 |

**Longs compound cleanly out to 6R.** Shorts plateau at 4R — raising to 5R actually regresses per trade (+1.434 → +1.338) because the extra winners that would run 4R→5R on longs don't exist on shorts. The market's "scared rally" dynamic is real: short winners take profit or get scare-bounced before making runner-scale moves.

Short sample is small (n=21) so this should be treated as a directional prior, not a hard rule. But it's consistent with pt 11's 11:00–11:30 ET short-squeeze finding: shorts face a *structural* buy-the-dip dynamic on the tape.

## 5. Setup asymmetry — L1 doesn't extend

Per-setup:

| Setup | target | n  | WR    | sumR   | perR   |
|-------|-------:|---:|------:|-------:|-------:|
| **H1** |  3.0R | 43 | 67.4% | +53.53 | +1.245 |
|       |  4.0R | 43 | 67.4% | +67.18 | +1.562 |
|       | **5.0R** | **43** | **67.4%** | **+75.68** | **+1.760** |
| **H2** |  3.0R | 19 | 63.2% | +23.95 | +1.260 |
|       |  4.0R | 19 | 63.2% | +31.29 | +1.647 |
|       | **5.0R** | **19** | **63.2%** | **+31.72** | **+1.670** |
| **L1** |  3.0R | 17 | **58.8%** | +17.94 | **+1.055** |
|       |  4.0R | 17 | 52.9% | +18.18 | +1.069 |
|       |  5.0R | 17 | 52.9% | +14.73 | **+0.866** |
| **L2** |  3.0R |  4 | 100.0% | +9.94  | +2.485 |
|       |  4.0R |  4 | 100.0% | +11.94 | +2.985 |
|       |  5.0R |  4 | 100.0% | +13.36 | +3.340 |

**L1 is the outlier**: raising target from 3R to 5R *costs* −0.19R per trade and drops 1 WIN (the one trade in the 3–4R MFE bucket that would have become a loss at 4R). L1 setups in SPT don't extend past 3–4R as reliably as H1/H2 — consistent with the short-side asymmetry (most L1 trades are shorts and most shorts plateau at 4R).

L2 (n=4, too small) looks spectacular but we cannot act on it alone; note the 100% WR.

H1 and H2 both want 4R+; H1 still wants 5R.

## 6. Max drawdown

Identical across every variant tested:

| Variant | final R | max DD |
|---------|--------:|-------:|
| B0 (3R)  | +105.35 | −4.00 |
| B1 (3R+runner to 5R) | +120.92 | −4.00 |
| B2 (3R+runner to 6R) | +124.06 | −4.00 |
| B3 (three-tier)      | +109.66 | −4.00 |
| B4 (tight trail)     | +106.26 | −4.00 |
| B5 (loose trail)     | +118.04 | −4.00 |
| Fixed 5R             | +135.49 | −4.00 |
| Fixed 6R             | +141.77 | −4.00 |

Because losses always exit at −1R regardless of target, DD is governed by cluster of consecutive losses, not exit policy. **Lifting the target is a free improvement on the DD dimension.**

Variance grows (σ(R) 1.75 at 3R → 2.44 at 6R), so Sharpe-style metrics will regress. But per-trade expectancy is the right metric here — positions are sized to 1R risk and DD is flat across variants.

## 7. Finding: pt 14 resolver MFE/MAE fallback bug

While replicating pt 14's resolver, found a latent issue:

```python
mfe = fallback_mfe or 0.0    # stored in PRICE UNITS
mae = fallback_mae or 0.0    # stored in PRICE UNITS
if mae <= -1.0:
    return -1.0, "stop_fallback"
if mfe >= target_r:          # target_r is in R!
    return target_r, "target_fallback"
```

MFE/MAE in the `detections` table are stored in price units (verified via `SELECT MIN/MAX(mfe)` = `[-1.25, 175.91]` across the SPT universe — MFE values of $175 are clearly not R-units). The `if mae <= -1.0` branch never triggers (mae is non-negative magnitude). The `if mfe >= target_r` branch silently fires on any high-priced stock whose mfe (in dollars) exceeds the target's R threshold — e.g., a $150 stock with $0.40 risk and mfe=$2.05 (5.1R) where chart_json ran out before target hit, pt 14 records +3R target hit; but a $500 stock with $0.40 risk and mfe=$2.05 (5.1R) where chart_json ran out, pt 14 ALSO records +3R because `2.05 >= 3.0` is false — so this particular case is unaffected, but a $150 stock with mfe=$4 (10R) where chart_json ran out would trigger the fallback.

Practically, this resolver replaces exactly **1 trade out of 83** on my pure bar-walk replication — changing WR 66.3% → 67.5% and perR +1.269 → +1.392. Well inside the pt 14 bootstrap 95% CI [+1.003, +1.773], so pt 14's conclusions still stand.

Flagged for follow-up note: audit pt 10-14 resolvers, normalize MFE/MAE-to-R conversion before any fallback test. Every result reported in this note (pt 16) uses pure bar-walk + chart_end-close with NO MFE/MAE fallback — i.e., is honest-pessimistic.

## 8. Proposed PLAYBOOK change

**Raise rule 9 target from 3R → 5R.**

Expected gains at full stack:

| | current (3R) | proposed (5R) | Δ |
|-|-------------:|--------------:|--:|
| WR    | 66.3%   | 65.1%   | −1.2 pp |
| sumR  | +105.35 | +135.49 | +30.14R |
| perR  | +1.269  | +1.632  | +0.363R/trade |
| max DD| −4.00   | −4.00   | 0 |

**Caveats / sub-rules**:

- **L1 setups stay at 3R.** The 17 L1 trades regress at higher targets. Add a sub-rule: "L1 setups use a 3R target; all others (H1/H2/L2) use 5R."
- **Shorts cap at 4R.** Optional tightening: "shorts (any setup) use max 4R target." The 21-trade sample is modest, so this is a soft recommendation.
- **Keep stop at 1R and rule 9's "hold to resolution" otherwise.** All seven tested trailing/partial variants lost to a clean single-leg exit.

Conservative single-parameter change (applies to all setups uniformly) is 4R; that gains +0.28R/trade (+23R total) and loses 1 WIN trade. The 5R variant adds another +0.08R/trade at the cost of 0 additional WR or DD.

## 9. Open questions created by this note

- **Q26** — L1 underperformance at higher targets may be a short-side / small-sample artifact. Verify after short-side subset grows (or drop L1 separately if it persists).
- **Q27** — Does the 5R-target choice generalize outside the current 8-month window? The three ≥ 10R trades are likely clustered in a specific regime; walk-forward test at 5R before committing.
- **Q28** — Resolver audit (pt 14 MFE/MAE unit bug). Does any prior pt 10–14 conclusion flip when the fallback is dropped entirely or normalized to R? Unlikely to change the headline conclusion (rule 8 drop), but confirmatory.
- **Q29** — Hybrid rule: 3R on L1, 5R on H1/H2, 4R on L2 / shorts — runs full stack re-validation (walk-forward + LOO-week) to confirm the per-setup / per-side cuts are robust before codifying.

## 10. Cross-references

- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — pt 3 (raw-population target analysis, precursor to this note).
- [[small-pullback-trend-stack-validation-2026-04-18]] — pt 14 (baseline 3R numbers for the hardened stack).
- [[small-pullback-trend-PLAYBOOK]] — needs a rule-9 edit if recommendation adopted.
- [[small-pullback-trend-INDEX]] — append pt 16 row.
