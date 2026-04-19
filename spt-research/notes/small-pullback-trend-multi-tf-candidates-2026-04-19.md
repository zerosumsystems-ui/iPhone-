# SPT multi-timeframe candidate scan — 2026-04-19

**Purpose.** Take the SPT PLAYBOOK off paper and find, right now, stocks whose tape matches the small-pullback-trend signature across multiple timeframes. Same definition Brooks teaches: a directional move where every retrace stays shallow and the window closes near the extreme. One candidate list per timeframe so the trader can match the signature to the horizon they're working.

Data: Databento `XNAS.ITCH` (EQUS single names) and ETF composites, through Fri 2026-04-17 cash close. Universe n=52 (mega-cap tech + momentum + financials + industrials + defensives + the big 4 index ETFs). Timeframes {daily, 60m, 30m, 15m, 5m}. Scan script at `/tmp/spt_scan/scan.py`, re-rank at `/tmp/spt_scan/rerank.py`, raw per-symbol rows at `/tmp/spt_scan/raw.json`.

## Methodology

Per (symbol, timeframe) pull the last N bars and compute:

- **net_R** — net close-to-close move in units of window-mean true range. Directionless magnitude; sign = trend direction.
- **pullback_pct** — largest adverse excursion against the trend, divided by |net move|. SPT is **shallow pullbacks**, so lower is better.
- **close_pos** — position of last close within the window's high-low range (0..1). Long-SPT wants ≥ 0.75; short-SPT wants ≤ 0.25.
- **composite** — `0.5·|net_R| + 2.0·extreme_closeness + 2.0·pullback_penalty`, signed by direction.

Timeframe-scaled gates (intraday has more bar-to-bar noise so gates widen):

| TF   | min \|net_R\| | max pullback_pct | min closeness | window |
|------|--------------:|-----------------:|--------------:|-------:|
| daily | 2.0 | 0.40 | 0.75 | 20 |
| 60m   | 2.0 | 0.45 | 0.75 | 20 |
| 30m   | 2.0 | 0.55 | 0.70 | 26 |
| 15m   | 1.8 | 0.70 | 0.65 | 26 |
| 5m    | 1.5 | 0.90 | 0.55 | 78 |

Grade **A** = ≥1.3× min_R, ≤0.7× max_pb, ≥min_close+0.10. Grade **B** = inside gate. Grade **C** (failed gate) dropped from the lists below.

## Recommendations by timeframe

### Daily — position / swing (weeks)

Strong tape across the top of the universe. No short-side daily SPT passes the gate today; that matches the index read (SPY/QQQ/IWM/DIA all daily-SPT longs).

| # | Grade | Symbol | netR | pullback | close | 20d move |   last |
|---|:-----:|--------|-----:|---------:|------:|---------:|-------:|
| 1 | B | **AVGO**  | +6.61 | 0.39 | 0.99 | +29.6% | $406.06 |
| 2 | B | **AMD**   | +6.65 | 0.38 | 0.97 | +36.9% | $277.25 |
| 3 | B | **AMZN**  | +5.94 | 0.37 | 0.91 | +21.1% | $250.70 |
| 4 | B | **BAC**   | +5.07 | 0.36 | 0.83 | +13.6% | $53.93  |

All four are closing in the top 20% of their 20-day range with pullbacks sub-40% of the net move — textbook daily SPT. AMD and AVGO are the purest reads (near-high close, +29–37% 20-day moves).

### 60m — intraday swing (1–3 sessions)

Broad tape shows up here: the index ETFs (DIA, SPY) themselves qualify, plus CAT as the industrial proxy. No individual single-names pass the 60m gate that weren't already daily-strong — meaning the 60m run IS the daily run, not a separate signal.

| # | Grade | Symbol | netR | pullback | close |   last |
|---|:-----:|--------|-----:|---------:|------:|-------:|
| 1 | B | **DIA**  | +7.69 | 0.41 | 0.78 | $494.21 |
| 2 | B | **CAT**  | +6.64 | 0.43 | 0.82 | $794.53 |
| 3 | B | **SPY**  | +5.97 | 0.38 | 0.82 | $710.04 |

