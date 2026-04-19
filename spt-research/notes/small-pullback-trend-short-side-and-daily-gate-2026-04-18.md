# SPT — Short-Side Deep Dive + Daily-Confirmation Gate (2026-04-18, pt 11)

Twelfth follow-up to [[small-pullback-trend]]. Resolves two of the remaining open items from the series:

- **Q11** (short-side directional asymmetry, OPEN since pt 5) — is the long/short gap a regime artifact or structural, and is any subset of short-side viable?
- **Q21** (max-trades-per-day cap, OPEN since pt 10) — 2026-04-14 blew up at 23 trades; is there a filter that rejects that day without rejecting the other heavy days?

Script: `~/code/aiedge/scanner/scratch/spt_short_side_and_heavy_days_2026_04_18.py`.
Population (refreshed): n=137 after the full §7 policy stack. Slightly down from the n=153 in [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] — some detections reclassified or un-resolved between runs. Dataset spans 2025-12-16 → 2026-04-17.

## 0. Refreshed baseline

| | n | WR% | sum R | per trade |
|---|--:|----:|------:|----------:|
| Policy stack rules 1-7 (full) | 137 | 59.1 | +157.66 | +1.151R |
| Longs only | 87 | 62.1 | +113.63 | **+1.306R** |
| Shorts only | 50 | 54.0 | +44.03 | **+0.881R** |

Short-side lifted substantially from the pt 5 reading (32–35% WR, +0.20 to +0.27R). The pt 5 snapshot was at urgency ≥ 4 only, before the `always_in`, `gap_direction`, time-of-day, skip-1st, and mkt_ord filters were layered in. At the full stack, shorts are positive but ~0.43R/trade behind longs — still a real gap, but not the "suppress entirely" zone pt 5 implied.

## 1. Short-side localization — Q11 CLOSED

### 1.a Time-of-day (30-min buckets, shorts only)

| window | n | WR% | per trade |
|--------|--:|----:|---------:|
| 10:30–11:00 | 3 | 33.3 | −0.63R |
| **11:00–11:30** | **9** | **0.0** | **−1.00R** |
| 11:30–12:00 | 14 | 64.3 | +1.16R |
| 12:00–12:30 | 16 | 62.5 | +1.41R |
| 13:00–13:30 | 5 | 80.0 | +1.41R |
| 13:30–14:00 | 3 | 100.0 | +3.00R |

**The 11:00–11:30 ET bucket is a pure short-side trap: 0/9 trades, −9R.** Every single short entry taken in this window stopped. Brooks' text on counter-trend temptation at the "European close / US mid-morning" maps here — it's when bears think the early trend is exhausted and gets squeezed.

Shorts from 11:30 ET onward: n=38, **68.4% WR, +1.44R/trade** — on par with long-side economics.

### 1.b Ticker-day ordinal (shorts only)

| ordinal | n | WR% | per trade |
|--------:|--:|----:|---------:|
| 1 | 29 | 44.8 | +0.55R |
| 2 | 13 | 61.5 | +1.24R |
| 3+ | 4 | 75.0 | +1.51R |
| **skip-1st (2+)** | **21** | **66.7** | **+1.34R** |

Skip-1st works on shorts identically to how it works across the board (pt 10). The 1st short on a ticker-day is Brooks' "loud bar at the climax" — 45% WR.

### 1.c Market-wide ordinal (shorts only)

| mkt_ord | n | WR% | per trade |
|---------|--:|----:|---------:|
| 1–2 | 30 | 43.3 | +0.63R |
| 3–5 | 19 | 68.4 | +1.16R |
| 6+ | 1 | 100.0 | +3.00R |

Mkt_ord > 2 on shorts: n=20, 70% WR, +1.25R/trade. Confirms the pt 10 finding generalizes to shorts.

### 1.d Stacked filters on short-side only

| rule | n | WR% | per trade |
|------|--:|----:|---------:|
| raw shorts (stack 1–7) | 50 | 54.0 | +0.88R |
| + u ≥ 6 | 37 | 56.8 | +0.93R |
| + skip-1st | 21 | 66.7 | +1.34R |
| + mkt_ord > 2 | 20 | 70.0 | +1.25R |
| + 11:30–14:15 ET | 38 | 68.4 | +1.44R |
| + skip-1st + mkt_ord > 2 | 14 | 71.4 | +1.37R |
| **+ skip-1st + mkt_ord > 2 + 11:30+** | **11** | **90.9** | **+2.02R** |

Shorts with all three cross-cutting filters layered (skip-1st, mkt_ord > 2, entry after 11:30 ET) print **91% WR, +2.02R/trade** (n=11). Small sample but the uplift is consistent with the long-side after the same filter stack.

**Resolution**: The short-side gap **was a time-of-day artifact concentrated in the 10:30–11:30 window**, not a structural regime bias. Four-week data extension lifted the floor on short WR by removing the "bull-regime only" sampling concern. Shorts are viable when entered 11:30 ET or later with skip-1st and mkt_ord > 2. The opening-hour-short suppression flagged in [[small-pullback-trend-entry-side-filters-2026-04-18]] §c was correct in direction but should be hardened: do not short an SPT in the 10:30–11:30 ET window.

