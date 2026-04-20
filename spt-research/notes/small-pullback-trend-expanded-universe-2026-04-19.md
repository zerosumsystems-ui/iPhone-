# SPT expanded-universe scan — 2026-04-19 (pt 36)

**Operational deliverable.** Scheduled-task autonomous run. Pt 33/34/35 ran on a curated 52-name universe; every prior note flagged "a cleaner SPT outside this list can outrank anything here" as a caveat. This part retires that caveat: a **194-name expansion** (sector leaders, style ETFs, REITs, fintech, transports, biotech, EM/country ETFs) scanned with the same `spt_score` + A/B/C tier gates. Data cutoff: Fri 2026-04-17 cash close.

**Top-line result:** the expansion produces **stronger candidates than pt 33 on every timeframe**. Target, FedEx, and Simon Property replace WMT/MRK/AMD at the top of the cross-TF leaderboard. Real-estate is the new, previously-invisible sector leg. DraftKings is a brand-new A-tier short that nearly matches ADBE's quality.

**Run artifacts:** `/tmp/spt_scan_pt36/scan_expanded.py` (reproducible), `raw.json`, `tiered.json`, `cross_tf.json`, `summary.json`. 194 new symbols × 5 TFs = 970 scans, 10 fetch errors (VIX didn't resolve on XNAS.ITCH + 9 low-liquidity minute-data gaps).

## One pick per horizon — pt 36 universe vs. pt 35 pick

Where pt 36 outranks pt 35 on combined score, pt 36 wins the slot.

| Horizon | Dir | Pt 35 winner | Pt 36 winner | New top combined | Verdict |
|---|:---:|---|---|---:|---|
| **Daily — position** | 🟢 LONG | AMD (A, 4.00) | **CROX** (A) | **5.49** | 🆕 CROX overtakes |
| **Daily — short** | 🔴 SHORT | (none clean) | (none clean) | — | **Zero daily shorts at any tier** confirmed across 246 names |
| **60m — intraday swing** | 🟢 LONG | DIA (B, 3.54) | **FDX** (A) | **7.55** | 🆕 FDX overtakes by 2× |
| **60m — short** | 🔴 SHORT | (none clean) | (none clean C only) | — | No B-tier short on 60m in the full 246-name scan |
| **30m — intraday hours** | 🟢 LONG | MRK (B, 4.91) | **FDX** (A) | **5.80** | 🆕 FDX overtakes, A-tier |
| **30m — short** | 🔴 SHORT | ADBE (C, 0.47) | **MPC** (A) | **4.29** | 🆕 **MPC** — first A-tier 30m short anywhere |
| **15m — scalp-to-swing** | 🟢 LONG | WMT (B, 4.78) | **UUP** (A) | **7.20** | 🆕 dollar-index ETF — macro bid |
| **15m — short** | 🔴 SHORT | ADBE (A, 3.12) | **DKNG** (A) | **3.71** | 🆕 **DKNG** overtakes ADBE |
| **5m — scalp** | 🟢 LONG | WMT (B, 10.44) | **ARKW** (B, 28.43)* | **28.43** | ⚠️ outlier, see caveat |
| **5m — short** | 🔴 SHORT | ADBE (B, 3.54) | **DKNG** (A) | **7.95** | 🆕 **DKNG A-tier** overtakes ADBE |

*ARKW 5m is a single near-vertical push (+65 netR) similar in shape to pt 33's WMT 5m outlier. **Do not size off raw score** — treat as "late-stage with rule-11 opp_tail risk on any Monday pullback."

## Per-TF top 5 — expanded universe only

### Daily (position, weeks)

🟢 **LONGS (all A-tier unless noted)** — CROX 5.49 · MRVL 5.12 · INTC 4.40 · FCX 4.39 · AFRM 4.28

🔴 **SHORTS** — only 2 C-tier names (UNG 0.95, LCID 0.76); **zero A or B-tier daily shorts** in 194 names. **The daily-short desert is structural, not a universe artifact.**

### 60m (intraday swing, 1–3 sessions)

🟢 **LONGS** — FDX 7.55 A · URBN 5.81 A · EQIX 5.12 A · XLRE 5.00 A · TGT 4.94 A

🔴 **SHORTS** — MPC 0.73 C · PSX 0.65 C · LMT 0.49 C; **no B-tier short on 60m.**

### 30m (intraday hours)

🟢 **LONGS** — FDX 5.80 A · URBN 5.41 A · EMR 5.38 A · TGT 4.89 B · INDA 4.63 A

🔴 **SHORTS** — **MPC 4.29 A** · RSG 2.55 B · PSX 2.35 C · LMT 2.09 B · DKNG 1.57 C

### 15m (scalp-to-swing)

🟢 **LONGS** — UUP 7.20 A · ADI 5.25 A · TGT 5.17 A · FDX 4.06 A · KMI 3.85 B

🔴 **SHORTS** — **DKNG 3.71 A** · EWZ 2.73 B · TTD 2.18 B · FTNT 1.91 C · DAL 1.74 C

### 5m (scalp, minutes)

🟢 **LONGS** — ARKW 28.43 B* · TGT 10.04 A · XLRE 8.80 A · SPG 8.65 A · KMI 8.53 B

🔴 **SHORTS** — **DKNG 7.95 A** · EWZ 6.22 B · DAL 5.02 B · LMT 4.41 B · TTD 2.82 C

## Cross-TF stars — pt 36 universe

Summed `combined` across TFs where symbol passed a tier gate. Cutoff: ≥ 2 TFs hit.

### LONGS

| Symbol | TFs hit | Tiers | Σ combined | Read |
|---|---|---|---:|---|
| 🟢 **TGT** | 5 (all) | B · A · B · A · A | **26.92** | **Dominant cross-TF long; 3 A-tier intraday.** Target specifically — not sector ETF. |
| 🟢 **FDX** | 4 (60m + 30m + 15m + 5m) | A · A · A · B | **23.51** | **Triple A-tier intraday.** Transport bid — clearest single-name leg. |
| 🟢 **XLRE** | 5 (all) | B · A · B · B · A | **23.13** | **REIT sector rotation** — previously invisible with pt 33 universe. |
| 🟢 **SPG** | 5 (all) | C · A · A · B · A | **21.73** | Individual REIT confirming XLRE sector-leg read. |
| 🟢 **ADI** | 5 (all) | C · B · B · A · A | **21.03** | Second semi-leg (alongside AVGO from pt 33). Cleaner on intraday than daily. |
| 🟢 **URBN** | 5 (all) | B · A · A · B · A | **20.53** | Retail momentum — dangerous because of low float / earnings risk. |
| 🟢 **MRVL** | 5 (all) | A · C · B · B · B | **16.46** | Third semi; **only 5-TF long with daily A-tier** in pt 36 — weekly-hold candidate. |
| 🟢 **AMT** | 4 (60m + 30m + 15m + 5m) | B · B · A · A | **16.12** | Tower REIT — reinforces REIT leg. |
| 🟢 **CSCO** | 5 (all) | C · A · C · A · A | **15.16** | Legacy tech; quiet and clean. |
| 🟢 **ON** | 5 (all) | B · B · C · B · B | **13.10** | Semi — same leg as ADI/MRVL; four B-tiers is rare (disciplined trend). |
| 🟢 **EQIX** | 3 (daily + 60m + 30m) | A · A · B | 12.53 | Data-center REIT — weekly-hold adjacent to XLRE. |
| 🟢 **YUM** | 3 (30m + 15m + 5m) | C · A · A | 12.46 | QSR defensive; late-session push. |
| 🟢 **KMI** | 2 (15m + 5m) | B · B | 12.38 | Pipeline — energy infra counter-trend to MPC/PSX shorts. |
| 🟢 **ETN** | 5 (all) | C · B · A · C · C | 11.92 | Industrial electricals. |
| 🟢 **INDA** | 3 (daily + 60m + 30m) | C · B · A | 10.49 | India ETF — EM bid. |

### SHORTS

| Symbol | TFs hit | Tiers | Σ combined | Read |
|---|---|---|---:|---|
| 🔴 **DKNG** | 3 (30m + 15m + 5m) | C · A · A | **13.23** | **New A-tier short cluster.** Matches ADBE's quality (pt 35 called ADBE "only A-tier short" — that's now retired). |
| 🔴 **EWZ** | 2 (15m + 5m) | B · B | 8.95 | Brazil ETF — EM weakness paired with INDA strength (rotation, not macro-bear). |
| 🔴 **LMT** | 4 (60m + 30m + 15m + 5m) | C · B · C · B | 7.73 | Defense — unusual multi-TF short. |
| 🔴 **MPC** | 3 (60m + 30m + 5m) | C · A · C | 7.32 | **Refiner** — reinforces pt 33's XOM/CVX/COP daily-short thesis but with clean intraday structure. |
| 🔴 **TTD** | 3 (30m + 15m + 5m) | C · B · C | 6.83 | AdTech weakness — pairs with CRM/ADBE software-complex read. |
| 🔴 **DAL** | 2 (15m + 5m) | C · B | 6.76 | Airline — note: unusual because FDX (transport) is the top long. Airlines ≠ parcels-transport. |

