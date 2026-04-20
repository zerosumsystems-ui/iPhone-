# SPT research — phone briefing

Autonomous small-pullback-trend research notes, phone-readable. Long-form notes live in [`notes/`](notes/); phone-friendly PDFs in [`pdfs/`](pdfs/); the latest TL;DR is at the top of this README.

> 📚 **Full research archive — [ARCHIVE.md](ARCHIVE.md)** lists every SPT note ever written (34 notes in reading order + the PLAYBOOK + the INDEX). Canonical mirror at the aiedge-vault: [github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts](https://github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts).

**Last updated:** 2026-04-19 late · pt 34 — tier-graded candidates (A/B/C per TF)

> 📅 **Monday 2026-04-20 open — actionable watchlist:** [notes/small-pullback-trend-monday-watchlist-2026-04-20.md](notes/small-pullback-trend-monday-watchlist-2026-04-20.md). One-table view + PLAYBOOK entry gates + cross-TF stars + gap-risk checklist. Start here before the 10:30 ET entry window.

## Latest — pt 34: tier-graded SPT candidates per timeframe

**Run:** 2026-04-19 late, scheduled-task autonomous. Quality complement to pt 33: same 52×5 raw scan, re-graded into PLAYBOOK A/B/C tiers. Pt 33 ranked the *strongest* moves; pt 34 ranks the *cleanest*. Several pt 33 picks drop entirely (no daily/60m short candidates pass even C-gate); several quiet B-tier names get promoted (AVGO 3-TF, IWM, CAT, AAPL).

Data: same source as pt 33 (`/tmp/spt_scan/raw.json`, Databento `XNAS.ITCH` through Fri 2026-04-17 cash close). Re-grader: `/tmp/spt_scan_pt34/tier.py`.

## 🏆 Tier definitions

| Tier | net_R | pullback | closeness | Read |
|------|------:|---------:|----------:|------|
| **A** | ≥2.0 | ≤0.40 | ≥0.75 | Brooks-canonical SPT — trade as-is |
| **B** | ≥3.0 | ≤0.60 | ≥0.75 | Strong-clean — trade with size discipline |
| **C** | ≥2.0 | ≤0.90 | ≥0.70 | Strong-extended — wait for next pullback, don't chase |

## 🎯 Top picks per horizon — quality-graded

| Horizon | Dir | Name | Tier | Why |
|---------|:---:|------|:----:|-----|
| Position (weeks) | 🟢 | **AVGO** | A | Daily A, closeness 0.99 — cleanest weekly long |
| Position alt | 🟢 | **AMD** | A | Q49 resolved: continuation, not exhaustion |
| Intraday swing | 🟢 | **MRK** | B×4 | Only 4-TF cross-star; B on every intraday TF |
| Intraday hours (30m) | 🟢 | **MRK** | B | +9.84 netR / 0.48 pb — cleanest 30m large-cap |
| Intraday scalp (5m) | 🟢 | **WMT** | B | +18.69 netR with pb discipline (0.43) |
| Best short | 🔴 | **ADBE 15m** | **A** | Only A-tier short anywhere in the scan |
| Cross-TF short | 🔴 | **CRM** | B×2 | Cleaner than ORCL after grading |

**No daily or 60m short candidate at any tier.** Bear exposure is intraday-only this run.

## 📊 Per-TF tier counts (long / short)

| TF | A | B | C | Notes |
|----|:-:|:-:|:-:|------|
| daily | 4/0 | 4/0 | 8/0 | Bull breadth solid; no clean daily short anywhere |
| 60m | 1/0 | 11/0 | 6/0 | SPY is the only A; 11 B-grade longs |
| 30m | 0/0 | 9/0 | 7/1 | MRK B+9.84 netR is the cleanest 30m large-cap |
| 15m | 0/**1** | 3/2 | 5/0 | **ADBE A-short — only A-tier short in scan** |
| 5m | 0/0 | 5/2 | 3/3 | WMT +18.69 netR is the strongest single reading |

## ⭐ Cross-timeframe quality stars (A or B on ≥2 TFs)

| Symbol | TFs | Tiers | Star | vs pt 33 |
|--------|-----|-------|:----:|:--------:|
| **MRK** long | 60m·30m·15m·5m | B·B·B·B | ★★★★ | = |
| **AVGO** long | daily·60m·30m | A·B·B | ★★★ | ⬆ NEW |
| **CAT** long | daily·60m·30m | B·B·B | ★★★ | ⬆ NEW |
| **DIS** long | 60m·30m·15m | B·B·B | ★★★ | = |
| **UNH** long | 60m·30m·5m | B·B·B | ★★★ | ⬆ |
| **IWM** long | daily·60m | B·B | ★★ | ⬆ NEW (small-caps) |
| **DIA** long | daily·60m | B·B | ★★ | = |
| **SPY** long | 60m·30m | A·B | ★★ | ⬆ |
| **HD** long | 60m·30m | B·B | ★★ | = |
| **AAPL** long | 60m·30m | B·B | ★★ | ⬆ NEW |
| **QQQ** long | 60m·30m | B·B | ★★ | ⬆ |
| **WMT** long | 15m·5m | B·B | ★★ | = |
| **ADBE** short | 15m·5m | **A**·B | ★★ | = |
| **CRM** short | 15m·5m | B·B | ★★ | = |

## ⚠️ Pt 33 picks that DROP at quality grading

- **Energy daily shorts** — CVX (pb 0.97), COP (pb 1.15), XOM (pb 1.59) all fail C-gate. Bounces inside the window are as large as the net move. Broken sectors with live counter-rallies, not clean SPT-shorts. **Do not enter short on these from current levels.**
- **NFLX 60m short** — pb 0.98, fails. Same story.
- **AMZN 15m short** — pb 1.03 (counter-bounce > net). Was already flagged as divergent vs daily long; tier grading confirms.
- **GE 15m + 5m short, ORCL 5m short** — drop from A/B to C-tier on 5m. Magnitudes real but pullback discipline broke.

## ✅ Q49 resolved — AMD: continuation, NOT exhaustion

AMD's +36.9%/20d move passes A-gate cleanly: pullback 0.38, closeness 0.97, net_R +6.65. Aggregate stats are inconsistent with climactic-exit profile (which would show pb ≥0.50 with deteriorating closeness). Treat as continuation long, not tightening-stop fade. PLAYBOOK rule 5 stops still apply.

## 📖 Tape read — quality-graded one-liner

**Long-side: bull breadth is broader than pt 33 implied** — adding small-caps (IWM), industrials (CAT), and AAPL to the mega-cap tech leadership. **Short-side: there is no clean SPT-short on daily or 60m. The "energy short" and "NFLX broken" reads from pt 33 are real damage but live bounces — wait for fresh structure before shorting.** ADBE 15m is the single highest-quality short anywhere in the scan.

## 🚫 Caveats

- Same Friday 04-17 close cutoff as pt 33. Monday gaps invalidate, especially smaller TFs.
- Universe is the same 52-name list — not expanded.
- Tier B/C cutoffs are arbitrary midpoints; not yet validated against perR (Q50).

## Follow-ups

- **Q50** (new) — Tier × perR study. Does A-tier outperform B-tier outperform C-tier in actual SPT trades? If yes the gate is informative; if no the tier system is decorative.
- **Q51** (new) — Sector tilt of B-tier longs (industrials + small-caps + defensives) — regime-shift signal or one-day artifact?
- **Q47** carries — cross-TF SPT vs single-TF backtest, now with tier conditioning.

Full vault note: [small-pullback-trend-tier-graded-candidates-2026-04-19.md](https://github.com/zerosumsystems-ui/aiedge-vault/blob/main/Brooks%20PA/concepts/small-pullback-trend-tier-graded-candidates-2026-04-19.md)

---

## Previous — pt 33: strongest-trend recommendations per timeframe

**Run:** 2026-04-19 evening, scheduled-task autonomous. First operational SPT post — takes the 11-rule PLAYBOOK off paper and scans 52 symbols × 5 timeframes for the biggest, cleanest directional trends right now.

Data: Databento `XNAS.ITCH` through Fri 2026-04-17 cash close. Windowed composite score = 0.5·|net_R| + 2.0·closeness-to-extreme + 2.0·pullback-penalty. Sanity filter only (|net_R|≥2, closeness≥0.70) — lists ranked by pure strength, not gate-quality.

## 🎯 Top picks by trade horizon

| Horizon | Dir | Name | Why |
|---------|:---:|------|-----|
| Position (weeks) | 🟢 | **AMD** | +36.9% 20d · closes 97% of range · 0.38 pullback |
| Intraday swing | 🟢 | **MRK** | Only name on 4 consecutive TFs (60m→5m) all up |
| Hours | 🟢 | **MRK** +9.84 netR on 30m | Cleanest large-cap 30m trend in the scan |
| Scalp | 🟢 | **WMT** +18.69 netR on 5m | Strongest single-TF reading in the entire scan |
| Intraday short | 🔴 | **ADBE** (3-TF short) | Cleanest short across 30m/15m/5m |
| Position short | 🔴 | **CVX** | Daily −8.9% · closeness 0.83 · energy selloff |

## 📊 Strongest trends per timeframe

### Daily (weeks)
**🟢 LONGS:** AVGO (+29.6%) · **AMD (+36.9%)** · AMZN (+21.1%) · BAC (+13.6%) · SPY (+8.8%)
**🔴 SHORTS:** **CVX (−8.9%)** · COP (−9.0%) · XOM (−8.4%) — **energy is the one weak sector**

### 60m (1-3 sessions)
**🟢 LONGS:** **DIA** · DIS · MRK · CAT · HD (broad tape + defensives + industrials)
**🔴 SHORT:** **NFLX** (−10% with big magnitude — loudest broken single-name)

### 30m (hours)
**🟢 LONGS:** **MRK +9.84 netR** · DIA · HD · DIS · UNH (defensive rotation dominates)
**🔴 SHORTS:** **ADBE** · ORCL (software-complex breaking)

### 15m (tens of minutes)
**🟢 LONGS:** **WMT** · MRK · COST · DIS · PFE (defensive + retail)
**🔴 SHORTS:** **ORCL** · ADBE · CRM · AMZN(!) · GE (AMZN 15m-short vs daily-long = intraday pullback inside bull, don't fade)

### 5m (minutes) — strongest readings in the whole scan
**🟢 LONGS:** **WMT +18.69 netR** · COST · MRK · UNH · CVX(!)
**🔴 SHORTS:** **GE −10.42 netR** · CRM · ADBE · ORCL · MU

## ⭐ Cross-timeframe stars (signal on ≥ 2 TFs)

| Symbol | Timeframes | Conviction |
|--------|-----------|:----------:|
| **MRK** long | 60m · 30m · 15m · 5m | ★★★★ |
| **DIS** long | 60m · 30m · 15m | ★★★ |
| **ORCL** short | 30m · 15m · 5m | ★★★ |
| **ADBE** short | 30m · 15m · 5m | ★★★ |
| **HD** long | 60m · 30m | ★★ |
| **DIA** long | 60m · 30m | ★★ |
| **UNH** long | 30m · 5m | ★★ |
| **WMT** long | 15m · 5m | ★★ |
| **COST** long | 15m · 5m | ★★ |
| **CRM** short | 15m · 5m | ★★ |
| **GE** short | 15m · 5m | ★★ |

## 📖 The tape read in one line

**Broad tape is bull** (AMD/AVGO/AMZN/BAC daily + SPY/QQQ). **Under the hood the bid rotated into defensives** (MRK/DIS/HD/UNH/WMT/COST) **and out of enterprise software + select momentum** (ORCL/ADBE/CRM/GE) in Friday afternoon. **Energy is the one weak broad sector** on the daily (CVX/COP/XOM all −8-9%).

## ⚠️ Divergence flags (do NOT fade these)

- **AMZN** — daily long, 15m short. Intraday pullback inside bull. Don't short the dip.
- **CVX** — daily short, 5m long. Counter-trend bounce inside a broken sector. Don't buy the bounce.

## 🚫 Caveats

- Last close is Fri 04-17. Monday open can invalidate — especially the 5m list.
- No live `day_type` label (PLAYBOOK rule 2); live scanner adds that Monday morning.
- Universe is 52 hand-picked names; smaller active SPTs outside this list are invisible.
- Pullback-pct is unbounded on smaller moves — read netR + closeness first.

## Follow-ups

- **Q47** — cross-timeframe-SPT backtest. Does requiring SPT on ≥2 TFs lift perR above the PLAYBOOK?
- **Q48** — software SPT-short regime persistence. Re-scan Monday EOD for ORCL/ADBE/CRM cluster.
- **Q49** — AMD +36.9%/20d is extreme. Is it a mature-SPT (late-stage) or continuation (early-stage)? Flag for manual review.

Full vault note: [small-pullback-trend-multi-tf-candidates-2026-04-19.md](https://github.com/zerosumsystems-ui/aiedge-vault/blob/main/Brooks%20PA/concepts/small-pullback-trend-multi-tf-candidates-2026-04-19.md)

---

## Previous — pt 32: rule-11 setup-level concentration

(pt 32 headline archived in [ARCHIVE.md](ARCHIVE.md)). Zero-compute decomposition of pt 27 §5. Rule 11's +0.19 R/trade lift concentrates at H1-long (+0.30) and L1-short (+0.15); neutral on H2-long; L2-short n=3 artifact. Conditional-variant analysis: C3ʹ (rule 11 on H1-long + L1-short only) gives identical perR at +11% throughput but **widens DD floor 50%** (−2 → −3). Decision: keep uniform C3. Rule 11's primary operational value is DD improvement.
