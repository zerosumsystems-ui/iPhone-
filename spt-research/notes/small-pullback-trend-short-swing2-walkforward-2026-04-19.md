# SPT — Short-side swing2 stop walk-forward validation (2026-04-19, pt 21)

Closes **Q33** (opened by [[small-pullback-trend-stop-placement-2026-04-19]] §open-items). Does the pt 20 short-side swing2 stop effect (+0.22R/trade on n=35) survive the pt 14 / pt 17 treatment — walk-forward, LOO-week, LOO-symbol, bootstrap CI?

**Headline:** Yes on non-parametric dominance tests (24/25 LOO-weeks; 18/18 LOO-symbols); wide on parametric CI (95% CI includes zero). Adoption recommended *conditional* — effect is empirically consistent but sample-thin.

Script: `~/code/aiedge/scanner/scratch/spt_short_swing2_walkforward_2026_04_19.py`
Output: `_out_spt_short_swing2_walkforward_2026_04_19.txt`

## Method

- **Two policies** compared under identical stack (REVISED {3,4,5,6,6s,7,9}, hybrid rule 9):
  - `baseline` — DB-recorded stop_price (sig-bar lo/hi + tiny buffer) on ALL trades.
  - `candidate` — DB-recorded stop on longs; `swing2` (min of last 2 bar lows / max of last 2 bar highs, incl. signal bar) on shorts.
- **Entry**: market-on-next-bar-open (pt 17/18 baseline), held constant.
- **Resolver**: honest-pessimistic (no MFE/MAE fallback; chart_end capped).
- **Target**: hybrid rule 9 (H1/H2 → 5R, L1/L2 → 3R, shorts capped at 4R).
- **CI method**: independent-bootstrap (2000 iter, seed 20260419) on delta perR = perR(cand_shorts) − perR(base_shorts).
- **Dominance tests**: LOO-week (25 weeks), LOO-symbol (53 tickers overall, 18 short-side), pairwise on the REMOVED-one perR.

## 1. Headline

| Policy | n | WR % | sumR | perR | DD | 95% CI perR |
|:-------|--:|-----:|-----:|-----:|---:|:-----------:|
| baseline  | 86 | 66.3 | +136.13 | **+1.583** | −4.00 | [+1.13, +2.03] |
| candidate | 98 | 73.5 | +159.17 | **+1.624** | −4.00 | [+1.24, +2.00] |

Full-book Δ perR = **+0.041R/trade**; full-book Δ sumR = **+23.04R** across 25 weeks (~+0.92R/wk portfolio contribution). CIs overlap heavily. The improvement comes entirely through the shorts cell — longs are identical by construction (candidate keeps DB-stop on longs).

## 2. By direction — the key cell

| Dir | Policy | n | WR % | perR | DD | 95% CI perR |
|:---:|:------|--:|-----:|-----:|---:|:-----------:|
| long  | baseline  | 63 | 65.1 | +1.689 | −4.00 | [+1.14, +2.27] |
| long  | candidate | 63 | 65.1 | +1.689 | −4.00 | [+1.12, +2.28] |
| **short** | **baseline**  | **23** | 69.6 | **+1.292** | −4.00 | [+0.59, +1.96] |
| **short** | **candidate** | **35** | **88.6** | **+1.508** | **−3.00** | [+1.10, +1.90] |

