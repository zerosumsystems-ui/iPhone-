# SPT per-timeframe recommendations — 2026-04-19 (scheduled run)

Consolidated action sheet. One-glance per-timeframe picks with the 11-rule PLAYBOOK entry gates layered on. No new data — synthesis of [pt 33 multi-TF scan](small-pullback-trend-multi-tf-candidates-2026-04-19.md) + [Monday watchlist](small-pullback-trend-monday-watchlist-2026-04-20.md).

Data anchor: Fri 2026-04-17 cash close. Universe: 52 liquid US single-names + big-4 index ETFs.

## Top picks by timeframe

| TF | 🟢 Long | 🔴 Short | Cross-TF rank |
|:-:|---|---|---|
| **Daily (weeks)** | **AMD** (+6.65 netR, +36.9%/20d, close 97% of range) — best mature SPT on weeks | **CVX** (−2.84 netR, −8.9%/20d) — energy is the one weak sector | — |
| **60m (1–3 sessions)** | **MRK** (+6.56, close 97%) — healthcare leg, part of 4-TF cross | **NFLX** (−6.53, −10.08%) — loudest single-name bear | MRK ★★★★ · NFLX single |
| **30m (hours)** | **MRK** (+9.84, close 96%) — cleanest 30m uptrend in scan | **ADBE** (−4.24, close 93%) — cleanest software short | MRK ★★★★ · ADBE ★★★ |
| **15m (tens of min)** | **WMT** (+8.87, close 98%) | **ORCL** (−7.05, close 85%) | WMT ★★ · ORCL ★★★ |
| **5m (scalp)** | **WMT** (+18.69 netR) — single strongest trend reading in entire 52×5 scan | **GE** (−10.42) — industrial momentum exhaustion | WMT ★★ · GE ★★ |

## Cross-timeframe conviction stars (signal on ≥ 2 TFs same direction)

**🟢 LONGS** — defensive/healthcare rotation

| Symbol | Last | TFs up | Rank |
|---|---:|---|:-:|
| **MRK** | $119.07 | 60m · 30m · 15m · 5m | ★★★★ |
| **DIS** | $106.28 | 60m · 30m · 15m | ★★★ |
| **HD** | $349.40 | 60m · 30m | ★★ |
| **DIA** | $494.21 | 60m · 30m | ★★ |
| **UNH** | $324.50 | 30m · 5m | ★★ |
| **WMT** | $127.47 | 15m · 5m | ★★ |
| **COST** | $999.63 | 15m · 5m | ★★ |

**🔴 SHORTS** — software-complex + select momentum

| Symbol | Last | TFs down | Rank |
|---|---:|---|:-:|
| **ORCL** | $175.06 | 30m · 15m · 5m | ★★★ |
| **ADBE** | $244.38 | 30m · 15m · 5m | ★★★ |
| **CRM** | $182.17 | 15m · 5m | ★★ |
| **GE** | $304.12 | 15m · 5m | ★★ |

## Entry frame per recommendation (PLAYBOOK applied)

Monday 2026-04-20 entry windows with C3 stack (9 core + rules 10 & 11):

| Horizon | Dir | Pick | Entry window (ET) | Stop | Target |
|---|:-:|---|---|---|:-:|
| Position / weeks | 🟢 | AMD | Daily H1/H2 on next pullback (no intraday window) | Swing-low of pullback day | 5R |
| Intraday swing | 🟢 | MRK (60m) | 10:30–14:15 · urgency ≥ 4 · `always_in ≠ opposed` | Signal-bar low | 5R |
| Intraday hours | 🟢 | MRK (30m) | 10:30–14:15 · gap-up aligned | Signal-bar low | 5R |
| Intraday scalp | 🟢 | WMT (5m) | 10:30–14:15 · urgency ≥ 4 · `opp_tail < 0.25` | Signal-bar low | 5R |
| Intraday short | 🔴 | ADBE | **11:30–14:15** (short-only window) · urgency ≥ 4 | Max of last-2-bar highs (rule 10) | 4R |
| Swing short | 🔴 | CVX | Daily L1/L2 on a bounce | Swing-high of bounce | 4R |

## Do-not-fade divergences

- **AMZN** — daily LONG (+21.1%/20d) but 15m SHORT. Treat Monday weakness as a pullback in the daily long, not a short entry.
- **CVX** — daily SHORT but 5m LONG (+9.07). Treat Monday strength as a dead-cat bounce, not a long entry.

## Mandatory gate checklist (every fire, every TF)

```
[ ] setup ∈ {H1, H2, L1, L2}
[ ] day_type ∈ {trend_from_open, spike_and_channel}
[ ] urgency ≥ 4  (≥ 6 on trend_from_open)
[ ] gap_direction aligned with setup
[ ] always_in ≠ opposed(setup)
[ ] time ∈ [10:30, 14:15) ET  (shorts: [11:30, 14:15))
[ ] not 1st ticker-day detection (rule 7)
[ ] if ≥ 3 SPT trades already today → ≥ 2 won (rule 8)
[ ] signal-bar opp_tail < 0.25 (rule 11)
```

Expected economics if clean fires present (C3 stack, 8-mo backtest):
**n ≈ 18 trades/month · WR 74.6% · +1.84R/trade · max DD −2R**

## One-line narrative

Broad tape is bull (tech/financials on daily), but under the hood the bid has rotated **into defensives** (MRK, DIS, HD, UNH, WMT, COST) and **out of enterprise software + select momentum** (ORCL, ADBE, CRM, GE) in the last hours of Friday's cash.

## Caveats

- Last tick Fri 16:00 ET — ~64h of news exposure before Monday open.
- Live aiedge-scanner is canonical for execution; this is a pre-Monday ranking overlay.
- Universe is 52 hand-picked names — any cleaner SPT outside the list (live scanner pickup) can outrank.
- Pt 33 scan is pure magnitude (no `day_type` / urgency layer); PLAYBOOK gates are applied at entry, not in the scan.

## Related

- [[small-pullback-trend-multi-tf-candidates-2026-04-19]] — source ranking (pt 33)
- [[small-pullback-trend-monday-watchlist-2026-04-20]] — full PLAYBOOK overlay
- [[small-pullback-trend-PLAYBOOK]] — 11-rule operational policy
- [[small-pullback-trend-INDEX]] — 33-note research arc