Actionable read: don't fade the Dow or industrials into Monday. If you're looking for a 60m-timeframe entry, you're pattern-matching to the same move the daily list is already expressing.

### 30m — intraday (hours)

First timeframe where the **defensive/healthcare/consumer rotation** shows up cleanly. Tech/financials are less represented here than on the daily — the 30m window captures the late-session Friday bid into defensives.

| # | Grade | Symbol | netR | pullback | close | last |
|---|:-----:|--------|-----:|---------:|------:|-----:|
| 1 | B | **MRK**  | +9.84 | 0.48 | 0.96 | $119.07 |
| 2 | B | **DIA**  | +9.55 | 0.40 | 0.74 | $494.21 |
| 3 | B | **HD**   | +8.93 | 0.43 | 0.83 | $349.40 |
| 4 | B | **DIS**  | +7.86 | 0.54 | 0.98 | $106.28 |
| 5 | B | **UNH**  | +7.45 | 0.55 | 0.91 | $324.50 |

MRK is the standout (+9.84 netR, closing at 96% of range). The list is defensively-skewed — pharma, retail, entertainment, managed care — which is worth noting: tech led the daily, defensives led the 30m. If Monday opens strong-tech the 30m defensives may be rotating out.

### 15m — intraday scalp-to-swing (tens of minutes)

First timeframe with **short-side SPT**. ADBE and CRM both qualify as A-grade shorts — pullback ≤ 0.46, close in bottom 20% of range — which means the software complex is quietly weaker than the broad tape. Single long-side A-grade: WMT.

| # | Grade | Dir | Symbol | netR | pullback | close |
|---|:-----:|:---:|--------|-----:|---------:|------:|
| 1 | A | L | **WMT**   | +8.87 | 0.45 | 0.98 |
| 2 | B | L | **MRK**   | +5.62 | 0.55 | 0.96 |
| 3 | B | L | **DIS**   | +4.52 | 0.60 | 0.97 |
| 4 | B | L | GOOGL     | +4.16 | 0.63 | 0.88 |
| 5 | B | L | CAT       | +4.69 | 0.69 | 0.72 |
| 1 | B | S | **ORCL**  | −7.05 | 0.54 | 0.85 |
| 2 | A | S | **ADBE**  | −5.59 | 0.38 | 0.90 |
| 3 | A | S | **CRM**   | −5.73 | 0.46 | 0.83 |

Signal: software stack (ORCL/ADBE/CRM) diverging from mega-cap tech (AAPL/NVDA/GOOGL). This is the **best cross-timeframe divergence** in the whole scan — daily says tech is green, 15m says enterprise software is red. Watch for continuation Monday.

### 5m — scalp (minutes)

Finest timeframe. The 5m SPT lists confirm the 15m read at tighter granularity: WMT + defensives up, enterprise-software down. Short list is loud — 5 A/B-grade short-side SPTs is unusual and suggests the last hour of Friday's cash was a genuine software-complex flush.

| # | Grade | Dir | Symbol | netR | pullback | close |
|---|:-----:|:---:|--------|-----:|---------:|------:|
| 1 | A | L | **WMT**   | +18.69 | 0.43 | 0.98 |
| 2 | B | L | **COST**  | +12.64 | 0.77 | 0.95 |
| 3 | A | L | **MRK**   | +10.47 | 0.55 | 0.96 |
| 4 | A | L | **UNH**   |  +8.97 | 0.47 | 0.89 |
| 5 | A | L | **GOOGL** |  +7.90 | 0.54 | 0.88 |
| 1 | B | S | **GE**    | −10.42 | 0.69 | 0.98 |
| 2 | A | S | **CRM**   |  −8.96 | 0.53 | 0.83 |
| 3 | A | S | **ADBE**  |  −7.71 | 0.49 | 0.90 |
| 4 | B | S | **ORCL**  |  −7.99 | 0.83 | 0.85 |
| 5 | B | S | **MU**    |  −7.25 | 0.65 | 0.85 |

WMT at +18.69 netR is the cleanest 5m SPT in the scan — defensive retail on a shallow-pullback push into the weekend.

