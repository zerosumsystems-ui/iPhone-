---
kind: phone_card
created: 2026-04-20T05:40-04:00
data_anchor: 2026-04-17 cash close (Fri) — unchanged since pt 37 (00:05 ET)
source_notes: pt 37 (unified), pt 38 (clusters), pt 39 (one-pager), pt 40 (fallback + clock)
market_opens: 09:30 ET Mon 2026-04-20 (~3h 50m from now)
---

# SPT Monday picks — phone card · 2026-04-20

Fifth `small-pullback-trend-research` run today (after pts 37/38/39/40 at 00:05 / 01:07 / 02:15 / 03:35 ET). **Mac mini under extreme load (125 load-avg, ~14 MB free) — file-read-only, zero new compute, zero API calls.** No market data has changed since Fri 2026-04-17 close. This card mirrors the Monday recommendations to the phone-briefings repo, which had been stale at pt 34 since 2026-04-19. Pure re-presentation.

## 🎯 Per-timeframe picks

| TF | 🟢 LONG | 🔴 SHORT |
|---|---|---|
| **Daily** (weeks) | **CROX** A · pb 0.24 · cls 0.97 · $105.74 | **none** (0 A/B shorts in 244 names) |
| **60m** (1-3 sessions) | **FDX** A · pb 0.34 · cls 0.97 · $391.97 | **none** |
| **30m** (hours) | **FDX** A · pb 0.25 · cls 0.94 | **MPC** A · pb 0.39 · cls 0.84 · $213.69 |
| **15m** (scalp-to-swing) | **UUP** A · pb 0.26 · cls 0.96 · $27.34 | **DKNG** A · pb 0.39 · cls 0.91 · $22.81 |
| **5m** (scalp) | **TGT** A · pb 0.33 · cls 0.96 · $127.70 | **DKNG** A · pb 0.34 · cls 0.91 |

**Legend.** A-tier = |netR| ≥ 2 · pullback ≤ 40% · closeness-to-extreme ≥ 75%. Pullback = fraction of trend window retraced. Closeness = 1.0 means closed at the extreme.

**Single-best any-TF long:** **FDX** — 3 A-tier intraday cells (60m / 30m / 15m) · standalone transport exposure.
**Single-best any-TF short:** **DKNG** — 2 A-tier cells (15m / 5m) · outside the software-short correlation cluster.

## 🪜 Fallback ladder (if primary killed)

Move down the moment a kill condition fires on the active pick. Don't skip tiers — B still beats no trade. (Daily / 60m short ladders are genuinely empty in the 244-name merged universe.)

### Long ladders

| TF | Primary | Fallback 1 | Fallback 2 | Fallback 3 |
|---|---|---|---|---|
| Daily | CROX (5.49) | MRVL (5.12) | INTC (4.40) | FCX (4.39) |
| 60m | FDX (7.55) | URBN (5.81) | EQIX (5.12) | XLRE (5.00) |
| 30m | FDX (5.80) | URBN (5.41) | EMR (5.38) | INDA (4.63) |
| 15m | UUP (7.20) | ADI (5.25) | TGT (5.17) | FDX (4.06) |
| 5m | TGT (10.04) | XLRE (8.80) | SPG (8.65) | AMT (8.48) |

### Short ladders

| TF | Primary | Fallback 1 | Fallback 2 |
|---|---|---|---|
| Daily | **empty** | — | — |
| 60m | **empty** | — | — |
| 30m | MPC (4.29) | *(no second A-tier; pull 30m-short entirely if killed)* | — |
| 15m | DKNG (3.71) | ADBE (3.12) | ORCL B (2.76) |
| 5m | DKNG (7.95) | EWZ B (6.22) | DAL B (5.02) |

## 🧯 Kill conditions — what drops a pick

- **Gap-against ≥ 1%** in the pick's direction → drop (rule 4 gap_direction auto-fires).
- **Pre-market move > 5%** on the underlying (earnings, halt, M&A) → drop outright.
- **Day-type degrades** by 10:00 ET to TR / tight_TR / trending_TR / undetermined → drop all except UUP (UUP is DXY-macro, separate day-type logic).
- **Pick triggers as 1st ticker-day detection** at open → skip per rule 7; wait for the 2nd.
- **Signal-bar opp_tail ≥ 0.25** → drop this specific signal; primary still eligible if it re-fires clean.
- **Rule-8 trip** (≥ 3 stack trades fired, < 2 won by 12:30 ET) → stop trading today regardless of ladder.

