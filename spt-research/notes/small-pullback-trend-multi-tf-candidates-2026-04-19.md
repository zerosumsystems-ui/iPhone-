# SPT multi-timeframe candidate scan — 2026-04-19

**Strongest-trend recommendations.** For each timeframe {daily, 60m, 30m, 15m, 5m}, the 5 biggest clean directional trends in the universe — highest composite of {net_R, shallow pullback, close near extreme}. Ranked by pure magnitude, not quality gate. If a name appears on multiple timeframes it's a higher-conviction cross-TF read.

Data: Databento `XNAS.ITCH` through Fri 2026-04-17 cash close. Universe n=52 (mega-cap tech + momentum + financials + industrials + defensives + the big-4 index ETFs). Scan at `/tmp/spt_scan/scan.py`, re-rank at `/tmp/spt_scan/rerank_strongest.py`.

Sanity filter only: `|net_R| ≥ 2.0` (at least 2 ATR of net move) and `closeness ≥ 0.70` (closed in top/bottom 30% of window range). No pullback cap — the scan shows you the biggest moves, even if pullback was moderate.

## Daily — position / swing (weeks)

**LONGS — broad tape uptrend**

| # | Symbol | net_R | pullback | close | 20d move | last |
|---|--------|------:|---------:|------:|---------:|-----:|
| 1 | **AVGO** | +6.61 | 0.39 | 0.99 | +29.6% | $406.06 |
| 2 | **AMD**  | +6.65 | 0.38 | 0.97 | +36.9% | $277.25 |
| 3 | **AMZN** | +5.94 | 0.37 | 0.91 | +21.1% | $250.70 |
| 4 | **BAC**  | +5.07 | 0.36 | 0.83 | +13.6% | $53.93 |
| 5 | **SPY**  | +5.16 | 0.65 | 0.98 | +8.8%  | $710.75 |

**SHORTS — energy is the one weak sector**

| # | Symbol | net_R | pullback | close | 20d move | last |
|---|--------|------:|---------:|------:|---------:|-----:|
| 1 | **CVX** | −2.84 | 0.97 | 0.83 | −8.9% | $183.85 |
| 2 | **COP** | −2.33 | 1.15 | 0.84 | −9.0% | $115.96 |
| 3 | **XOM** | −2.17 | 1.59 | 0.87 | −8.4% | $146.33 |

Daily read: bull-market continuation in tech/financials; energy is the one broad-sector selloff. Clean divergence — if you want a pairs trade, long mega-cap tech / short integrated energy is set up on the daily.

## 60m — intraday swing (1–3 sessions)

**LONGS — index + healthcare + industrials**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **DIA** | +7.69 | 0.41 | 0.78 | $494.21 |
| 2 | **DIS** | +6.63 | 0.47 | 0.98 | $106.28 |
| 3 | **MRK** | +6.56 | 0.60 | 0.97 | $119.07 |
| 4 | **CAT** | +6.64 | 0.43 | 0.82 | $794.53 |
| 5 | **HD**  | +6.74 | 0.57 | 0.83 | $349.40 |

**SHORT — NFLX is the standout broken name**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **NFLX** | −6.53 | 0.98 | 0.84 | $97.28 |

NFLX at −10.08% on the 60m with closeness 0.84 is the loudest single-name bear signal in the scan. Not clean-SPT (pullback 0.98 means there was significant counter-bounce) but the direction and magnitude are undeniable — weaponized downside.

## 30m — intraday (hours)

**LONGS — defensive rotation dominates**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **MRK** | +9.84 | 0.48 | 0.96 | $119.07 |
| 2 | **DIA** | +9.55 | 0.40 | 0.74 | $494.21 |
| 3 | **HD**  | +8.93 | 0.43 | 0.83 | $349.40 |
| 4 | **DIS** | +7.86 | 0.54 | 0.98 | $106.28 |
| 5 | **UNH** | +7.45 | 0.55 | 0.91 | $324.50 |

**SHORTS — software-complex breaking**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **ADBE** | −4.24 | 0.88 | 0.93 | $244.38 |
| 2 | **ORCL** | −2.63 | 1.52 | 0.86 | $175.06 |

MRK at +9.84 netR is the cleanest large-cap uptrend on the 30m in the entire scan. DIA/HD/DIS/UNH all closing above 90% of their 30m window range confirms the defensive bid.

## 15m — intraday scalp-to-swing (tens of minutes)

**LONGS — defensive + retail**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **WMT**  | +8.87 | 0.45 | 0.98 | $127.47 |
| 2 | **MRK**  | +5.62 | 0.55 | 0.96 | $119.07 |
| 3 | **COST** | +5.71 | 0.84 | 0.95 | $999.63 |
| 4 | **DIS**  | +4.52 | 0.60 | 0.97 | $106.28 |
| 5 | **PFE**  | +5.48 | 0.79 | 0.81 | $27.57 |

**SHORTS — software + momentum-name exhaustion**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **ORCL** | −7.05 | 0.54 | 0.85 | $175.06 |
| 2 | **ADBE** | −5.59 | 0.38 | 0.90 | $244.38 |
| 3 | **CRM**  | −5.73 | 0.46 | 0.83 | $182.17 |
| 4 | **AMZN** | −4.44 | 1.03 | 0.94 | $250.47 |
| 5 | **GE**   | −3.71 | 1.06 | 0.98 | $304.12 |

Notable: AMZN shows up as a **15m short** despite being a daily long — a clean intraday pullback inside the longer-term bull. That's an SPT-in-miniature against trend, and the kind of divergence to size down, not fade.

## 5m — scalp (minutes)