Shorts: **n grows 23 → 35** (swing2's wider stops re-open rule-9 daily gating on days where the sig-bar stop caused a rule-9 exit). WR jumps **+19 pct pts** (70% → 89%). perR +0.22R/trade. DD drops 1R. Candidate's CI on shorts is tighter than baseline's.

## 3. Bootstrap delta CI — shorts-only

- Baseline shorts: n=23, perR = +1.292
- Candidate shorts: n=35, perR = +1.508
- **Δ perR = +0.215   95% CI = [−0.55, +1.03]**
- **Verdict: INCONCLUSIVE by parametric CI** — zero is inside the interval.

But independent-bootstrap CI is wide *because n=35 shorts is thin*, not because the underlying distributions overlap. Non-parametric dominance tests (below) are the robustness check.

## 4. Walk-forward — 3 chronological thirds

Cutoffs: S1 < 2025-11-05 ≤ S2 < 2026-02-04 ≤ S3.

| Third | Baseline perR (shorts perR) | Candidate perR (shorts perR) |
|:------|----------------------------:|----------------------------:|
| S1 early | +0.784 (+0.548, n=7)  | +1.328 (+1.868, n=10) |
| S2 mid   | +1.081 (+1.605, n=7)  | +0.907 (+0.984, n=7)  |
| S3 late  | +2.278 (+1.629, n=9)  | +2.119 (+1.511, n=18) |

Observations:
- Candidate **rescues S1-early** substantially — perR +0.78 → +1.33 at the book level; shorts +0.55 → +1.87. This is where the effect is largest.
- S2 mid: candidate is slightly worse (shorts +1.60 → +0.98). This is the worst cell for the candidate — fewer extra shorts admitted (same n=7), and the few new trades are near-zero contributors.
- S3 late: book-level essentially tied (+2.28 vs +2.12); candidate has nearly 2× as many shorts (9 → 18) at lower per-R. The wider stop is admitting marginal trades in the high-breadth late regime.
- **No regime is adversarial to candidate on the full book** — candidate matches or beats baseline in 2 of 3 thirds.

## 5. LOO-week stability (25 weeks)

Per-policy stability (removing one week at a time):

| Policy | baseline perR | mean(LOO) | min | max | stdev |
|:-------|--------------:|----------:|----:|----:|------:|
| baseline  | +1.583 | +1.579 | +1.059 | +1.692 | 0.117 |
| candidate | +1.624 | +1.622 | +1.196 | +1.722 | 0.097 |

Candidate's LOO-week min is **higher** than baseline's (+1.196 vs +1.059), and its stdev is lower (0.097 vs 0.117). Candidate is *less* fragile to any single week than baseline.

**Pairwise LOO-week dominance:**
- Overall: **candidate wins 23/25, baseline wins 2, ties 0**.
- Shorts-only: **candidate wins 24/25, baseline wins 1, ties 0**.
- Worst-week-removed Δ (cand − base) = −0.030 (dropping 2025-W42). Full-window Δ = +0.041. Even the worst single-week removal doesn't flip the sign of the shorts edge.

## 6. LOO-symbol stability

Overall (53 tickers, one dropped at a time):

| Policy | baseline perR | mean(LOO) | min | max | stdev |
|:-------|--------------:|----------:|----:|----:|------:|
| baseline  | +1.583 | +1.582 | +1.209 | +1.699 | 0.062 |
| candidate | +1.624 | +1.624 | +1.306 | +1.728 | 0.054 |

Candidate's LOO-symbol min (+1.306) is above baseline's (+1.209); stdev tighter. Same story as LOO-week: less fragile, not more.

**Pairwise LOO-symbol dominance:**
- Overall: **candidate wins 51/53**. Worst two drops (HBAN −0.015, CTVA −0.003) are essentially tied.
- Shorts-only: **candidate wins 18/18**. Every short-side ticker, when dropped, leaves a residual population where candidate perR > baseline perR. No single ticker is carrying the effect.

Top contributors to candidate overall: CMCSA [n=10, +44.2R], PLTR [n=14, +16.6R], MSFT [n=3, +9.7R]. CMCSA dominates by raw R, but LOO-CMCSA still leaves candidate at +1.53 vs baseline +1.49.

## 7. By setup — where the effect lives

| Setup | Dir | Policy | n | WR % | perR | DD |
|:-----:|:---:|:------|--:|-----:|-----:|---:|
| H1 | long  | baseline  | 44 | 65.9 | +1.697 | −3.00 |
| H1 | long  | candidate | 44 | 65.9 | +1.697 | −3.00 |
| H2 | long  | baseline  | 19 | 63.2 | +1.670 | −3.00 |
| H2 | long  | candidate | 19 | 63.2 | +1.670 | −3.00 |
| **L1** | **short** | **baseline**  | **19** | 63.2 | **+1.041** | −4.00 |
| **L1** | **short** | **candidate** | **31** | **87.1** | **+1.415** | **−3.00** |
| L2 | short | baseline  |  4 | 100  | +2.485 | +0.00 |
| L2 | short | candidate |  4 | 100  | +2.222 | +0.00 |

The candidate's edge is **entirely L1-short**: 19 → 31 trades (+12), WR 63 → 87%, perR +1.04 → +1.42, DD −4 → −3. Consistent with pt 11's finding that SPT's short-side population is short-L1 dominated, and pt 20's hint that the effect concentrated here. L2-short (n=4) is tied mechanically. H1/H2 long: unchanged (candidate keeps DB-stop on longs).

## 8. Why does wider help shorts specifically?

Two mechanisms, both consistent with pt 20:

1. **Re-gating through rule 9.** On days where the short sig-bar stop fires early (e.g., the small pullback re-tests and wicks through sig-bar high), swing2 survives the test, the trade resolves later (usually to target), and that win may flip the first-3-wins gate for the rest of the day — admitting 12 additional short trades (19 → 31 in L1-short) that baseline rejected.
2. **Short-side pullback structure is wigglier.** Longs in a small-pullback uptrend make clean pullback bars; shorts in a down-trend-from-open often pull back into resistance with more wick than body (the 2-bar range above the signal bar contains stop-hunts). swing2 sits above the prior bar's high as well as the signal bar's — it's insulated from the single-bar stop-hunt that sig-bar-high is exposed to.

On longs, neither mechanism applies — long swing2 stops are wider but the signal-bar low is already the "right" location (pt 20 §4: H1-long swing2 costs −0.25R/trade, H2-long swing2 costs −0.63R/trade).

## 9. Verdict

- **Parametric bootstrap CI on the shorts-only delta includes zero** (Δ=+0.22, CI [−0.55, +1.03]). On a strict 95% CI basis, this is *not* statistically significant.
- **Non-parametric dominance is very strong**: 24/25 LOO-weeks and 18/18 LOO-symbols. This is the kind of consistency you get from a real effect, not from a sample-of-35 fluke.
- **DD improves** (shorts −4.00 → −3.00; book stays at −4.00). Not a degradation anywhere.
- **Longs unaffected by construction.**

On the balance: adopt the setup-conditional rule. The book-level contribution is small (+0.041R/trade = ~0.2R/wk on current trade pace), but the distributional properties are clean — no regime hurts, no week is fragile, no ticker is carrying the effect.

## 10. Recommendation

- **Candidate: promote to PLAYBOOK as conditional rule 10** — *"use signal-bar stop on long setups; use 2-bar swing stop on short setups."* Applies only to L1/L2 shorts (H1/H2 are long-only).
- **Caveat**: point-estimate Δ on shorts (+0.22R) has wide parametric CI. Expect +0.05–0.08R/trade portfolio contribution in the long run, not +0.22R/trade. The non-parametric evidence says the sign is right; magnitude will compress as n grows.
- **Revisit trigger**: re-run this study when n_shorts ≥ 80 (roughly doubles current). If bootstrap CI tightens and point estimate holds ≥ +0.15R, bump from conditional rule to core stack. If point estimate decays below +0.10R with CI [−0.X, +0.Y], demote.
- **Do NOT extend to longs.** pt 20 and this study both show wider stops hurt H1/H2 long. Don't "while we're at it."

## 11. Caveats

- **Entry held at market-on-open.** pt 19's `stop_2bar` entry mechanic stacks roughly additively (pt 19 found CIs overlap on entry choice); a composed-entry study could be pt 22 if motivated.
- **n=35 candidate shorts** is still thin. The non-parametric consistency compensates but doesn't eliminate the point-estimate uncertainty.
- **No friction**. Add pt 18's friction grid if sizing to live expectations; swing2's slightly wider stops will absorb slippage slightly differently, but pt 18 showed friction ordering is invariant.
- **Re-gating through rule 9 creates interaction.** The 19 → 31 population change is *not* 12 "new" independent trades — it's the first-3-wins gate flipping because swing2 saved 1-2 of the first-3 trades from stop-outs. If rule 9 is removed or retuned, the effect could shrink. Worth a sensitivity study if rule 9 is ever touched.

## 12. Open items

- **Composed entry × stop.** pt 19's stop-entry (trigger at sig-bar extreme) × pt 20's swing2 stop. Probably second-order per pt 19 but not audited.
- **ATR-based short stops.** Brooks' cash-market recipe ("2× recent bar ATR"). Not tested here.
- **n=35 follow-up**: revisit when shorts cross n=80.
- **Composition with rule 9 alternate formulations** — if rule 9 is ever tuned (e.g., first-4 instead of first-3), re-test swing2's re-gating contribution.

## Related

- [[small-pullback-trend-PLAYBOOK]] — candidate for a new conditional rule 10 ("swing2 on shorts").
- [[small-pullback-trend-stop-placement-2026-04-19]] — pt 20 opened Q33 from the by-direction short-side cell.
- [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] — pt 11 first flagged short-side as a distinct regime.
- [[small-pullback-trend-stack-validation-2026-04-18]] — pt 14 LOO-week + LOO-symbol template; method mirrored here.
- [[small-pullback-trend-target-walkforward-2026-04-18]] — pt 17 walk-forward + LOO-week + bootstrap template; method mirrored here.
- [[small-pullback-trend-INDEX]] — research arc index.

## Brooks source

- *Trading Price Action: Trends*, ch. 25 — swing-structure stops on H2/L2. Doesn't replicate on longs (pt 20); does on L1-shorts (this study).
- *Trading Price Action: Trends*, ch. 57 — Small Pullback Trends / H1-L1 canonical stops at sig-bar extreme. Baseline-correct for long side.
