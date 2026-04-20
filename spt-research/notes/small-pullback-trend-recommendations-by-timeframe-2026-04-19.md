# SPT recommendations by timeframe — 2026-04-19 (pt 35)

**Operational deliverable.** Scheduled-task output consolidating [[small-pullback-trend-multi-tf-candidates-2026-04-19|pt 33]] (magnitude) and [[small-pullback-trend-tier-graded-candidates-2026-04-19|pt 34]] (quality tiers) into a single per-timeframe recommendation rank. One top long + one top short per TF, plus cross-TF stars.

**Run:** 2026-04-19 Sun, autonomous. No new API calls — reuses `/tmp/spt_scan/raw.json` (pt 33) and `/tmp/spt_scan_pt34/tiered.json` (pt 34). Combined-rank script at `/tmp/spt_scan_combined/rank_combined.py`; output at `/tmp/spt_scan_combined/ranked.json`. Cutoff: Fri 2026-04-17 cash close (Monday is tomorrow).

## Scoring — combined magnitude × quality

`combined = |net_R| × extreme_closeness × max(0, 1 − pullback_pct)`

- **Magnitude term** (`|net_R| × closeness`) — pt 33's composite: how large is the net push, measured in ATR, adjusted for whether it closed at the extreme of the window.
- **Quality term** (`1 − pullback_pct`) — Brooks-canonical "small pullback" factor: penalizes deep counter-moves inside the window. At `pullback_pct = 1.0` (counter-bounce equals net move), quality = 0 — score collapses, matching pt 34's rejection of those names.

Tier annotation from pt 34 (A ≤ 0.40 pb; B ≤ 0.60 pb; C ≤ 0.90 pb; — = failed all gates).

## One pick per horizon

| Horizon | Dir | Symbol | combined | tier | netR / pb / cls | Last |
|---|:---:|---|---:|:---:|---|---:|
| **Daily — position (weeks)** | 🟢 LONG | **AMD** | 4.00 | A | +6.65 / 0.38 / 0.97 | $277.25 |
| **Daily — short** | 🔴 SHORT | **(none clean)** | — | — | all three fail quality | — |
| **60m — intraday swing** | 🟢 LONG | **DIA** | 3.54 | B | +7.69 / 0.41 / 0.78 | $494.21 |
| **60m — short** | 🔴 SHORT | **(none clean)** | — | — | NFLX pb 0.98 | — |
| **30m — intraday hours** | 🟢 LONG | **MRK** | 4.91 | B | +9.84 / 0.48 / 0.96 | $119.07 |
| **30m — short** | 🔴 SHORT | **ADBE** | 0.47 | C | −4.24 / 0.88 / 0.93 | $244.38 |
| **15m — scalp-to-swing** | 🟢 LONG | **WMT** | 4.78 | B | +8.87 / 0.45 / 0.98 | $127.47 |
| **15m — short** | 🔴 SHORT | **ADBE** | 3.12 | **A** | −5.59 / 0.38 / 0.90 | $244.38 |
| **5m — scalp** | 🟢 LONG | **WMT** | 10.44 | B | +18.69 / 0.43 / 0.98 | $127.47 |
| **5m — short** | 🔴 SHORT | **ADBE** | 3.54 | B | −7.71 / 0.49 / 0.90 | $244.38 |

**Reading the `combined` column:** higher is better. A score above ~3.0 is a genuinely strong trend with disciplined pullback behaviour. Scores under ~1.0 usually mean the "trend" is actually a chop-filled drift.

## Per-TF runners-up (top 3 long, top 3 short)

### Daily (position, weeks)
- 🟢 **AMD** 4.00 A · **AVGO** 3.99 A · **AMZN** 3.41 A · **BAC** 2.69 A
- 🔴 CVX 0.07 — · XOM/COP 0.00 — · *no clean short on daily at any tier*

### 60m (intraday swing, 1–3 sessions)
- 🟢 **DIA** 3.54 B · **DIS** 3.44 B · **CAT** 3.10 B · SPY 3.04 A
- 🔴 NFLX 0.11 — · *no clean short on 60m at any tier*

### 30m (intraday hours)
- 🟢 **MRK** 4.91 B · **DIA** 4.24 C · **HD** 4.22 B · DIS 3.54 B
- 🔴 **ADBE** 0.47 C · ORCL 0.00 — · *no B-tier short on 30m*

### 15m (scalp-to-swing)
- 🟢 **WMT** 4.78 B · **MRK** 2.43 B · **DIS** 1.75 B · GOOGL 1.35 C
- 🔴 **ADBE** 3.12 **A** · **ORCL** 2.76 B · **CRM** 2.57 B

### 5m (scalp, minutes)
- 🟢 **WMT** 10.44 B · **MRK** 4.52 B · **UNH** 4.23 B · GOOGL 3.20 B
- 🔴 **ADBE** 3.54 B · **CRM** 3.50 B · GE 3.17 C

## Cross-TF stars — combined score summed across TFs

These are the highest-confidence names: visible SPT structure at multiple scales means a clean run with low noise.

### LONGS

