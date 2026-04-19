# SPT — Bar-N Time-Stop Expectancy (2026-04-18, pt 6)

Seventh follow-up to [[small-pullback-trend]]. Closes Q12's **experimental bar-25 time-stop** addition by running the actual simulation. Result: **rejected** — holding to target strictly dominates any time-stop at bar 20 or earlier.

Population: same as prior note — H1/H2/L1/L2 in `trend_from_open` + `spike_and_channel`, `urgency >= 4`, `gap_direction` aligned with setup. **n=153 resolved** (was 146 in prior note; +7 from newly-resolved detections in the last 24 h).

Snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite` @ 2026-04-18 (afternoon run).
Script: `~/code/aiedge/scanner/scratch/spt_time_stop_analysis.py`.

## 1. Simulated strategies — summary

| Strategy                          |  n  | WR   | Sum R   | Per-trade |
|-----------------------------------|----:|-----:|--------:|----------:|
| **Baseline: 3R target, no time-stop** | 153 | 52.9% | **+139.51R** | **+0.91R** |
| Bar-30 time-stop, 3R target       | 153 | 53.6% | +138.41R | +0.90R   |
| Bar-20 time-stop, 3R target       | 153 | 51.6% | +127.58R | +0.83R   |
| Bar-10 time-stop, 3R target       | 153 | 56.9% |  +87.49R | +0.57R   |
| Current 2R target (no time-stop)  | 153 | 52.9% |  +74.17R | +0.48R   |

Readings:

- **Bar-10 time-stop is the worst** by a wide margin. It raises WR (56.9%) because more losers are exited near entry, but caps winners that haven't yet reached 3R — and 46% of winners haven't hit 2R yet by bar 10. Cost: **−52R vs baseline.**
- **Bar-20 time-stop costs ~12R** (−8% of total expectancy). Not catastrophic, but strictly worse.
- **Bar-30 time-stop is effectively equal** to the baseline (−1.1R). It almost never kicks in: only 1 of 67 winners is still open at bar 30.
- **3R target vs 2R** confirms the prior Q7 finding on this tighter population: +0.43R per trade (+65R total) simply from raising the target.

## 2. Why the time-stop fails — glidepath of still-open trades

The question is, conditional on a trade *still being open at bar N*, what's the expectancy of holding vs exiting at close:

| Still open at | n   | Hold-to-3R | Exit-at-close | Δ (exit − hold) |
|---------------|----:|-----------:|--------------:|----------------:|
| Bar 10        |  66 | +1.26R/trade | +0.47R/trade | **−0.79R/trade** |
| Bar 20        |  25 | +1.01R/trade | +0.53R/trade | **−0.48R/trade** |
| Bar 30        |  18 | +0.68R/trade | +0.61R/trade | −0.07R/trade    |

Trades that survive past bar 10 are *not* chop — they are slow winners. Exiting at their mid-trade close strips out the continuation that SPT regimes are designed to deliver.

**Winner time-to-target** (2R, n=67):
- Median: bar 10.
- Mean: 11.1 bars.
- Max: bar 36.
- 91% hit 2R by bar 20; **99% by bar 30.**

So the bar-25 region is almost entirely "still alive = still grinding" trades, not "stalled" ones. The premise of a time-stop — "if it hasn't worked by now, it's not going to" — doesn't hold in SPT regimes at this urgency threshold.

## 3. Why the simulated expectancy is trustworthy

The simulation approximates the 3R target by using `mfe` (max favorable excursion, known from data). For WINs with mfe ≥ 3R, we record +3R; for WINs with mfe in [2R, 3R) we record the mfe — which is optimistic (price could have reversed and hit the 1R stop after touching 2R<mfe<3R).

**Distribution of WIN `mfe` (n=67, urgency-gated SPT regime)**:

| mfe band | n   | %   |
|----------|----:|----:|
| 2.0–2.5R |  1  | 1%  |
| 2.5–3.0R |  2  | 3%  |
| 3.0–4.0R | 18  | 27% |
| 4.0–5.0R |  9  | 13% |
| 5.0R+    | 37  | **55%** |

**96% of winners run past 3R MFE.** The optimistic approximation affects only 3 trades — worst-case overstatement ≈ 3 × 2R = 6R out of +139.5R, or ~4%. The ranking between strategies is unchanged.

This also re-confirms the Brooks dictum for SPT days: winners don't "take 2R" and stop, they *grind*. The median winner continues well past 5R MFE before any reversal.

## 4. Direction and day-type splits

**Long vs short (3R target)**:

| Direction | n  | Baseline sum | Per-trade | Bar-20 TS | Δ       |
|-----------|---:|-------------:|----------:|----------:|--------:|
| long      | 92 | +108.3R      | **+1.18R** | +104.2R  | −4.1R  |
| short     | 61 |  +31.2R      | **+0.51R** |  +23.4R  | −7.9R  |

Short-side continues to run ~2× weaker than long-side (confirms Q11). Bar-20 time-stop hurts shorts more (−0.13R/trade) than longs (−0.04R/trade), likely because fewer shorts achieve the 5R+ continuation that makes late-stage holding most valuable.

**By day-type (3R target)**:

| Day-type           | n   | Baseline per-trade | Bar-20 TS per-trade | Δ       |
|--------------------|----:|-------------------:|--------------------:|--------:|
| trend_from_open    | 107 | +0.86R             | +0.75R              | −0.11R  |
| spike_and_channel  |  46 | **+1.03R**         | +1.03R              | **≈0**  |

Interesting wrinkle: spike_and_channel is **indifferent to a bar-20 time-stop**. S&C winners resolve earlier (channel phase tends to thrust to target faster), so by bar 20 most WINs have already completed. TFO days carry more slow-grinders that benefit from holding past bar 20.

This doesn't motivate a policy split — the marginal cost on TFO is small enough that a single unified "no time-stop" rule is simpler and no worse than an S&C-only exception.

## 5. Verdict on Q12

**CLOSED — rejected.** The experimental bar-25 time-stop from [[small-pullback-trend-alignment-filters-2026-04-18]] §4 does not improve expectancy.

- Bar-20 TS: costs ~12R/quarter (−8%).
- Bar-25 TS (interpolated): costs somewhere between bar-20 and bar-30, so 3–12R cost.
- Bar-30 TS: breakeven with baseline, but ~1% of trades affected — not worth the rule.

Reason: the `urgency ≥ 4` + gap-aligned filter already removes the chop trades. What remains are genuine SPT winners (median mfe > 5R), and mid-trade exits harvest less than holding. The time-stop was solving a problem — late-session chop/reversal — that the upstream filters already solve.

**Policy remains** (per [[small-pullback-trend-urgency-generalization-2026-04-18]] §4 + [[small-pullback-trend-alignment-filters-2026-04-18]] §7):
- `urgency >= 4` (≥ 6 on pure TFO), with-trend only, gap_aligned, `always_in != opposed`.
- **3R fixed target, 1R stop, hold to resolution.**
- Suppress entries 13:45–14:45 ET.
- No bar-N time-stop.

## 6. Updated expectancy benchmark

Current consolidated policy, Pattern Lab 2026-01-22 → 2026-04-18:
- **n=153 trades, WR 52.9%, +139.5R total, +0.91R/trade** at 3R target.
- ≈ 12 trades/week at current detection rate; ~11R/week theoretical.

Compare against raw unfiltered baseline on the same date window: 310 SPT-regime with-trend trades at 2R cap = −0.12R/trade (≈ −37R / 3 months), per the prior targets note. The policy stack (urgency + gap + always_in + 3R) converts **−37R → +139R**.

## 7. What's next (updated research queue)

- **Q1 (blocked)**: SPT-component-raw bucketed WR. Unchanged — requires scanner migration to persist the 0–3 raw score on detections.
- **Q5 (open)**: cross-asset SPT. The 153-trade population is 90%+ US single-names + some liquid ETFs. SPY/QQQ produce very few urgency-≥ 4 signals, consistent with the prior note's instrument-dispersion finding. Multi-asset calibration unlocks more data but depends on the forex/expansion project.
- **Q11 (open)**: short-side weakness. Now further confirmed — +0.51R (shorts) vs +1.18R (longs). Still plausibly bull-regime artifact; unchanged — revisit when data spans a bear leg.
- **New — untested**: conditional exit on *adverse* event (e.g. first bear trend bar closing below bar-10 EMA on a long). Distinct from a pure time-stop. Would require bar-by-bar event reconstruction, not just checkpoints.

## Related

- [[small-pullback-trend]] — parent concept, research queue.
- [[small-pullback-trend-urgency-generalization-2026-04-18]] — urgency threshold derivation.
- [[small-pullback-trend-alignment-filters-2026-04-18]] — gap_aligned + always_in filters, original bar-25 hypothesis.
- Analysis: `~/code/aiedge/scanner/scratch/spt_time_stop_analysis.py`.
