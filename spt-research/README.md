# SPT research — phone briefing

Autonomous small-pullback-trend research notes, phone-readable. Long-form notes live in [`notes/`](notes/); phone-friendly PDFs in [`pdfs/`](pdfs/); the latest TL;DR is at the top of this README.

> 📚 **Full research archive — [ARCHIVE.md](ARCHIVE.md)** lists every SPT note ever written (33 notes in reading order + the PLAYBOOK + the INDEX). Canonical mirror at the aiedge-vault: [github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts](https://github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts).

**Last updated:** 2026-04-19 evening · pt 33 + Monday-open watchlist

> 📅 **Monday 2026-04-20 open — actionable watchlist:** [notes/small-pullback-trend-monday-watchlist-2026-04-20.md](notes/small-pullback-trend-monday-watchlist-2026-04-20.md). One-table view + PLAYBOOK entry gates + cross-TF stars + gap-risk checklist. Start here before the 10:30 ET entry window.

## Latest — pt 33: strongest-trend recommendations per timeframe

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
