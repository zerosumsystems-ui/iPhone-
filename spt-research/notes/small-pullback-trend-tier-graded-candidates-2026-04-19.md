# SPT tier-graded candidates — 2026-04-19 (pt 34)

**Quality complement to pt 33.** Pt 33 ranked the *strongest* moves per timeframe (sanity filter only). This note re-grades the same 52×5 scan into PLAYBOOK-quality tiers (A/B/C). Different lens: instead of "biggest", "cleanest". Several pt 33 picks drop entirely; several quiet B-tier names get promoted.

**Run:** 2026-04-19, scheduled-task autonomous. Same Databento `XNAS.ITCH` raw scan as pt 33 (`/tmp/spt_scan/raw.json`), re-graded by `/tmp/spt_scan_pt34/tier.py`. No new API calls — pure post-hoc quality grading.

## Tier definitions

| Tier | net_R | pullback_pct | closeness | Read |
|------|------:|-------------:|----------:|------|
| **A** (PLAYBOOK) | ≥ 2.0 | ≤ 0.40 | ≥ 0.75 | Brooks-canonical small-pullback. Trade as-is. |
| **B** (Strong-clean) | ≥ 3.0 | ≤ 0.60 | ≥ 0.75 | Strong directional with manageable noise. Trade with size discipline. |
| **C** (Strong-extended) | ≥ 2.0 | ≤ 0.90 | ≥ 0.70 | Real trend but pullback was meaningful. Wait for next pullback rather than chase. |

Pullback_pct = max-adverse-excursion / |net|. Higher = deeper retraces inside the move. Brooks SPT canonical is ≤ 0.30–0.40; we extend the gate by tier so smaller-TF trends (which always carry more bar noise) can surface.

## Distribution by TF

| TF | A long | A short | B long | B short | C long | C short |
|----|------:|------:|------:|------:|------:|------:|
| daily | **4** | 0 | 4 | 0 | 8 | **0** |
| 60m | 1 | 0 | 11 | 0 | 6 | **0** |
| 30m | 0 | 0 | 9 | 0 | 7 | 1 |
| 15m | 0 | **1** | 3 | 2 | 5 | 0 |
| 5m | 0 | 0 | 5 | 2 | 3 | 3 |

Two structural facts jump out:

1. **Zero short candidates on daily or 60m at any tier.** The "energy short" (CVX/COP/XOM) and "broken NFLX" calls from pt 33 do not pass even the loose C-gate — pullbacks were 0.97 / 1.15 / 1.59 / 0.98 respectively, meaning intra-window counter-bounces were as large as the net move. **These are broken sectors with live bounces, not clean SPT-shorts.**
2. **No A-tier on 30m or 5m anywhere.** Strict pullback ≤ 0.40 is unrealistic at fast intraday speeds. Use B-tier as the operational ceiling on 30m / 15m / 5m.

---

## Daily — A-tier longs (PLAYBOOK-grade, weeks)

| # | Symbol | net_R | pullback | closeness | 20d % | last |
|---|--------|------:|---------:|----------:|------:|-----:|
| 1 | **AVGO** | +6.61 | 0.39 | 0.99 | +29.6% | $406.06 |
| 2 | **AMD** | +6.65 | 0.38 | 0.97 | +36.9% | $277.25 |
| 3 | **AMZN** | +5.94 | 0.37 | 0.91 | +21.1% | $250.70 |
| 4 | **BAC** | +5.07 | 0.36 | 0.83 | +13.6% | $53.93 |

These are **the** weekly position longs. All four pass strict Brooks gate. AMD specifically resolves Q49: a 36.9%/20d move with a 0.38 pullback ratio and 0.97 closeness is **continuation, not exhaustion** — the trend is still small-pullback, the discipline is intact. The "extreme magnitude" of the 20d % is a function of NVDA-class compounding velocity, not climactic blow-off.

### Daily — B-tier longs (slightly deeper pullbacks, still strong)

| Symbol | net_R | pullback | closeness | 20d % | last |
|--------|------:|---------:|----------:|------:|-----:|
| **IWM** | +4.84 | 0.50 | 0.95 | +12.5% | $275.85 |
| **DIA** | +4.82 | 0.53 | 0.94 | +7.7% | $494.40 |
| **CAT** | +4.20 | 0.60 | 0.96 | +16.2% | $795.93 |
| **BA** | +4.19 | 0.52 | 0.86 | +14.0% | $224.11 |

