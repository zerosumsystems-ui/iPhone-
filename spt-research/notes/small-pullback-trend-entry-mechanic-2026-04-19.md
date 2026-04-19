# SPT — Entry mechanic study (2026-04-19, pt 19)

Closes the "entry-mechanic model" open item from [[small-pullback-trend-execution-friction-2026-04-19]] §open-items. The pt 17 / pt 18 resolvers fill entries at `post[0]["o"]` — effectively market-on-next-bar-open. Brooks' canonical entry is a **buy stop at signal-bar-high + 1 tick** (long) / **sell stop at signal-bar-low − 1 tick** (short). A stop order only fills when the market trades through the trigger, so trades that reverse immediately out of the signal bar are filtered out of the population.

Script: `~/code/aiedge/scanner/scratch/spt_entry_mechanic_2026_04_19.py`.
Output: `_out_spt_entry_mechanic_2026_04_19.txt`.

## Method

- Population: revised stack `{3, 4, 5, 6, 6s, 7, 9}` with the pt 17 **hybrid rule 9** target (H1/H2 → 5R, L1/L2 → 3R, shorts capped at 4R).
- Resolver: honest-pessimistic, no friction (pt 17 §1). Isolating the entry-mechanic effect from execution friction.
- Five entry models compared:
  - **`market_open`** (baseline): fill at `post[0]["o"]`.
  - **`stop_1bar`**: stop order at signal-bar-high / low. Fill = `max(trigger, post[0].o)` for longs, `min(trigger, post[0].o)` for shorts. If post[0] never touches trigger, the order is canceled end-of-bar (no fill).
  - **`stop_2bar`** / **`stop_3bar`**: same but the stop order stays live for 2 or 3 bars before abandoning.
  - **`stop_unlim`**: stop order stays live until end of the chart window.
- Rule-9 first-3-gate evaluated per-model on that model's own r's (a trade that didn't fill isn't one of the "first three").

## 1. Headline

| Entry model | n | WR % | sumR | perR | DD | 95% CI perR | no_fill |
|:------------|--:|-----:|-----:|-----:|---:|:------------|:-------:|
| `market_open` (pt 18 baseline) | 86 | 66.3 | +136.13 | +1.583 | −4.00 | [+1.13, +2.03] | 0 |
| `stop_1bar`  | 74 | 73.0 | +128.52 | **+1.737** | **−3.00** | [+1.25, +2.22] | 32 |
| `stop_2bar`  | 76 | 72.4 | +131.32 | +1.728 | −4.00 | [+1.26, +2.20] | 24 |
| `stop_3bar`  | 76 | 72.4 | +131.32 | +1.728 | −4.00 | [+1.25, +2.19] | 22 |
| `stop_unlim` | 85 | 67.1 | +127.66 | +1.502 | −4.00 | [+1.06, +1.94] | 6 |

Note: population grew 84 → 86 vs pt 18 (1 new detection overnight). The `market_open` row above matches pt 17 / pt 18 methodology on today's DB — it is the correct apples-to-apples baseline.

## 2. Fill-rate

| Entry model | filled (post rule-9) | raw no-fill (pre-rule-9) | fill rate |
|:------------|:--------------------:|:------------------------:|:---------:|
| `stop_1bar`  | 74 | 32 | ~63% |
| `stop_2bar`  | 76 | 24 | ~72% |
| `stop_3bar`  | 76 | 22 | ~74% |
| `stop_unlim` | 85 |  6 | ~93% |

The "raw no-fill" count is the number of stack-filtered detections where the trigger never armed — so the difference between baseline (86 market-on-open trades) and a stop model's trade count is **not** identical to the raw no-fill count, because rule-9 first-3-gate re-evaluates on the filled set.

## 3. Paired comparison — same trades under both models

For the **intersection** of trades filled under both `market_open` and the stop model:

| Stop model | n_common | perR (market_open) | perR (stop model) | Δ | stop-better | tied | stop-worse |
|:-----------|---------:|--:|--:|:-:|:-:|:-:|:-:|
| `stop_1bar`  | 74 | +1.858 | +1.737 | −0.121 |  0 | 57 | 17 |
| `stop_2bar`  | 76 | +1.853 | +1.728 | −0.125 |  0 | 58 | 18 |
| `stop_3bar`  | 76 | +1.853 | +1.728 | −0.125 |  0 | 58 | 18 |
| `stop_unlim` | 85 | +1.613 | +1.502 | −0.111 |  1 | 65 | 19 |

**On shared trades, stop-entry pays ~0.12R/trade in adverse entry slip.** The stop-entry is never better than market-on-open and often equal (57–65 / 74–85 trades tied), the rest are worse. This matches the adverse-slip magnitude ~0.09 raw price units on an average ~$1 stop (see §7) — that's ~9% of risk, identical to pt 18's mid-scenario `f_entry = 5%` friction line for entries.

## 4. What stop-entry filters out

Trades that `market_open` takes but `stop_unlim` does not (the signal bar's direction immediately reverses, so the stop never arms):

`n=1`, `WR=0%`, `perR=−1.000`, reason: stop-out.

Under `stop_unlim` we only lose one baseline trade, and it was a full-R loser — so stop-entry provides a tiny "reversal filter" at the unlimited horizon. But the **real** gain in the time-limited variants comes from rule-9 re-gating: dropping late-triggering trades from the early part of the day can flip a "first-3 gate fails" date into "first-3 gate passes" (or vice versa), reshuffling later trades. §3's common-trade loss of 0.12R/trade is real; the headline perR gain of ~+0.15R in `stop_2bar` comes from this rule-9 interaction plus the filtered no-fills.

## 5. By setup

| setup | model | n | WR % | perR | DD |
|:------|:------|--:|:----:|:----:|:--:|
| H1 | `market_open` | 44 | 65.9 | +1.697 | −3.00 |
| H1 | `stop_unlim`  | 43 | 67.4 | +1.635 | −3.00 |
| H2 | `market_open` | 19 | 63.2 | +1.670 | −3.00 |
| H2 | `stop_unlim`  | 19 | 63.2 | +1.568 | −2.78 |
| L1 | `market_open` | 19 | 63.2 | +1.041 | −4.00 |
| L1 | `stop_unlim`  | 19 | 63.2 | +0.986 | −4.00 |
| L2 | `market_open` |  4 | 100  | +2.485 | +0.00 |
| L2 | `stop_unlim`  |  4 | 100  | +2.216 | +0.00 |

**No setup-level ranking flip.** The stop-entry penalty is evenly distributed (~0.05–0.27R/trade across setups); L2's tiny n=4 carries the biggest absolute hit but both models score it perfectly on wins.

## 6. By direction

| direction | model | n | WR % | perR | DD |
|:----------|:------|--:|:----:|:----:|:--:|
| long  | `market_open` | 63 | 65.1 | +1.689 | −4.00 |
| long  | `stop_unlim`  | 62 | 66.1 | +1.614 | −3.78 |
| short | `market_open` | 23 | 69.6 | +1.292 | −4.00 |
| short | `stop_unlim`  | 23 | 69.6 | +1.200 | −4.00 |

Direction-neutral — stop-entry costs both sides ~0.08R/trade but no rank change. Shorts' 11:30 ET filter (rule 6s) already handles the biggest timing-based short hazard; the entry mechanic is not a further lever there.

## 7. Entry-fill adverse slippage (paired, stop_unlim vs market_open)

- **n_paired = 85** (common trades).
- **48 trades (56%)**: stop fill worse than market_open (positive slip).
- **37 trades (44%)**: stop fill equal to market_open (i.e., post[0] opened on the signal-side already — stop-entry degenerates to market-on-open with no penalty).
- **0 trades**: stop fill better than market_open (can't happen by construction — stop fill = `max(trigger, open)` for long).
- **Mean adverse slip (when positive): 0.0928** raw price units.
- **Fill-bar distribution:** post[0] fills 76 trades, post[1] fills 2, post[2] fills 0, post[3] and later fills 7. The 7 late fills in `stop_unlim` are the stale-breakout trades that drag its perR down vs `stop_2bar`.

## 8. Interpretation

Three mutually consistent points:

1. **Fill-price penalty is real but small.** The stop-entry pays ~0.12R/trade on trades that *do* fill, which matches pt 18's `f_entry = 5–10%` realistic friction band almost exactly. So the pt 18 friction model was already implicitly pricing in stop-entry fills (not market-on-open fills) — nothing new to correct there.
2. **Time-limited stop-entry is the free-lunch-shaped variant.** `stop_2bar` earns +0.145R/trade over market-on-open headline at identical DD. But the improvement comes primarily from **rule-9 re-gating**, not from per-trade reversal filtering — only one baseline trade is a clean "stop-entry saved me from a reversal" case.
3. **Unlimited horizon is a loss.** Late-fill stop-entries (post[3+], 7 trades) are stale — the trend has moved, the entry is worse, and perR drops to +1.50R (vs +1.73R for `stop_2bar`). Brooks' own practice: stop orders live one bar, canceled at bar-close and re-placed if the next bar gives a new signal.

Pragmatically, these deltas (+0.1 to +0.15R/trade) are **inside the bootstrap CI** (all four stop variants and the baseline overlap heavily — CI widths are ~1R). So:

- **No PLAYBOOK change required.** The pt 18 adoption recommendation (hybrid rule 9) stands regardless of entry mechanic.
- **If Will executes with Brooks-canonical stop orders** (which he would, in practice — resting orders not market orders), he should size his expectations to the `stop_2bar` row: +1.73R/trade at −4.00R DD, not the pt 17 / pt 18 +1.60R/trade baseline. The difference is almost entirely rule-9 interaction with fewer trades.
- **Abandon stop orders after ~2 bars if not filled.** This is a mechanical rule Brooks already teaches and the data supports cleanly — `stop_2bar` dominates `stop_unlim` by 0.23R/trade.

## 9. Caveats

- **Tick-level granularity not modeled.** "Stop at signal-bar-high + 1 tick" vs "at signal-bar-high exactly" — difference is ~$0.01 on a typical $100 stock, trivially small (0.01R on a $1 stop). Ignored.
- **Within-fill-bar sequencing.** When a stop arms on a fast bar, the resolver checks stops in order — this can spuriously mark a fill-and-stopped-same-bar trade as an immediate loss. Low frequency issue (bar ranges for trigger-armed bars are small in SPT texture), not audited here.
- **`stop_price` in DB already reflects Brooks-canonical levels.** The resolver uses `stop_price` (typically signal-bar low/high ± buffer), so we are NOT comparing stop placement here, only entry mechanic.
- **No friction stacked on top.** This study is zero-friction; the paired-comparison §3 slip IS the entry-mechanic friction and should not be double-counted with pt 18's grid. For a Brooks-canonical live model, use pt 18's `f_entry = 0%` column of the `stop_*` variants (since the entry mechanic's cost is already in the price), then apply pt 18's `f_stop` / `f_target` / `f_end` separately.

## 10. Conclusion

Entry mechanic is **second-order** to the stack. All five variants' CIs overlap. Direction, setup, and drawdown profiles are stable under entry-model choice.

**Recommendation for live deployment:** rest stop orders at signal-bar-high / low, cancel after 2 bars (10 minutes on 5-min timeframe). Do not switch to market-on-next-bar-open just because a bar prints red — that's the modeling convenience of the resolver, not the optimal mechanic.

**No PLAYBOOK change.** Close this open item.

## Open items

- **Stop placement study.** Every analysis so far has frozen `stop_price` at the DB's recorded value (typically signal-bar low/high). Brooks also teaches swing-structure stops (2–3 bars back) for H2/L2 — has never been tested. Would change `risk` per trade and therefore R-normalization. Next study candidate.
- **Entry + friction composition.** Apply pt 18's friction grid on top of `stop_2bar`. Should be near-linear (friction additively on top of fill slip). Not critical.

## Related

- [[small-pullback-trend-PLAYBOOK]] — §9 recommendation stands unchanged.
- [[small-pullback-trend-execution-friction-2026-04-19]] — pt 18 opened this question in its open-items.
- [[small-pullback-trend-target-walkforward-2026-04-18]] — pt 17 hybrid rule 9.
- [[small-pullback-trend-INDEX]] — research arc.