## 2. Daily-confirmation gate — Q21 CLOSED (actionable)

The heavy-day population has changed shape since pt 10. We now have FOUR days with n ≥ 8 in the policy-stack:

| date | n | WR% | sum R | per trade | tickers | L/S split |
|------|--:|----:|------:|---------:|--------:|----------:|
| 2026-04-14 | 23 | 34.8 | −2.66 | −0.12R | **17** | 20/3 |
| 2026-04-16 | 17 | 88.2 | +43.00 | **+2.53R** | 4 | 17/0 |
| 2026-04-15 | 7 | 100.0 | +21.00 | +3.00R | 3 | 7/0 |
| 2026-01-21 | 6 | 66.7 | +7.00 | +1.17R | 2 | 6/0 |

2026-04-14 is still a clear outlier but now on ONE clean axis: **ticker diversity**. 17 unique tickers / 23 trades = 0.74 ratio. Compare 2026-04-16's 4 tickers / 17 trades = 0.24 ratio. The heavy-good days are a single ticker (or small cluster) firing repeatedly as the SPT develops; the heavy-bad day is every gappy name in the market screening simultaneously.

### 2.a Ticker-diversity forward-lookahead filter

| rule | n | WR% | sum R | per trade |
|------|--:|----:|------:|---------:|
| baseline | 137 | 59.1 | +157.66 | +1.15R |
| drop days where total tickers > 5 | 114 | 64.0 | +160.32 | **+1.41R** |

Dropping 2026-04-14's 23 trades costs $2.66$R and gains $+$0.26R/trade across the remaining 114. **BUT this filter requires day-end lookahead** — you don't know the day's total ticker count when entering at 10:35 ET. Useful as a backtest attribution, not a live rule.

### 2.b First-3-trades-of-day as live proxy

Under the hypothesis that "the day tells you what it is by the 3rd trade," group rest-of-day by the number of wins in the first 3 policy-stack trades:

| first-3 wins | days | rest-of-day n | rest WR% | rest per trade |
|:------------:|-----:|--------------:|---------:|--------------:|
| 0 | 2 | 4 | 50.0 | +1.00R |
| 1 | 4 | 23 | 43.5 | +0.27R |
| 2 | 3 | 5 | 60.0 | +0.42R |
| **3** | **6** | **23** | **91.3** | **+2.47R** |

**The signal is in the 3/3 bucket.** Days where the first 3 policy-stack trades all win produce rest-of-day that's +2.47R/trade on 91% WR. Days with 1/3 wins produce mediocre rest-of-day (+0.27R/trade on 44% WR — the 2026-04-14 regime).

### 2.c Continuation-gate rules

Applied to full policy stack: take all first-3 trades, then continue only if N of the first 3 were winners.

| continuation rule | n | WR% | sum R | per trade |
|------|--:|----:|------:|---------:|
| baseline (take all) | 137 | 59.1 | +157.66 | +1.15R |
| continue iff first3 ≥ 1 win | 133 | 59.4 | +153.66 | +1.16R |
| **continue iff first3 ≥ 2 wins** | **110** | **62.7** | **+147.40** | **+1.34R** |
| continue iff first3 ≥ 3 wins | 105 | 62.9 | +145.29 | +1.38R |

`≥ 2` vs `≥ 3` is a small selectivity trade-off. `≥ 2` keeps more trades with minimal per-R cost. Either works.

Cost of the gate: −10R total vs baseline, but from 137 → 110 trades (20% fewer). Per-R rises +0.19R/trade. Matches the selectivity pattern from mkt_ord > 2 and skip-1st.

### 2.d Compound with existing filters

| rule | n | WR% | sum R | per trade |
|------|--:|----:|------:|---------:|
| first3 ≥ 2 | 110 | 62.7 | +147.40 | +1.34R |
| first3 ≥ 2 + skip-1st | 48 | 77.1 | +91.06 | +1.90R |
| first3 ≥ 2 + mkt_ord > 2 | 44 | 79.5 | +81.04 | +1.84R |
| **first3 ≥ 2 + skip-1st + mkt_ord > 2** | **33** | **81.8** | **+66.06** | **+2.00R** |
| first3 ≥ 3 + skip-1st + mkt_ord > 2 | 28 | 85.7 | +63.95 | +2.28R |

The three selectivity gates (first3, skip-1st, mkt_ord) are largely additive. Stacking all three gives 82–86% WR and +2.00–2.28R/trade on 28–33 trades over the window.

### 2.e Why first-3-gate beats hard-cap

The hard-cap rule "take first N trades per day only" (§Q21.e) is *worse* than no cap:

| hard cap | n | per trade |
|:--------:|--:|---------:|
| first 3 | 82 | +1.08R |
| first 5 | 103 | +1.15R |
| first 10 | 117 | +1.20R |
| no cap | 137 | +1.15R |

