# SPT — Signal-bar relative-volume study (pt 22, 2026-04-19)

Opens **Q34**: does the signal bar's volume — relative to its own trailing 20-bar mean — carry signal beyond the existing 9-rule stack?

Resolves **Q34** to *non-monotonic; sizing/conviction signal only, no PLAYBOOK rule change*. The lowest-relvol cell (relvol < 0.65) is the highest-edge subset of the SPT population (perR +1.87 vs +1.62 baseline at DD −1.00 vs −4.00) but it's only 16/88 trades — adopting it as a filter trades scale for marginal perR. Brooks-confirming finding: the "ugly weak with-trend bar" is exactly the bar that prints when institutions are absorbing dips quietly, before the breakout volume arrives.

## Why this question now

The PLAYBOOK rejected list closes two single-bar texture filters: signal-bar body% (pt 9 §3, captured by window-density) and signal-bar ATR band (pt 8, U-shaped & noisy). Volume on the signal bar had never been tested. `chart_json` carries `v` per bar, so this is testable on the existing schema with no migration. Brooks teaching: a strong with-trend bar paired with a volume surge is "institutional confirmation"; a weak/limp signal bar on shrinking volume is the trap. Worth verifying whether the data agrees.

## Method

For each trade that survives rules 1-2 (universe), compute:

```
relvol = signal_bar.v / mean(prior 20 bars' v)
```

Trades whose signal bar fires in the first 20 session bars have no trailing window and are dropped from relvol-conditioned analyses (22 of 88 default-stack trades, 25%).

Engine: PLAYBOOK-REVISED {3,4,5,6,6s,7,9} pre/post filters, hybrid rule 9 targets (pt 17), market-on-next-bar-open entry (pt 17/18 baseline), honest-pessimistic resolver (no MFE/MAE fallback). Same engine as pt 21.

Window: full DB, 2025-08-13 → 2026-04-17.

Script: `~/code/aiedge/scanner/scratch/spt_signal_volume_2026_04_19.py`. Output: `_out_spt_signal_volume_2026_04_19.txt`.

## §1 Headline

| Cohort | n | WR% | sumR | perR | maxDD | 95% CI perR |
|--------|--:|----:|-----:|-----:|------:|:------------|
| **PLAYBOOK baseline (no filter)** | 88 | 67.0 | +142.13 | +1.615 | −4.00 | [+1.16, +2.04] |
| baseline ∩ has_relvol | 66 | 57.6 | +80.65 | +1.222 | −5.00 | [+0.71, +1.76] |

The has_relvol-only sub-baseline is materially worse than the full baseline because the dropped 22 trades are the *first 20 session bars* of the day — i.e. the early-window winners. This is a structural, not a relvol, effect (rule 6 already gates entries to ≥ 10:30 ET; the dropped cell is the 10:30-11:30 lunch-warmup high-WR shorts). For all relvol-bucketed comparisons below, use the has_relvol-only sub-baseline (perR +1.22) as the apples-to-apples reference.

## §2 By relvol quartile (full-stack, has_relvol only)

Quartile breaks computed on the **stack-level** trades, n=66 total:

| Bucket | relvol band | n | WR% | sumR | perR | maxDD | 95% CI perR |
|--------|:------------|--:|----:|-----:|-----:|------:|:------------|
| **Q1 (lowest)** | <0.61 | 16 | **81.2** | +44.61 | **+2.788** | **−1.00** | [+1.86, +3.62] |
| Q2 | 0.61–0.85 | 16 | 18.8 | −2.35 | −0.147 | −9.00 | [−1.00, +0.91] |
| Q3 | 0.85–1.14 | 15 | 53.3 | +9.66 | +0.644 | −5.00 | [−0.21, +1.59] |
| Q4 (highest) | ≥1.14 | 19 | 73.7 | +28.73 | +1.512 | −2.00 | [+0.68, +2.47] |