## Merged top-10 longs — pt 33 + pt 36 combined

Re-ranking by Σ combined across timeframes. This is the canonical Monday watchlist going forward.

| Rank | Symbol | TFs | Tiers | Σ combined | Source |
|---:|---|---|---|---:|---|
| 1 | **ARKW** | 3 (60m + 30m + 5m) | B · B · B | 33.36 | pt 36 |
| 2 | **TGT** | 5 | B · A · B · A · A | 26.92 | **pt 36 new** |
| 3 | **FDX** | 4 | A · A · A · B | 23.51 | **pt 36 new** |
| 4 | **XLRE** | 5 | B · A · B · B · A | 23.13 | **pt 36 new** |
| 5 | **SPG** | 5 | C · A · A · B · A | 21.73 | **pt 36 new** |
| 6 | **ADI** | 5 | C · B · B · A · A | 21.03 | **pt 36 new** |
| 7 | **URBN** | 5 | B · A · A · B · A | 20.53 | **pt 36 new** |
| 8 | **WMT** | 5 | — · — · — · B · B | 17.65 | pt 33/35 |
| 9 | **AMT** | 4 | B · B · A · A | 16.12 | **pt 36 new** |
| 10 | **MRVL** | 5 | A · C · B · B · B | 16.46 | **pt 36 new** |

