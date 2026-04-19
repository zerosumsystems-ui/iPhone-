# SPT Monday-open watchlist — 2026-04-20

**Operational companion to [[small-pullback-trend-multi-tf-candidates-2026-04-19|pt 33]].** Pt 33 ranked the strongest multi-timeframe trends by pure magnitude. This note layers the 11-rule [[small-pullback-trend-PLAYBOOK|PLAYBOOK]] onto those picks so Monday open has a ready watchlist with entry windows, stops, targets, and invalidation lines. No new data or research — a consumption-layer pass on pt 33.

Data anchor: Fri 2026-04-17 cash close. Entry-gate rules from current PLAYBOOK (C3 stack: 9 core rules + conditional rules 10 & 11). Expected economics if clean fires present: **n ≈ 18 trades/month · WR 74.6% · +1.84R/trade · max DD −2R**.

## The one-table watchlist

Top pick per horizon with pt 33's magnitude score + PLAYBOOK-enforced entry frame.

| Horizon | Dir | Name | Last | Entry window (ET) | Signal required | 1R stop zone | Target (hybrid rule 9) | Invalidation |
|---|:---:|---|---:|---|---|---|:---:|---|
| Position (weeks) | 🟢 | **AMD** | $277.25 | — (not intraday) | Daily H1/H2 on next pullback | Swing-low of pullback day | 5R | Break of Fri low $269.xx; close < 20-MA |
| Intraday swing | 🟢 | **MRK** | $119.07 | 10:30 – 14:15 | H1/H2 on 30m or 60m · urgency ≥ 4 · `always_in ≠ opposed` | Signal-bar low | 5R | Close < prior-day low $117.xx |
| Hours | 🟢 | **MRK (30m)** | $119.07 | 10:30 – 14:15 | H1/H2 with gap-up aligned | Signal-bar low | 5R | 30m close < $118.20 |
| Scalp | 🟢 | **WMT (5m)** | $127.47 | 10:30 – 14:15 | H1/H2 · urgency ≥ 4 · `opp_tail < 0.25` | Signal-bar low | 5R (or first 5R touch) | 5m close < $126.80 |
| Intraday short | 🔴 | **ADBE** | $244.38 | **11:30 – 14:15** (short-only window) | L1/L2 · urgency ≥ 4 · swing-2 stop | Max of last-2-bar highs | 4R (H2-short cap) | 30m close > $247.50 |
| Swing short | 🔴 | **CVX** | $183.85 | — | Daily L1/L2 on a bounce | Swing-high of bounce | 4R | Daily close > 20-MA (~$188) |

**Reminders from PLAYBOOK:**
- Rule 7 — skip the 1st ticker-day detection.
- Rule 8 — after the day's first 3 SPT trades, continue only if ≥ 2 won.
- Rule 11 — reject any signal bar with `opp_tail ≥ 0.25` (counter-trend wick).
- Rule 10 — **shorts only:** stop = max of last 2 bar highs incl. signal bar. Longs keep signal-bar stop.

## Cross-TF stars — maximum-alignment watch

These names are in SPT on ≥ 2 timeframes in pt 33; a fresh H1/H2 or L1/L2 fire on Monday maps cleanly to continuation.

**🟢 LONGS (cross-TF)**

| Symbol | TFs up | Monday priority | What to watch |
|---|---|:---:|---|
| **MRK** | 60m · 30m · 15m · 5m | ★★★★ | Highest conviction in the entire scan. Any clean H1/H2 10:30 – 14:15 is a take. |
| **DIS** | 60m · 30m · 15m | ★★★ | $106.28; break of Fri high with urgency ≥ 6 = TFO-grade fire. |
| **HD** | 60m · 30m | ★★ | $349.40; retail-adjacent defensive bid confirmation. |
| **DIA** | 60m · 30m | ★★ | Index ETF — if DIA fires, single-name longs get a breadth tailwind (rule 8 confirmation). |
| **UNH** | 30m · 5m | ★★ | $324.50; healthcare rotation leg. |
| **WMT** | 15m · 5m | ★★ | $127.47; Friday was near-vertical — watch for rule-11 `opp_tail` on any pullback (Q2 volume trap risk). |
| **COST** | 15m · 5m | ★★ | $999.63; three-digit round number is a natural magnet — don't chase above $1005. |