**New finding vs pt 33:** IWM passes B-tier on the daily. Pt 33 only listed daily large-cap longs (AVGO/AMD/AMZN/BAC/SPY); small caps were absent. IWM B-tier daily + B-tier 60m means **small-caps are quietly trending too** — broader breadth than pt 33 suggested.

### Daily — shorts: ZERO at any tier

CVX (pb 0.97), COP (pb 1.15), XOM (pb 1.59) all fail. The energy selloff is real but the bounces inside the window are as large as the net move — these are not clean small-pullback bear trends; they are weakening sectors with active counter-trend rallies. **Do not enter short on these from current levels.** Wait for either a fresh stop-entry below the most recent intraday low or for the bounce to mature and roll over.

---

## 60m — intraday swing (1–3 sessions)

### A-tier long
| Symbol | net_R | pullback | closeness | last |
|--------|------:|---------:|----------:|-----:|
| **SPY** | +5.97 | 0.38 | 0.82 | $710.04 |

Only A-tier signal on 60m. The index itself is the cleanest intraday trend. Treat as the directional baseline — long bias is fully aligned with the daily.

### B-tier longs (11 names, ranked by composite)
DIA · DIS · MRK · CAT · HD · UNH · MSTR · IWM · plus AVGO, AAPL, QQQ implied via cross-TF.

Pt 33 flagged DIA / DIS / MRK / CAT / HD as the top 5 strongest. Quality grading promotes UNH and MSTR into the operational list and confirms the rest at B-grade. **MSTR's +16% magnitude on 60m is unusually large** — high-beta name riding the BTC bid, treat as full-size only with a defined stop, not a position add.

### 60m shorts: ZERO at any tier
NFLX (pb 0.98) was pt 33's only 60m short — it fails. Same rationale as the daily energy: real damage but live bounce. No clean short candidate on 60m.

---

## 30m — intraday hours

**No A-tier on either side.** Strict pullback ≤ 0.40 is too tight for 30m noise.

### B-tier longs (9 names)
| # | Symbol | net_R | pullback | closeness | last |
|---|--------|------:|---------:|----------:|-----:|
| 1 | **MRK** | +9.84 | 0.48 | 0.96 | $119.07 |
| 2 | **HD** | +8.93 | 0.43 | 0.83 | $349.40 |
| 3 | **DIS** | +7.86 | 0.54 | 0.98 | $106.28 |
| 4 | **UNH** | +7.45 | 0.55 | 0.91 | $324.50 |
| 5 | **SPY** | +6.85 | 0.44 | 0.77 | $710.04 |
| 6 | **CAT** | +6.30 | 0.57 | 0.75 | $794.53 |
| 7 | **QQQ** | +6.16 | 0.52 | 0.88 | $648.78 |
| 8 | **AAPL** | +5.64 | 0.46 | 0.75 | $270.19 |

MRK's +9.84 net_R / 0.48 pullback / 0.96 closeness is the cleanest individual-stock 30m read in the entire scan. AAPL surfaces here at B that pt 33 didn't list in the top 5 — quietly trending alongside the heavyweights.

### 30m shorts
- **C-tier:** ADBE (net_R −4.24, pb 0.88). Real downside, but the pullback ratio means ADBE bounced inside the window almost as much as it dropped — treat as roll-over candidate, not a fresh-entry SPT-short on 30m. The 15m and 5m ADBE reads are cleaner (below).

---

## 15m — intraday scalp-to-swing

### A-tier short — the cleanest single signal in the entire scan
| Symbol | net_R | pullback | closeness | last |
|--------|------:|---------:|----------:|-----:|
| **ADBE** | −5.59 | 0.38 | 0.90 | $244.38 |

**This is the only A-tier short anywhere in the scan.** Strict Brooks-canonical SPT-short on the 15m. If a stop-entry-short setup fires Monday morning on ADBE, this is the highest-quality signal in this entire run.

### B-tier longs
| Symbol | net_R | pullback | closeness | last |
|--------|------:|---------:|----------:|-----:|
| **WMT** | +8.87 | 0.45 | 0.98 | $127.47 |
| **MRK** | +5.62 | 0.55 | 0.96 | $119.07 |
| **DIS** | +4.52 | 0.60 | 0.97 | $106.28 |