**8 of the top 10 are NEW names that the pt 33 universe missed.** The curated 52 was mid-cap-tech + mega-cap-tech + a handful of financials/defensives; it was blind to transports, REITs, analog semis, and retail momentum.

## Merged top shorts — pt 33 + pt 36 combined

| Rank | Symbol | TFs | Tiers | Σ combined | Source |
|---:|---|---|---|---:|---|
| 1 | **DKNG** | 3 | C · A · A | 13.23 | **pt 36 new** |
| 2 | **EWZ** | 2 | B · B | 8.95 | **pt 36 new** |
| 3 | **LMT** | 4 | C · B · C · B | 7.73 | **pt 36 new** |
| 4 | **MPC** | 3 | C · A · C | 7.32 | **pt 36 new** |
| 5 | **ADBE** | 3 | C · A · B | 7.13 | pt 33/35 |
| 6 | **TTD** | 3 | C · B · C | 6.83 | **pt 36 new** |
| 7 | **DAL** | 2 | C · B | 6.76 | **pt 36 new** |
| 8 | **CRM** | 2 | B · B | 6.06 | pt 33/35 |

**Short-side expansion is bigger than long-side.** Pt 35 explicitly called ADBE "the only A-tier short anywhere" — expansion found 3 more A-tier shorts (DKNG 15m/5m, MPC 30m) across different sectors. **The no-short-on-daily-or-60m finding holds**: 0 A or B-tier shorts on daily/60m across 246 names — the structural signal is real.

## New sector reads

Cross-TF long clusters reveal sector rotation that pt 33 couldn't see:

1. **REIT leg** — XLRE · SPG · AMT · EQIX (4 names, multi-TF). Interest-rate-sensitive longs bid. NEW.
2. **Transport leg** — FDX (parcel) alone (UPS didn't tier). Narrow but very clean.
3. **Semi breadth** — ADI · MRVL · ON (pt 36) + AVGO · AMD (pt 33) = **5-name semi leg**. Broad tech-rally confirmation, not narrow mega-cap.
4. **Retail leg** — URBN · TGT (pt 36) + WMT · COST (pt 33) = 4-name retail leg. QSR bid (YUM) adjacent.
5. **EM rotation** — INDA long (India) vs EWZ short (Brazil). Rotation trade, not broad EM weakness.

**Sector read is bullish with defensive-ish leadership.** REITs + transport + retail + semis = broad rally. Bearish pockets confined to refiners (MPC), defense (LMT), gaming (DKNG), AdTech (TTD), and Brazil. No broad-tape bear case visible at daily/60m.

## Execution — gates unchanged

All pt 34/35 PLAYBOOK gates apply unchanged. A tier-A candidate on pt 36 **is not an entry** — it's a watchlist. Entry still requires a live H1/H2/L1/L2 fire inside the 10:30-14:15 ET window with urgency ≥ 4 and rule-11 `opp_tail < 0.25`.

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
 H1/H2 long  → 5R
 L1/L2 short → 4R cap
```

Expected C3-stack economics when gates clear: **n ≈ 18/mo · WR 74.6% · +1.84R/trade · max DD −2R**.

## Monday 2026-04-20 action — which new names deserve priority

Adding these to the existing pt 35 + late-memo queue:

### Add to long book (replaces or supplements pt 35 picks)

- **TGT 30m/15m/5m** — ★★★★ (higher conviction than MRK; 3-A tier). Watch for H1/H2 in the 10:30-14:15 window.
- **FDX 60m/30m/15m** — ★★★ (beats DIA on 60m). Clean transport leg.
- **XLRE & SPG** — ★★★ sector-rotation longs. Buy-the-pullback on either — note XLRE is an ETF (lower R/bar than single names).
- **ADI & MRVL** — ★★ semi adjunct to AVGO.
- **CROX** — ★★ single-name daily A-tier; watch for pullback to 20-MA.

### Add to short book

- **DKNG 30m/15m/5m** — ★★★ **equals ADBE quality** (pt 35 was wrong to call ADBE unique). Apply rule 10 swing-2 stop. 11:30-14:15 short window.
- **MPC 30m** — ★★★ first A-tier 30m short. Refiner — reinforces CVX/XOM/COP broken-energy daily read with clean intraday structure.
- **LMT / TTD / DAL** — ★★ B-tier short options.

### Demote

- **ADBE** — still an A-tier short on 15m, but **no longer unique**. DKNG and MPC match or beat it. De-concentrates the short book.

## Caveats

- **194-name universe is now 246 total** when merged with pt 33. Broader but still not full S&P 500 (~500) or Russell 1000 (~1000). Further expansion could find more — diminishing returns expected.
- **Data cutoff Fri 2026-04-17 cash close.** Monday 2026-04-20 gap or overnight news can invalidate any 15m/5m pick. The live aiedge-scanner remains canonical for intraday execution.
- **ARKW 5m +65 netR is an outlier.** Late-stage vertical push — treat as pt 33 treated WMT 5m. Size down, rule-11 mandatory.
- **TGT +19.7% 20-day / FDX 20-day n/a** — check next for earnings risk before Monday open.
- **No live day_type / urgency / `always_in` gating here.** Ranking ≠ entry. PLAYBOOK gates are mandatory.
- **No behavioural change vs pt 35** for tools or parameters — same `spt_score`, same tier thresholds. Only the universe expanded.
- **DKNG A-tier short** — verify Monday-eve for sports/gaming news flow that can gap it against the short.

## Related

- [[small-pullback-trend-multi-tf-candidates-2026-04-19]] — pt 33 (original 52-name scan)
- [[small-pullback-trend-tier-graded-candidates-2026-04-19]] — pt 34 (tier grading on pt 33)
- [[small-pullback-trend-recommendations-by-timeframe-2026-04-19]] — pt 35 (combined-score ranking on pt 33)
- [[small-pullback-trend-monday-watchlist-2026-04-20]] — pt 33-based watchlist
- [[small-pullback-trend-PLAYBOOK]] — 11-rule operational stack
- [[small-pullback-trend-INDEX]] — full reading order
- `/tmp/spt_scan_pt36/scan_expanded.py` — reproducible scan script
- `/tmp/spt_scan_pt36/summary.json` — machine-readable output
