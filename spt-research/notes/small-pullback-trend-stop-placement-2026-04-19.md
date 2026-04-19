# SPT — Stop placement study (2026-04-19, pt 20)

Closes **Q32** (opened by [[small-pullback-trend-entry-mechanic-2026-04-19]] §open-items as the next study candidate).

Every prior SPT study used the DB's recorded `stop_price` — in practice the signal-bar low (long) / high (short) with a small buffer. Brooks also teaches *swing-structure* stops for pullback setups: on H2/L2, the stop belongs **beyond the prior failed attempt** (usually 2–3 bars back), not immediately behind the signal bar. Wider stops change per-trade `risk` and therefore R-normalization — we can't compare absolute P/L, we compare WR, perR, DD, and the risk-distribution.

Script: `~/code/aiedge/scanner/scratch/spt_stop_placement_2026_04_19.py`.
Output: `_out_spt_stop_placement_2026_04_19.txt`.

## Method

- **Entry held constant** at market-on-next-bar-open (pt 17/18 baseline). This isolates the stop effect from the entry-mechanic (pt 19).
- **Stack**: revised `{3, 4, 5, 6, 6s, 7, 9}` with pt 17 hybrid rule 9 targets (H1/H2 → 5R, L1/L2 → 3R, shorts capped at 4R).
- **Resolver**: honest-pessimistic, no friction.
- **Five stop models**:
  - `db_stop` — DB-recorded stop_price (baseline; ~sig-bar lo/hi + buffer).
  - `sigbar` — raw signal-bar low (long) / high (short), no buffer.
  - `swing2` — min-of-last-2-bar-lows (long) / max-of-last-2-bar-highs (short), INCLUDING signal bar.
  - `swing3` — same, 3-bar lookback.
  - `swing5` — same, 5-bar lookback.

## 1. Headline

| Stop model | n | WR % | sumR | perR | DD | 95% CI perR | Δ median risk vs db |
|:-----------|--:|-----:|-----:|-----:|---:|:------------|---:|
| `db_stop` (baseline) | 86 | 66.3 | +136.13 | **+1.583** | −4.00 | [+1.13, +2.03] | 0.23% |
| `sigbar` (raw)       | 86 | 66.3 | +137.67 | +1.601 | −4.00 | [+1.15, +2.10] | 0.23% |
| `swing2`             | 98 | 74.5 | +136.11 | +1.389 | −4.00 | [+1.04, +1.76] | +54% wider |
| `swing3`             | 98 | 74.5 | +123.43 | +1.259 | −4.00 | [+0.94, +1.62] | +59% wider |
| `swing5`             | 101 | 75.2 | +105.19 | +1.041 | −4.84 | [+0.75, +1.33] | +74% wider |

`sigbar` == `db_stop` to inside any noise — the DB's buffer is trivially small. All five CIs overlap heavily; the point estimates trend **monotonically downward** as the stop widens. Sum-R stays roughly flat through `swing2` (larger population compensates for lower perR), but perR falls cleanly. Shortest stop wins; each step wider costs ~0.2R/trade.

## 2. Per-trade risk — how much wider is "swing"?

Median risk as a % of entry price:

- `db_stop` / `sigbar`: 0.23%
- `swing2`: 0.36%  (~1.6× wider)
- `swing3`: 0.37%  (~1.6× wider)
- `swing5`: 0.40%  (~1.7× wider)

Brooks' intuition is that swing stops should be "meaningfully wider" to survive the normal ATR-sized wiggle. The data confirms — a swing2 stop is on average 54–60% wider in absolute price, 56–74% wider in relative risk.

## 3. Paired comparison — same trade, different stop

Trades that fill under **both** `db_stop` and the variant (`db_stop` set intersected with variant set):

| Variant | n_common | perR_db | perR_variant | Δ | WR_db | WR_variant | better / tied / worse |
|:--------|---------:|--------:|-------------:|---:|------:|-----------:|----------------------:|
| `sigbar` | 86 | +1.583 | +1.601 | +0.02 | 66.3% | 66.3% | 5 / 79 / 2 |
| `swing2` | 86 | +1.583 | +1.388 | −0.20 | 66.3% | 70.9% | 7 / 48 / 31 |
| `swing3` | 86 | +1.583 | +1.266 | −0.32 | 66.3% | 70.9% | 7 / 42 / 37 |
| `swing5` | 86 | +1.583 | +1.116 | −0.47 | 66.3% | 74.4% | 11 / 29 / 46 |