Caps discard winners on good days (where many trades continue to hit) while doing nothing about bad days (the first 3 already lost). The first-3-gate does the opposite: keeps trades on good days, cuts them on bad days. That's the structure of the signal.

## 3. Updated one-page policy stack

Over [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] §6:

1. Setup ∈ {H1, H2, L1, L2}
2. day_type ∈ {trend_from_open, spike_and_channel}
3. urgency ≥ 4 (≥ 6 on trend_from_open)
4. gap_direction aligned with setup
5. always_in != opposed(setup)
6. signal time ∈ [10:30 ET, 14:15 ET) — with **hard exclusion 10:30–11:30 ET on shorts** (new, this note §1)
7. skip the 1st ticker-day detection
8. market_ord > 2
9. **NEW — after the day's first 3 policy-stack trades, continue only if ≥ 2 of them won** (this note §2)
10. Exit: 3R target / 1R stop / hold to resolution

**Projected expectancy, refreshed (n=137 → n=33 with full stack):**

- Policy stack 1–8 only (pt 10): +1.156R/trade, n=153 → +176.85R / 3.5 months (prior snapshot)
- Policy stack 1–8 only (refreshed): +1.151R/trade, n=137 → +157.66R / 3.5 months (refresh reshuffled some trades)
- Add rule 9 (first3 ≥ 2 wins): +1.340R/trade, n=110 → +147.40R
- Add rules 9 + short-side ToD: +1.35–1.40R/trade (modest lift from removing 9 dead-zone shorts)
- **Full refreshed stack (pt 11 compound): +2.00R/trade, n=33 → +66.06R**

Throughput trade-off is significant. Will's choice: high-confidence mode (full stack, ~33 trades/3mo at +2R/trade) vs higher-volume mode (rules 1–9 only, ~110 trades/3mo at +1.34R/trade). Total R slightly favors the 1–9 stack; per-trade economics and WR (82% vs 63%) favor the full stack.

## 4. What this resolves / opens

- **Q11 (short-side asymmetry)** — CLOSED. Not a regime bias: a time-of-day effect concentrated in 10:30–11:30 ET. The 11:00–11:30 ET short bucket is 0/9. Rule 6 addendum: no short entries before 11:30 ET. Shorts post-11:30 run at long-side economics after the full filter stack.
- **Q21 (max-trades-per-day cap)** — CLOSED in favor of a *different* rule shape. The "cap first N trades" framing is wrong — hard caps cost R on good days. The correct instrument is the **first-3-wins continuation gate** (take first 3, continue iff ≥ 2 won). This cleanly rejects 2026-04-14 (had 1/3 first wins) while preserving 2026-04-15/16 (had 3/3). Ticker-diversity filter works in backtest but needs day-end lookahead — not live.
- **NEW (Q22) — minimum-n for first-3-gate reliability** (OPEN, low-priority). The 3/3 bucket shows +2.47R/trade on 6 days. Small days (n < 4) are passed through ungated by design, but if a ticker-day only has 2–3 trades in total, the gate can't fire — they all get taken anyway. Worth monitoring whether very-low-n days drift.

## 5. What surprised me

The short-side gap was almost entirely a single 30-minute window. I expected a diffuse 20pp WR spread across all short setups driven by regime bias; the actual pattern is 9/9 losses in one bucket and parity everywhere else. This is consistent with Brooks' specific warning about counter-trend entries "shortly before the European close / US mid-morning" — bears think the morning trend is exhausted around 11:00 ET, step in, and get squeezed by the 12:00–13:00 ET continuation. The scanner catches the squeeze, not the reversal.

The first-3-wins gate producing cleaner separation than ticker count is the other surprise. Ticker diversity is the retrospective pattern (2026-04-14 = 17 tickers); first-3-wins is the live-tradable pattern. Both encode the same "is this one real SPT name or a breadth fire?" question, but only one is actionable pre-3rd-trade.

## Related

- [[small-pullback-trend]] — parent. Adds rule 9 (daily-confirmation gate) and short-side ToD carve-out to rule 6. Closes Q11 and Q21.
- [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] — pt 10, immediate predecessor. Baseline for the refreshed policy stack; rules 7–8 carry forward.
- [[small-pullback-trend-alignment-filters-2026-04-18]] — pt 5, where Q11 was first opened. Short-side gap was +0.88R vs +1.31R longs on the refresh — narrower than pt 5's read but still real.
- [[small-pullback-trend-entry-side-filters-2026-04-18]] — pt 7, proposed tightening short entries to 11:30–14:15 ET. This note hardens to 11:30–14:15 ET (exclusion before 11:30).
- Script: `~/code/aiedge/scanner/scratch/spt_short_side_and_heavy_days_2026_04_18.py`.
- Brooks, *Trading Price Action: Trends*, ch. 57 — counter-trend setup squeeze around 11:00 ET / European close.