**🔴 SHORTS (cross-TF)**

| Symbol | TFs down | Monday priority | What to watch |
|---|---|:---:|---|
| **ORCL** | 30m · 15m · 5m | ★★★ | $175.06; software-complex leg; L1/L2 in short window 11:30 – 14:15 ET. |
| **ADBE** | 30m · 15m · 5m | ★★★ | $244.38; cleanest short. Apply rule 10 (swing-2 stop). |
| **CRM** | 15m · 5m | ★★ | $182.17; third software-complex leg — if all three continue, regime read confirms (opens Q48 live). |
| **GE** | 15m · 5m | ★★ | $304.12; industrial/momentum exhaustion. |

## Divergences — do NOT fade the larger timeframe

Pt 33 flagged two counter-trend intraday bounces. These are fade-candidates in the direction of the higher-timeframe trend, not entry signals in the direction of the bounce.

- **AMZN** — daily LONG (+21.1% 20d), 15m SHORT (−4.44 netR). Treat Monday weakness as a pullback in the daily long, not a short entry. Watch for H1/H2 re-engagement on 30m/60m.
- **CVX** — daily SHORT (−8.9%), 5m LONG (+9.07 netR). Treat Monday strength as a bounce in the daily short, not a long entry. Watch for L1/L2 on 30m/60m once the bounce exhausts.

## Gap-risk checklist for open

Last price tick is Fri 16:00 ET — ~64 hours of news exposure. Before acting on any pt 33 pick at Monday open:

1. Scan overnight headlines on the cross-TF stars (MRK, ADBE, ORCL) for any earnings pre-announce / upgrade-downgrade.
2. If an opening gap > 1% in the trend direction: **wait for PLAYBOOK rule 6 window (10:30 ET+)** — the first-hour gap-fade is outside the research-validated SPT entry band.
3. If an opening gap AGAINST the trend direction: pt 33's ranking is provisional; re-confirm after the 10:30 ET bar closes that urgency is still ≥ 4 on 30m/60m.
4. If `day_type` on the live scanner flips to `trading_range` / `tight_tr` after the open — SPT entries are rule-2 blocked until it flips back to `trend_from_open` or `spike_and_channel`.

## Scoring quick-reference

For each candidate before pulling the trigger:

```
PLAYBOOK GATES (must all be true):
 [ ] setup ∈ {H1, H2, L1, L2}
 [ ] day_type ∈ {trend_from_open, spike_and_channel}
 [ ] urgency ≥ 4 (≥ 6 on trend_from_open)
 [ ] gap_direction aligned with setup
 [ ] always_in ≠ opposed(setup)
 [ ] time ∈ [10:30, 14:15) ET  (shorts: [11:30, 14:15))
 [ ] not the 1st ticker-day detection (skip rule 7)
 [ ] if ≥ 3 SPT trades already today → ≥ 2 won (rule 8)
 [ ] signal-bar opp_tail < 0.25   (rule 11)

STOPS:
 Longs  → signal-bar low
 Shorts → max of last-2-bar highs incl. signal bar (rule 10)

TARGETS (hybrid rule 9):
 H1 long  → 5R
 H2 long  → 5R
 L1 short → 4R (was 3R; watch for pt 21 re-measure at n_shorts≥80)
 L2 short → 4R
 H1 short → 5R
 H2 short → 4R
```

## Caveats

- This note does **not** re-run the scan. It's a consumption-layer overlay on pt 33's Fri 4/17 snapshot. If Monday opens with news flow or a regime break, the watchlist degrades fast — the live aiedge-scanner is canonical for intraday execution.
- Cross-TF stars are higher-conviction candidates, **not** entry triggers. Entry is still the 11-rule PLAYBOOK on the live detection, same as a non-cross-TF fire.
- Pt 33 caveat stands: universe is 52 hand-picked names. A cleaner SPT outside this list (e.g. active in the live scanner) can outrank anything here.

## Related

- [[small-pullback-trend-multi-tf-candidates-2026-04-19]] — the source ranking (pt 33).
- [[small-pullback-trend-PLAYBOOK]] — the 11-rule stack.
- [[small-pullback-trend-INDEX]] — full 33-note research arc.