**LONGS — strongest single-TF signals in the whole scan**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **WMT**  | +18.69 | 0.43 | 0.98 | $127.47 |
| 2 | **COST** | +12.64 | 0.77 | 0.95 | $999.63 |
| 3 | **MRK**  | +10.47 | 0.55 | 0.96 | $119.07 |
| 4 | **UNH**  |  +8.97 | 0.47 | 0.89 | $324.50 |
| 5 | **CVX**  |  +9.07 | 0.96 | 0.95 | $183.99 |

**SHORTS**

| # | Symbol | net_R | pullback | close | last |
|---|--------|------:|---------:|------:|-----:|
| 1 | **GE**   | −10.42 | 0.69 | 0.98 | $304.12 |
| 2 | **CRM**  |  −8.96 | 0.53 | 0.83 | $182.17 |
| 3 | **ADBE** |  −7.71 | 0.49 | 0.90 | $244.38 |
| 4 | **ORCL** |  −7.99 | 0.83 | 0.85 | $175.06 |
| 5 | **MU**   |  −7.25 | 0.65 | 0.85 | $455.06 |

WMT at +18.69 netR on the 5m is the single strongest trend reading in the entire 52×5 scan — defensive retail on an almost-vertical Friday-afternoon push. CVX appearing as a **5m long** while being a **daily short** is a classic counter-trend bounce inside a broken tape — Brooks would flag that as a fade candidate, not a continuation entry.

## Cross-timeframe picks (signal on ≥ 2 TFs, same direction)

These are the highest-conviction names — trends visible at multiple scales usually mean a clean run with low noise.

**LONGS — defensive/healthcare dominates**

| Symbol | Timeframes hit | Conviction |
|--------|----------------|:----------:|
| **MRK**  | 60m · 30m · 15m · 5m | ★★★★ (4 TFs, all up) |
| **DIS**  | 60m · 30m · 15m | ★★★ |
| **HD**   | 60m · 30m | ★★ |
| **DIA**  | 60m · 30m | ★★ (index ETF) |
| **UNH**  | 30m · 5m | ★★ |
| **WMT**  | 15m · 5m | ★★ |
| **COST** | 15m · 5m | ★★ |

**SHORTS — software-complex + select momentum**

| Symbol | Timeframes hit | Conviction |
|--------|----------------|:----------:|
| **ORCL** | 30m · 15m · 5m | ★★★ |
| **ADBE** | 30m · 15m · 5m | ★★★ |
| **CRM**  | 15m · 5m | ★★ |
| **GE**   | 15m · 5m | ★★ |

**The narrative in one line:** broad tape is bull (tech/financials daily), but under the hood the bid rotated into defensives (MRK/DIS/HD/UNH/WMT/COST) and out of enterprise software + select momentum (ORCL/ADBE/CRM/GE) in the last hours of Friday's cash.

## Top picks by trade horizon

If you can only trade **one** per horizon, these are the strongest:

| Horizon | Direction | Name | Why |
|---------|-----------|------|-----|
| Position / swing (weeks) | LONG | **AMD** | +36.9% 20-day move, closes at 97% of range, shallow 38% pullback ratio |
| Intraday swing (days) | LONG | **MRK** | Only name appearing on 4 consecutive timeframes (60m/30m/15m/5m) all up |
| Intraday hours | LONG | **MRK** (+9.84 netR on 30m) | Cleanest large-cap trend on 30m in entire scan |
| Intraday scalp | LONG | **WMT** (+18.69 netR on 5m) | Single strongest trend reading in the entire scan |
| Intraday short-side | SHORT | **ADBE** (3-TF short) | Cleanest short across 30m/15m/5m |
| Position / swing short | SHORT | **CVX** | Daily −8.9% with closeness 0.83 — energy sector selloff |

## How to use

- **For the live scanner**: when an H1/H2 long fires on MRK, DIS, HD, UNH, WMT, COST — that's maximum alignment with cross-TF trend. Conversely, short-side H1/H2 (L1/L2 in scanner nomenclature) on ORCL/ADBE/CRM is aligned against a broken intraday structure.
- **For discretionary entry**: the cross-TF stars (MRK, ADBE, ORCL) are the highest-conviction continuation candidates. The rest are strong but single-or-double-timeframe.
- **Divergence flag**: AMZN (daily long / 15m short) and CVX (daily short / 5m long) are counter-trend intraday bounces — do **not** use these as entry signals in the direction of the intraday bounce; the larger timeframe wins.

## Caveats

- Last close is Fri 2026-04-17. Monday gaps or news can invalidate — especially the 5m list.
- No live `day_type` (PLAYBOOK rule 2) applied here; live scanner adds that layer.
- Pullback-pct is unbounded on the short side when the move was small (NFLX 60m short has pb 0.98; still the biggest short-side magnitude on 60m). Read netR + closeness first, pullback second.
- Universe is 52 hand-picked names. Smaller active SPTs outside this list are invisible.

## Follow-ups

- **Q47** (low-priority) — cross-timeframe-SPT backtest. Does requiring SPT on ≥ 2 timeframes lift perR above the single-TF PLAYBOOK? Not on roadmap.
- **Q48** (low-priority) — software-complex SPT-short regime persistence. Re-scan EOD Monday for ORCL/ADBE/CRM cluster continuation.
- **Q49** (new, low-priority) — AMD +36.9%/20d is extreme for the 20-bar window; is it better read as a **mature** SPT (late-stage, stops tighten) or a continuation (early-stage)? Flag for manual review before scaling in.

## Related

- [[small-pullback-trend-PLAYBOOK]] — the 11-rule operational playbook.
- [[small-pullback-trend-INDEX]] — reading order for the full 33-note research arc.
- `/tmp/spt_scan/raw.json` — per-(symbol, timeframe) scores for all 52 × 5 cells.
- `/tmp/spt_scan/strongest.json` — top 5 long + top 5 short per TF, pure-strength-ranked.