## Cross-timeframe picks (where the scan agrees)

These are the names that show up as SPT on **≥ 2 timeframes** with matching direction. These are the highest-conviction candidates — Brooks intuition is that SPT on multiple timeframes means a very clean trend with low noise.

| Symbol | Direction | Timeframes hit | Notes |
|--------|:---------:|-----------------|-------|
| **WMT**  | long  | 15m · 5m | A-grade on both; best single-name 5m-SPT in the scan |
| **MRK**  | long  | 30m · 15m · 5m | Triple-timeframe healthcare winner |
| **DIS**  | long  | 30m · 15m | Media continuation |
| **UNH**  | long  | 30m · 5m | Managed-care bid |
| **DIA**  | long  | 60m · 30m | Dow index run |
| **CAT**  | long  | 60m · 15m | Industrials proxy |
| **GOOGL** | long | 15m · 5m | Only mega-cap tech with intraday SPT |
| **CRM**  | short | 15m · 5m | A-grade on both; cleanest short |
| **ADBE** | short | 15m · 5m | A-grade on both; complements CRM |
| **ORCL** | short | 15m · 5m | B-grade short; confirms software weakness |

**Narrative read.** Daily says tech and financials are in clean uptrends (AVGO/AMD/AMZN/BAC). Intraday (30m and finer) shows the bid rotated into defensives (MRK/UNH/HD/DIS/WMT) and out of enterprise software (ORCL/ADBE/CRM) in the last hours of Friday's cash. This is a **tape inside a tape** — broad bullish backdrop with a short-side SPT sub-regime in software.

## How to use this list with the PLAYBOOK

These are SPT **candidates**, not signal-bar entries. The PLAYBOOK's 9-rule stack (with conditional rules 10 and 11) sits on top — rules 1/3/4/5/6/7/8 still apply. Use this list as:

- **Universe pre-filter** for the live scanner's entry band. When the scanner fires H1/H2 longs on a name from the long list, that's a highly-aligned cross-timeframe signal.
- **Timeframe-of-trade anchor.** If you're scalping 5m, pick candidates from the 5m list (WMT/MRK long, CRM/ADBE short). If you're position-swinging, daily list (AVGO/AMD/AMZN).
- **Divergence flag.** CRM/ADBE/ORCL are short on 15m/5m but the daily universe is broadly long — that's a dangerous cell for longs on those three names, and a setup-selection nudge for shorts there.

## Caveats

- **No live day_type label.** PLAYBOOK rule 2 requires `day_type ∈ {trend_from_open, spike_and_channel}` which is scanner-derived; this static universe scan doesn't know today's day-type. Running the live scanner Monday adds that layer.
- **Last close was Fri 04-17.** Monday's tape can invalidate any of these reads — especially the 5m list, which is hours-old by open.
- **Universe is 52 names.** Smaller names with active SPTs (names not in the universe) are not seen. For the production run, use the scanner's full EQUS.MINI universe.
- **Gates are heuristic.** The PLAYBOOK's actual entry logic is bar-pattern (H1/H2/L1/L2 with opp_tail < 0.25) — these windowed composites are a pre-filter for that.

## Follow-ups

- **Q47 (low-priority)** — cross-timeframe-SPT backtest. Does requiring SPT on ≥ 2 timeframes lift perR above the single-TF stack? Would need per-bar timeframe stacking in scanner; not on roadmap. File against the scanner architecture next iteration.
- **Q48 (low-priority)** — software-complex SPT-short regime. CRM/ADBE/ORCL all short on 15m/5m is a cluster worth a look — is this a 1-day flush or a 2-3-week theme? Re-run the scan at EOD Monday to see if it persists.

## Related

- [[small-pullback-trend-PLAYBOOK]] — the consolidated 11-rule trading playbook.
- [[small-pullback-trend-INDEX]] — reading order for the 32-note research arc.
- `/tmp/spt_scan/raw.json` — raw per-(symbol, timeframe) scores, all 52 × 5 cells.
- `/tmp/spt_scan/ranked_v2.json` — filtered/graded top 5 long + top 5 short per timeframe.