## 🏢 Cluster correlation — one-per-theme

If two cluster-mates both fire, take ONE and count 1.5R exposure (not 2R).

- **REIT:** XLRE → SPG → EQIX → AMT
- **Retail:** TGT → URBN → CROX (CROX is daily; others intraday)
- **Semi:** MRVL → ADI → ON → INTC → AVGO
- **Software-short:** ADBE → CRM → ORCL → TEAM → HUBS (DKNG is **not** in this cluster)

**Standalone uncorrelated names:** FDX, CSCO, WMT, MRK, UUP, MPC.

## 🕐 ET action clock

```
08:00  futures wake-up       → read /ES, /NQ, /RTY overnight move
08:30  PRE-MARKET GATE       → kill picks with gap-against ≥ 1% or pre-mkt > 5%
                             → sector ETF read: XLRE / XLK / XLE / XRT
09:25  TLDR re-check         → 30 seconds, confirm pick still alive
09:30  OPEN — do NOT enter   → rule 6 floor active
10:00  day_type read         → require TFO or S&C (UUP exempt)
10:30  LONG entry opens      → H1/H2 gate check
11:30  SHORT entry opens     → L1/L2 gate check
12:30  mid-session rule-8    → ≥3 trades + <2 wins ⇒ stop for the day
14:15  LAST ENTRY            → 14:45+ is the worst band (pt 8)
15:00  runner check          → hybrid rule 9 runners should be ≥3R MFE
15:50  ADBE short hard-close if taken (pre-announced Adobe Summit Tue 4/21)
16:00  CASH CLOSE — log ACTIVITY
```

## 📏 Economics (if PLAYBOOK gates clear)

**C3 stack** (rules 1-9 + rule 10 swing2-shorts-stop + rule 11 opp_tail<0.25): **n=83, WR 77.1%, perR +1.909R [+1.50, +2.34 bootstrap CI], max DD −2.00R, 10.2 trades/mo, +19.5R/mo.** Walk-forward positive in all 3 chronological thirds. LOO-week min +1.61. Source: `scanner/scratch/spt_full_playbook_backtest_2026_04_19.py` · backtest ran at 04:15 ET today.

## 📌 Caveats

- ≥ 60h from Fri close to Mon open — aging risk on every pick, compounds with any weekend news.
- Tier-A grade is chart-only; calendar overlay is NOT in the score (ADBE Summit Tue forces hard-close on any ADBE short before 15:50 ET).
- 5m and 15m UUP are an ETF; execution liquidity differs from single-name equity. **Size ¼** on UUP.
- DKNG is in a regulated-news sector (state gambling, sports book volatility) — treat A-tier short as a setup, not a thesis.
- Short-side 89% WR on the C3 full-PLAYBOOK backtest likely carries bull-regime tailwind (Aug-2025 → Apr-2026 was ~99% bull tape). Re-measure in a bear leg.

## 📄 Source of truth

- **Canonical vault note:** [[../../aiedge-vault/Brooks%20PA/concepts/small-pullback-trend-pre-open-one-pager-2026-04-20|pt 39 — pre-open one-pager]] + [[small-pullback-trend-monday-fallback-and-clock-2026-04-20|pt 40 — fallback + clock]]
- **Thematic cluster cut:** [[../../aiedge-vault/Brooks%20PA/concepts/small-pullback-trend-monday-desk-digest-2026-04-20|pt 38]]
- **Unified 244-name scan:** [[../../aiedge-vault/Brooks%20PA/concepts/small-pullback-trend-unified-recommendations-2026-04-20|pt 37]]
- **11-rule operational stack:** [[../../aiedge-vault/Brooks%20PA/concepts/small-pullback-trend-PLAYBOOK]]
- **Full 40-note research arc:** [[../../aiedge-vault/Brooks%20PA/concepts/small-pullback-trend-INDEX]]

## 🤖 Why this card exists

Scheduled task `small-pullback-trend-research` fires on a recurring cadence. Today's earlier runs (pt 37-40) already answered the recommendation question on the vault side. The iPhone briefings repo had been stale at pt 34 since 2026-04-19 — Will's phone view didn't have the Monday picks. This card closes that gap. One phone-readable file, no new research, no API calls.

Tue pre-open note (pt 41 or similar) will fire at ~02:00 ET Tue 2026-04-21.