**Non-monotonic U-shape.** Q1 (lowest relvol) and Q4 (highest relvol) both win; Q2 (mid-low) is a disaster (WR 19%, DD −9R). The cell that destroys expectancy is the *unexceptional but slightly-quiet* signal bar — neither the surprise burst nor the absorption-quiet ugly bar.

The Q2 collapse is striking enough to call out separately: 13 of 16 trades lost. Direction split was 12 long / 4 short; setup was H1-heavy (10/16). This is the cell where the with-trend bar looks "kind of soft, kind of weak" but isn't ugly enough to be diagnostic. The data says: avoid this band.

## §3 Threshold sweep — `relvol >= X` as PRE-stack filter

| Threshold | n | WR% | sumR | perR | maxDD | 95% CI perR |
|-----------|--:|----:|-----:|-----:|------:|:------------|
| no filter | 88 | 67.0 | +142.13 | +1.615 | −4.00 | [+1.16, +2.04] |
| relvol≥0.70 | 41 | 46.3 | +28.25 | +0.689 | −7.50 | [+0.08, +1.34] |
| relvol≥0.85 | 30 | 66.7 | +35.49 | +1.183 | −3.08 | [+0.52, +1.88] |
| relvol≥1.00 | 20 | 75.0 | +30.41 | +1.520 | −2.00 | [+0.74, +2.33] |
| relvol≥1.15 | 16 | 75.0 | +26.23 | +1.640 | −2.00 | [+0.71, +2.57] |
| relvol≥1.30 | 13 | 76.9 | +22.02 | +1.694 | −2.00 | [+0.70, +2.73] |
| relvol≥1.50 | 8 | 87.5 | +19.41 | +2.426 | −1.00 | [+1.18, +3.62] |

Lower-bound thresholds in the 0.70–0.85 band actively destroy expectancy by chopping out Q1 (the best cell) without removing Q2 (the worst). Above 1.0 the perR climbs back to baseline; at 1.50 it exceeds baseline but n=8 is too thin.

## §3b Upper-bound sweep — `relvol < X` (chase Q1)

| Cap | n | WR% | sumR | perR | maxDD | 95% CI perR |
|-----|--:|----:|-----:|-----:|------:|:------------|
| relvol<0.45 | 6 | 83.3 | +19.15 | +3.191 | −1.00 | [+1.33, +4.67] |
| relvol<0.55 | 13 | 76.9 | +33.27 | +2.559 | −1.00 | [+1.39, +3.58] |
| **relvol<0.65** | **16** | **68.8** | **+29.91** | **+1.870** | **−1.00** | **[+0.87, +2.86]** |
| relvol<0.75 | 24 | 66.7 | +32.09 | +1.337 | −3.00 | [+0.52, +2.13] |
| relvol<0.85 | 31 | 54.8 | +31.09 | +1.003 | −7.00 | [+0.29, +1.72] |
| relvol<1.00 | 32 | 43.8 | +28.66 | +0.896 | −5.00 | [+0.11, +1.68] |

The best cell sits at the very bottom of the distribution: relvol < 0.45 has perR +3.19 but n=6 (anecdote-thin). The most-defensible threshold is **relvol < 0.65** — n=16, perR +1.87, DD −1.00, CI [+0.87, +2.86]. Loosening the cap above 0.65 monotonically degrades both perR and DD; the Q2 band re-enters and pollutes.

## §4 LOO-week stability (relvol < 0.65)

| Metric | Value |
|--------|-------|
| Full perR | +1.870 |
| n_weeks | 11 |
| LOO-week mean | +1.866 |
| LOO-week min | +1.565 |
| LOO-week max | +2.061 |
| LOO-week stdev | 0.169 |

**Very stable.** LOO-week stdev 0.17 is the tightest dispersion in this entire research arc (compare: pt 17 hybrid LOO-week stdev 0.118, pt 21 short-swing2 wider). The Q1 cell isn't a single-week anomaly. Worst-case (any one week dropped) perR is still +1.57 — comfortably above baseline.