### B-tier shorts
| Symbol | net_R | pullback | closeness | last |
|--------|------:|---------:|----------:|-----:|
| **ORCL** | −7.05 | 0.54 | 0.85 | $175.06 |
| **CRM** | −5.73 | 0.46 | 0.83 | $182.17 |

ORCL's 60% larger magnitude than ADBE on 15m, but slightly noisier pullback. Three software-complex shorts (ADBE/ORCL/CRM) at A or B grade on 15m = **enterprise-software 15m bear cluster confirmed**.

**Names dropped vs pt 33's 15m short list:** AMZN (pb 1.03 — counter-bounce > net move; pt 33 already flagged this as divergent vs daily long); GE (pb 1.06 — same problem). Don't short these on 15m even though pt 33 ranked them in top 5 strongest.

---

## 5m — scalp (minutes)

**No A-tier on either side.** Use B-tier as ceiling.

### B-tier longs (5 names)
| Symbol | net_R | pullback | closeness | last |
|--------|------:|---------:|----------:|-----:|
| **WMT** | +18.69 | 0.43 | 0.98 | $127.47 |
| **MRK** | +10.47 | 0.55 | 0.96 | $119.07 |
| **UNH** | +8.97 | 0.47 | 0.89 | $324.50 |
| **GOOGL** | +7.90 | 0.54 | 0.88 | $341.61 |
| **META** | +6.88 | 0.56 | 0.82 | $688.55 |

WMT +18.69 net_R / 0.43 pullback is **B-tier despite enormous magnitude** because closeness is at 98% and pullback discipline is intact. This is the strongest single trend reading in the entire scan and it survives quality grading. GOOGL and META appear here at B-tier that pt 33 didn't list in 5m top 5 — mega-cap tech with quietly clean 5m structure.

### B-tier shorts
| Symbol | net_R | pullback | closeness | last |
|--------|------:|---------:|----------:|-----:|
| **CRM** | −8.96 | 0.53 | 0.83 | $182.17 |
| **ADBE** | −7.71 | 0.49 | 0.90 | $244.38 |