| Symbol | TFs hit | Tiers | Σ combined | Read |
|---|---|---|---:|---|
| 🟢 **WMT** | 5 (daily + 60m + 30m + 15m + 5m) | — · — · — · B · B | **17.65** | Dominant cross-TF long; structural vertical on Friday. Watch for rule 11 `opp_tail` on any Monday pullback. |
| 🟢 **MRK** | 4 (60m + 30m + 15m + 5m) | B · B · B · B | **14.41** | Highest uniform-quality cross-TF long. Every intraday TF at B. |
| 🟢 **UNH** | 5 (all) | — · B · B · C · B | 11.66 | Healthcare rotation leg, broader-tape than pt 33 suggested. |
| 🟢 **DIS** | 5 (all) | — · B · B · B · C | 11.43 | 4-TF B-grade, quietly clean entertainment bid. |
| 🟢 **DIA** | 3 (daily + 60m + 30m) | B · B · C | 9.91 | Index itself is in SPT — breadth tailwind for single-name longs. |
| 🟢 **AVGO** | 5 (all) | **A** · B · B · — · C | 9.32 | Only 5-TF long with an **A** on daily. Cleanest weekly position long. |
| 🟢 **CAT** | 5 (all) | B · B · B · C · — | 7.79 | Industrial leg; not the biggest but 3 × B grade. |
| 🟢 **SPY** | 3 (daily + 60m + 30m) | C · A · B | 7.76 | 60m A-tier index — directional baseline. |
| 🟢 **IWM** | 3 (daily + 60m + 30m) | B · B · C | 7.25 | Small-caps are participating — NEW breadth read vs pt 33. |
| 🟢 **HD** | 5 (all) | C · B · B · — · — | 7.18 | Retail-adjacent defensive. |

### SHORTS

| Symbol | TFs hit | Tiers | Σ combined | Read |
|---|---|---|---:|---|
| 🔴 **ADBE** | 3 (30m + 15m + 5m) | C · **A** · B | **7.13** | **The only A-tier short anywhere in the scan.** Cleanest short by a wide margin. |
| 🔴 **CRM** | 2 (15m + 5m) | B · B | 6.06 | Second enterprise-software leg. |
| 🔴 **ORCL** | 3 (30m + 15m + 5m) | — · B · C | 3.91 | Real but 30m/5m pullback too deep for B-grade. |
| 🔴 **GE** | 2 (15m + 5m) | — · C | 3.17 | Momentum exhaustion; C-only — wait for next pullback. |

## Divergences — do NOT fade the higher TF

Same counter-trend intraday bounces pt 33 flagged; quality grading confirms:

- **AMZN** — daily LONG (A-tier, +21.1% 20d) / 15m SHORT (pb 1.03, fails all tiers). Monday weakness = pullback in daily long, not a short entry.
- **CVX** — daily SHORT (failed quality) / 5m LONG (pb 0.96, failed quality). Sector is broken; bounce doesn't qualify either way. Observe only.

## No-short-zone on daily & 60m

Key structural finding: **zero short candidates at any tier on daily or 60m.** CVX/COP/XOM (daily) and NFLX (60m) are all broken sectors with live counter-bounces — the intra-window bounce was as large as the net move. If you want bearish exposure with a multi-session hold, **there is no clean SPT-short setup right now.** The short side is intraday-only, and even there only ADBE/CRM/ORCL cluster survives B-tier grading.

## Execution checklist (applies to all picks)

Operational gates from the 11-rule [[small-pullback-trend-PLAYBOOK|PLAYBOOK]] — these apply on top of the candidate list above. A pick on this list **is not an entry signal**; it's a watchlist. Entry still requires a live H1/H2/L1/L2 fire that passes the gates.

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

## Caveats

- Data cutoff is Fri 2026-04-17 cash close. Markets are closed Sun; Monday 2026-04-20 gap or news can invalidate, especially on 15m/5m.
- No live `day_type` (rule 2) applied here — the live aiedge-scanner is canonical for intraday execution.
- Universe n=52 hand-picked names. A cleaner SPT outside this list (e.g. active in the live scanner) can outrank anything above.
- Combined score is a ranking device, not an expectancy estimate. Only the empirical [[small-pullback-trend-PLAYBOOK|PLAYBOOK]] delivers the +1.84R/trade number.
- WMT's 10.44 on 5m is an outlier driven by +18.69 net_R on a single near-vertical Friday-afternoon push; it may be late-stage. Watch for rule-11 `opp_tail` on any Monday pullback bar and for the Q2 relvol trap.

## Related

- [[small-pullback-trend-multi-tf-candidates-2026-04-19]] — pt 33 (magnitude ranking source)
- [[small-pullback-trend-tier-graded-candidates-2026-04-19]] — pt 34 (tier grading source)
- [[small-pullback-trend-monday-watchlist-2026-04-20]] — entry-frame watchlist for tomorrow's session
- [[small-pullback-trend-PLAYBOOK]] — 11-rule operational stack
- [[small-pullback-trend-INDEX]] — full reading order
- `/tmp/spt_scan_combined/rank_combined.py` — scoring script (reproducible)
- `/tmp/spt_scan_combined/ranked.json` — full per-TF + cross-TF output