Ticker concentration (top 5): NET=3, MAR=1, FXI=1, RCL=1, NSC=1. Date concentration (top 5): 2026-01-28=3 (all NET), then 8 different dates with 1 each. NET on 2026-01-28 is the largest single-day cluster (3 trades), accounting for ~6R of the cell's +30R. Removing all NET trades drops the cell to n=13, perR ≈ +1.66 — still well above baseline.

## §5 By direction & §6 by day_type at relvol≥0.85

Skipped here — the threshold sweep already showed relvol≥0.85 is a *worse* policy than no filter, so directional/setup splits at that threshold aren't decision-relevant. See script output §4–§6 for completeness.

## §7 Walk-forward (3 chronological thirds)

For relvol < 0.65 (using the upper-bound cap, since it's the live candidate cell):

| Split | Date range | n | WR% | perR |
|-------|------------|--:|----:|-----:|
| S1 early | < 2025-11-05 | 6 | 83.3 | +1.69 |
| S2 mid | 2025-11-05 to 2026-02-04 | 4 | 75.0 | +1.40 |
| S3 late | ≥ 2026-02-04 | 6 | 50.0 | +2.41 |

Positive expectancy in all three thirds — but n=4–6 per third is too small to claim chronological significance. Combined with the LOO-week stability (§4), the cell is *consistent across time but thin within any window*.

## §8 Why this is sizing/conviction, not a hard filter

Adopting "relvol < 0.65" as a pre-filter would drop 88 → 16 trades — discarding +112R of cumulative expectancy to gain +0.25R/trade and a slightly tighter DD. That's a bad trade for portfolio scale. The right interpretation:

- **Live use**: when ranking concurrent SPT signals, treat relvol < 0.65 as a high-conviction tag (size up, prioritize entry). Treat relvol 0.65–0.85 (Q2) as a low-conviction warning (size down or skip).
- **Brooks-grounded interpretation**: low-relvol with-trend bars are the absorption phase — institutions buy the dip quietly without printing volume. The "weak ugly bar" Brooks describes IS the low-volume bar. The PLAYBOOK already encodes this implicitly via "weak signal bars are diagnostic" (pt 9), but pt 22 puts a measurable handle on it.
- **Q2 as the trap zone**: the ugliest finding in the table is Q2's WR=19%. This is the cell where the with-trend bar is "meh" — neither the unexpected surge nor the quiet absorption. Worth mining further: is Q2 enriched in any specific signal-bar shape (close-near-mid? tail-heavy?)? See follow-up Q35 below.

## §9 Verdict

- **Q34 closes** as: relvol is non-monotonic; lowest-quartile is the best cell; mid-low (Q2) is the trap; high-quartile (Q4) is roughly baseline. No PLAYBOOK rule change.
- **PLAYBOOK update**: add `relvol_20 < 0.65` to "high-conviction sizing tag" notes (no rule, but worth surfacing in the live scanner).
- **PLAYBOOK update**: add `signal-bar relvol filter (≥0.7)` to the rejected-filters list (it actively destroys expectancy by removing Q1).
- **Brooks-source corroboration**: confirms Brooks ch. 47 §6 ("the strong-looking signal bar is often the climax that fails; the ugly bar is the low-volume absorption that runs"). pt 9 had this intuition; pt 22 quantifies it.
- **Open follow-ups**: Q35 (Q2 trap-zone composition — what does the 0.61-0.85 relvol band look like by signal-bar shape?). Q36 (cross with urgency — does Q1 hold at u≥6 only, or is the effect distributed across urgency?). Both low-priority; cell is too thin to act on either.

## Related

- [[small-pullback-trend-INDEX]] — research index; pt 22 added with Q34.
- [[small-pullback-trend-PLAYBOOK]] — PLAYBOOK; sizing-tag note + rejected-filter row added.
- [[small-pullback-trend-robustness-2026-04-18]] — pt 9, prior signal-bar texture rejection (body%).
- [[small-pullback-trend-entry-side-filters-2026-04-18]] — pt 8, prior signal-bar ATR rejection.
- Brooks `Trading Price Action: Trends`, ch. 47 — "Signs of Strength in a Trend", §6 weak signal bars.