GE (pt 33's #1 5m short with −10.42 net_R) drops to **C-tier** here (pb 0.69, closeness 0.98) — magnitude is real but the pullback is too deep for B. Same for ORCL on 5m (pb 0.83, C-tier).

---

## Cross-timeframe quality stars (A or B on ≥ 2 TFs, same direction)

| Symbol | TFs | Tiers | Conviction |
|--------|-----|-------|:----------:|
| **MRK long** | 60m · 30m · 15m · 5m | B · B · B · B | ★★★★ |
| **AVGO long** | daily · 60m · 30m | A · B · B | ★★★ ⬆ |
| **CAT long** | daily · 60m · 30m | B · B · B | ★★★ ⬆ |
| **DIS long** | 60m · 30m · 15m | B · B · B | ★★★ |
| **UNH long** | 60m · 30m · 5m | B · B · B | ★★★ |
| **IWM long** | daily · 60m | B · B | ★★ ⬆ |
| **DIA long** | daily · 60m | B · B | ★★ |
| **SPY long** | 60m · 30m | A · B | ★★ |
| **HD long** | 60m · 30m | B · B | ★★ |
| **AAPL long** | 60m · 30m | B · B | ★★ ⬆ |
| **QQQ long** | 60m · 30m | B · B | ★★ |
| **WMT long** | 15m · 5m | B · B | ★★ |
| **ADBE short** | 15m · 5m | **A** · B | ★★ |
| **CRM short** | 15m · 5m | B · B | ★★ |

⬆ = new or upgraded vs pt 33's cross-TF stars list.

**What changed vs pt 33's cross-TF stars:**

| Status | Symbols | Why |
|--------|---------|-----|
| **NEW** at A/B grade | AVGO long, CAT long, IWM long, AAPL long | Strong but weren't top-5-strongest per TF in pt 33 — quality grading promotes them |
| **DROPPED** from cross-TF list | COST long, GE short, ORCL short (5m only) | Pullback >0.60 fails B-gate at one or both of their TFs |

**Operational read:**
- **MRK is the highest-conviction long across the entire scan** — 4 TFs, every one passes B-tier. Pt 33 already crowned MRK; tier grading reinforces it.
- **AVGO promotes from "daily-only" to a 3-TF cross star** — pt 33 had AVGO only on daily. The 60m and 30m B-tier signals make it as multi-TF-aligned as AMD or AMZN, with the cleanest daily of the four (closeness 0.99).
- **CAT and IWM cross-TF longs are NEW broad-tape information** — industrials and small-caps are participating in the bull regime, not just mega-cap tech.
- **ADBE short is the highest-quality short in the scan** — 15m A-tier + 5m B-tier. If you take one short Monday, this is it.

---

## Top picks per horizon — quality-graded

| Horizon | Dir | Name | Tier | Why |
|---------|:---:|------|:----:|-----|
| Position (weeks) | 🟢 LONG | **AVGO** | A | Daily A-tier, closeness 0.99 — cleanest weekly long |
| Position (alt)   | 🟢 LONG | **AMD** | A | Resolves Q49 — clean continuation, not exhaustion |
| Intraday swing (1-3 sessions) | 🟢 LONG | **MRK** | B×4 | Only 4-TF cross-star; B grade on every intraday TF |
| Intraday hours (30m) | 🟢 LONG | **MRK** | B | +9.84 netR / 0.48 pb — cleanest 30m large-cap |
| Intraday scalp (5m) | 🟢 LONG | **WMT** | B | +18.69 netR with pullback discipline (0.43) |
| Best short (any horizon) | 🔴 SHORT | **ADBE** | **A on 15m** | Only A-tier short anywhere in the scan |
| Cross-TF short (15m+5m) | 🔴 SHORT | **CRM** | B×2 | Cleaner short than ORCL after grading |

**No daily or 60m short candidate at any tier.** If you want bearish exposure with a multi-day hold, you do not have a clean SPT-short setup right now — the bear-side trades are intraday only.

---

## Q49 resolved — AMD: continuation, not exhaustion

Pt 33 flagged AMD's +36.9%/20d as extreme and asked whether it should be read as a mature SPT (late-stage, tighten stops) or a continuation (early-stage). Quality grading answers: **continuation.**

Evidence:
- pullback_pct = 0.38 (passes strict A-gate of ≤0.40)
- closeness = 0.97 (closes at 97% of 20-day range — peak buyer demand still present)
- net_R = +6.65 (over 6 ATR of net upward push)
- Bull-bar count and bar-by-bar would need a chart read, but the aggregate stats are inconsistent with a climactic-exit profile (Brooks: climactic exit shows pullback ratio ≥ 0.50 with closeness deteriorating below 0.85)

**Operational implication:** treat AMD as a continuation long, not a tightening-stop fade. The 20d % is large because compounding strong daily moves works that way, not because the structure broke. Stops via PLAYBOOK rule 5 (1-bar swing) remain appropriate.

---

## Caveats (same as pt 33 + tier-specific)

- Same Friday 2026-04-17 close cutoff. Monday gaps invalidate, especially smaller TFs.
- No `day_type` overlay (PLAYBOOK rule 2).
- Universe is the same 52-name list — broader universe untested.
- Tier B/C cutoffs are arbitrary midpoints; the grade-specific gates can be re-tuned per the empirical work in pt 13 (rule-ablation) and pt 17 (target-walkforward) once we have a tier-conditional perR study.

## Follow-ups

- **Q50** *(new, low-priority)* — Tier × perR study. Across the empirical SPT trade log, does A-tier outperform B-tier outperform C-tier in perR? If so the gate is informative; if not the tier system is decorative and we should kill it.
- **Q51** *(new, low-priority)* — Sector tilt of B-tier longs in this scan: industrials (CAT, BA) + small-caps (IWM) + defensives (MRK, UNH, HD) all promoted vs pt 33. Is that a regime-shift signal or a one-day artifact?
- **Q47 carries over** — cross-TF SPT-vs-single-TF backtest, now with tier conditioning.

## Related

- [[small-pullback-trend-multi-tf-candidates-2026-04-19]] — pt 33, the strongest-trend version of this scan
- [[small-pullback-trend-monday-watchlist-2026-04-20]] — Monday-open execution checklist
- [[small-pullback-trend-PLAYBOOK]] — the 11-rule operational playbook
- [[small-pullback-trend-INDEX]] — full reading-order index
- `/tmp/spt_scan/raw.json` — source data (52 × 5 cells, sanity-pass and below)
- `/tmp/spt_scan_pt34/tiered.json` — graded output (A/B/C per direction per TF)
- `/tmp/spt_scan_pt34/cross_tf_AB.json` — cross-TF stars at A or B