On shared trades, **wider stops convert a few losers to winners (7–11 trades) but hurt many more (31–46 trades)** — typically by letting a loser run wider before stopping (larger R-denominator = smaller positive R on the same P/L) or by letting a late-stop wipe out a trade that had already stopped at sig-bar. Sig-bar stop is near-Pareto-optimal on shared trades. The WR gain (+4–8 pct pts) is real but the perR cost outweighs it.

## 4. By setup — the Brooks hypothesis fails on H2

Brooks specifically says H2/L2 benefit from swing-structure stops. The data says otherwise:

| setup | model | n | WR % | perR | DD | med risk% |
|:------|:------|--:|:----:|:----:|:--:|:---------:|
| H1 | db_stop | 44 | 65.9 | +1.697 | −3.00 | 0.23% |
| H1 | swing2  | 44 | 68.2 | +1.444 | −2.91 | 0.33% |
| **H2** | **db_stop** | **19** | **63.2** | **+1.670** | −3.00 | 0.23% |
| **H2** | **swing2**  | **19** | **63.2** | **+1.042** | −3.00 | 0.30% |
| **H2** | **swing5**  | **19** | **63.2** | **+0.834** | −2.24 | 0.35% |
| L1 | db_stop | 19 | 63.2 | +1.041 | −4.00 | 0.19% |
| **L1** | **swing2**  | **31** | **87.1** | **+1.415** | −3.00 | 0.66% |
| L2 | db_stop |  4 | 100  | +2.485 | +0.00 | 0.49% |
| L2 | swing2  |  4 | 100  | +2.222 | +0.00 | 0.67% |

Three cells carry the story:

- **H2 is the wrong direction.** Widening H2's stop cuts perR almost in half (+1.67 → +1.04 at swing2). H2 stop-outs at sig-bar lo are **signal**, not noise: when a H2 fails and trades through the sig-bar low, the pullback isn't done — letting it run wider just turns a −1R into a bigger loss on the same P/L curve. The Brooks lesson *"stop beyond the prior failed attempt"* does not survive in this dataset.
- **L1 is the outlier on the long side.** swing2 grows the population 19→31 (12 new trades that sig-bar's tighter stop filtered out via earlier stop-out reshuffling rule 9), and perR *rises* +1.04 → +1.42. But see §5 — this is almost entirely a short-side L1 effect.
- **L2 too small** (n=4) to signal either way. Every model scores all 4 trades as wins; only the magnitude changes (3R target cap.) No statistical weight.

## 5. By direction — the short-side exception

| direction | model | n | WR % | perR | DD |
|:----------|:------|--:|:----:|:----:|:--:|
| long  | db_stop | 63 | 65.1 | +1.689 | −4.00 |
| long  | swing2  | 63 | 66.7 | +1.323 | −4.10 |
| **short** | **db_stop** | **23** | **69.6** | **+1.292** | **−4.00** |
| **short** | **swing2**  | **35** | **88.6** | **+1.508** | **−3.00** |

**On shorts, swing2 dominates db_stop on every metric**:
- n 23 → 35 (+52% population via re-gating rule 9 at rescued early-day trades)
- WR 70% → 89% (+19 pct pts)
- perR +1.29 → +1.51 (+0.22R/trade)
- DD better (−3 vs −4)

This is the only cell where wider stops are a net improvement, and it flips the long/short perR ordering — under swing2, shorts are the *better* side of the book. This matches the L1 row above: most of SPT's short-side population is L1 (pt 11 §Q11 already flagged this).

## 6. Why does wider help less often than intuition suggests?

Wider stops trade stop-outs for `chart_end` resolutions, not for targets:

| model | target% | stop% | chart_end% |
|:------|--------:|------:|-----------:|
| db_stop | 23.3 | 32.6 | 44.2 |
| swing2  | 17.3 | 23.5 | 59.2 |
| swing3  | 12.2 | 23.5 | 64.3 |
| swing5  |  6.9 | 21.8 | 71.3 |

Stop-hit rate drops ~10 pct pts from db_stop to swing5 — that's the "wider stop survives the wiggle" effect Brooks teaches. But target-hit drops *faster* (23% → 7%) because with hybrid rule 9 the absolute price target is `risk × 3–5R`, and a wider risk pushes that target into the next session. Most of the "rescued" trades end up as chart_end scratches, not 3R/5R winners. Net R is slightly negative.

## 7. Chronological stability

No variant is regime-fragile — all five climb perR across thirds of the dataset (late-period S3 dominance is visible everywhere). Part-3 perR for all models is +1.6R to +2.8R/trade; part-1 perR is +0.8R to +1.0R. Stop choice doesn't interact with regime.

## 8. Interpretation

Three consistent points:

1. **Narrower is better on the average trade.** Across the full revised stack, sig-bar stop (or DB's equivalent) wins on perR and matches on DD. Each step wider costs ~0.2R/trade of expectancy.
2. **But the short-side / L1 cell is the exception.** swing2 on shorts gives +0.22R/trade and a meaningful WR boost (70% → 89%) at lower DD. If anything in this study becomes a PLAYBOOK candidate, it's a setup-conditional stop: "use signal-bar stop on H*, use swing2 stop on L1-short." But n=35 is thin, and CIs overlap with the all-stop-same-model baseline.
3. **The Brooks H2 swing-stop teaching does not replicate.** H2 specifically hurts from wider stops (−0.63R/trade at swing2). In this dataset, H2 stop-outs at sig-bar-low are trend-failure signal, not wiggle. Likely explanation: a 5-min SPT detection is a *tight* pullback by construction (rules 1, 2 require small pullback texture); a stop that's fired at the sig-bar low has already broken below the pullback structure, and widening it just lets the failed trade run further.

## 9. Recommendation

- **No PLAYBOOK change.** The `db_stop` / `sigbar` baseline is correct for the general stack. Keep signal-bar low/high ± tiny buffer as the mechanical stop.
- **Do NOT apply Brooks' H2/L2 swing-stop teaching blindly.** In this dataset it costs 0.5–0.8R/trade of expectancy. Brooks' teaching is from continuous tape-reading context, not a 5-min systematic scan with pre-filtered small-pullback texture.
- **Short-side swing2 is a candidate follow-up.** Before adopting, a rule-9-equivalent walk-forward and LOO-week on shorts-only would need to confirm the +0.22R/trade holds. Low priority — the overall book is long-dominated and shorts are only 27% of trades under the baseline. File under "could be worth 0.05–0.08R at the portfolio level if confirmed." Open as Q33.
- **Size expectations to the signal-bar stop.** Any execution model that uses Brooks-canonical stop orders at signal-bar extremes is already consistent with this study's `sigbar` row: perR +1.60R, DD −4.00R. Matches pt 19's `stop_2bar` +1.73R once rule-9 re-gating kicks in.

## 10. Caveats

- **Entry held constant at market-on-open**, not Brooks-canonical stop-entry (pt 19). Composing the two would require re-running pt 19's stop-entry × each stop-placement here. Effect should be roughly additive (entry mechanic is second-order per pt 19), but not audited.
- **No friction layered.** Add pt 18's friction grid to convert to live expectations.
- **Target is R-multiple, not absolute price.** If we fixed the absolute price target (Brooks actually does — "scale out at the prior swing high"), wider stops would look better on absolute P/L but worse on risk-adjusted. The R-multiple framing is the right one for a fixed-risk-per-trade portfolio.
- **Same-bar within-fill sequencing.** When a swing stop is wider, the chance the stop is hit within the fill bar itself drops — but the resolver doesn't model intrabar sequencing. Small effect.

## 11. Open items

- **Q33 (new)** — short-side swing2 walk-forward validation. The +0.22R/trade on n=35 is suggestive but needs the pt 14 / pt 17 treatment (LOO-week, LOO-symbol, bootstrap CI) before adopting a setup-conditional stop policy. Would bolt onto the revised stack as an optional rule 10.
- **Composed entry × stop grid.** pt 19's `stop_2bar` entry × this pt 20's stop placements. Likely low impact.
- **ATR-based stops.** Brooks' cash-market version is "2× recent bar ATR" for defensive stops; never tested here. Would require rolling ATR computation per ticker and probably deserves its own study.

## Related

- [[small-pullback-trend-PLAYBOOK]] — §9 recommendation stands unchanged; signal-bar stop is the baseline.
- [[small-pullback-trend-entry-mechanic-2026-04-19]] — pt 19 opened Q32 in §open-items.
- [[small-pullback-trend-execution-friction-2026-04-19]] — pt 18 baseline friction model stacks on top of this study's `sigbar` row.
- [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] — pt 11 first surfaced the short-side asymmetry; swing2 looks like a second lever there.
- [[small-pullback-trend-INDEX]] — research arc.

## Brooks source

- *Trading Price Action: Trends*, ch. 57 — H1/L1 pullback stops at sig-bar extreme (matches this study's recommendation).
- *Trading Price Action: Trends*, ch. 25 — "Protective stops on H2 / L2 go beyond the prior high/low of the preceding pullback attempt" (this is the teaching that does NOT replicate for tight 5-min SPTs).
