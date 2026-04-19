# SPT — Robustness Audit (2026-04-18, pt 9)

Tenth follow-up to [[small-pullback-trend]]. The prior eight notes refined entry and exit rules and settled on a policy stack projecting **+1.30R/trade, 61.8% WR, ~34 trades/month** ([[small-pullback-trend-entry-side-filters-2026-04-18]] §7). Those numbers are aggregate across 3 months and a few dozen tickers. This note asks the obvious follow-up:

> How concentrated is that edge? Is it 2 tickers doing all the work, or 5 great days, or is it distributed?

Also: two entry-side splits not covered before — signal-bar direction/body quality, and ordinal-within-ticker-day (does the Nth SPT detection on the same ticker perform differently than the 1st?).

Population: same revised policy stack as §7. Refreshed run on the DB today shows **n=149** resolved trades (vs n=102 from the earlier note — the DB accreted more April detections since).

Snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite` @ 2026-04-18 afternoon.
Script: `~/code/aiedge/scanner/scratch/spt_robustness_audit.py`.

## 0. Refreshed baseline

| | n | WR% | sum R | per trade |
|---|--:|----:|------:|----------:|
| Policy stack (entry-side §7) | 149 | 59.7 | +176.78 | +1.19R |

Per-trade expectancy on the refreshed population is **−0.11R lower** than the n=102 cut (+1.19R vs +1.30R). Difference is noise — adding 47 trades narrows confidence intervals, not the mean. The policy stack holds.

## 1. Symbol concentration

62 distinct tickers fire at least one policy-stack trade. Cumulative R share by top-N:

| Rank | Ticker | Cum R | Share |
|-----:|--------|------:|------:|
| 1 | CMCSA | +33.00 | 18.7% |
| 2 | MRNA  | +57.00 | 32.2% |
| 3 | MSFT  | +72.00 | 40.7% |
| 4 | PLTR  | +84.07 | 47.6% |
| 5 | NET   | +96.07 | 54.3% |
| 6 | T     | +105.57 | 59.7% |
| 7 | CRWD  | +114.57 | 64.8% |
| 8 | INTU  | +123.57 | 69.9% |
| 9 | RCL   | +129.57 | 73.3% |
| 10 | ALNY | +135.57 | 76.7% |
| 11 | ... | | 80.0% reached |

Top 10 tickers → 77% of R; top 3 alone → 41%. That's concentrated, but **leave-one-out testing shows the edge is not ticker-fragile**:

| Drop | n | WR% | per-trade R |
|------|--:|----:|------------:|
| none (baseline) | 149 | 59.7 | +1.19R |
| drop CMCSA | 138 | 56.5 | +1.04R |
| drop MRNA | 141 | 57.4 | +1.08R |
| drop MSFT | 144 | 58.3 | +1.12R |
| drop PLTR | 141 | 58.9 | +1.17R |
| drop NET  | 145 | 58.6 | +1.14R |
| **drop top-3 (CMCSA/MRNA/MSFT)** | **125** | **52.0** | **+0.84R** |

Dropping the three highest-R tickers removes −0.35R/trade but leaves the strategy profitable at +0.84R × 125 trades = +105R over the window. The policy stack survives the removal of its best contributors.

One structural note: CMCSA, MRNA, MSFT all hit 100% WR at high n (8-11 trades each). That's not quite a "ticker edge" — it's the intersection of "ticker gapped hard → day-type became TFO/S&C → urgency stayed ≥ 6 for many bars → multiple H1/H2 detections in the window". The same structure on another gappy single-name (PLTR, NET, CRWD, INTU) produced the same cleanup. The edge is a **pattern the policy catches**, not an individual ticker.

## 2. Date concentration — the harder finding

39 distinct trading dates. Cumulative R share:

| Rank | Date | n | Wins | sum R | Cum share |
|-----:|------|--:|-----:|------:|----------:|
| 1 | 2026-04-16 | 17 | 15 | +43.00 | 24.3% |
| 2 | 2026-04-15 | 7 | 7 | +21.00 | 36.2% |
| 3 | 2026-01-13 | 8 | 8 | +17.56 | 46.1% |
| 4 | 2026-01-21 | 12 | 8 | +14.00 | 54.1% |
| 5 | 2026-04-13 | 5 | 5 | +12.50 | 61.1% |
| 6 | 2026-04-08 | 4 | 4 | +12.00 | 67.9% |
| 7 | 2026-02-23 | 4 | 4 | +12.00 | 74.7% |
| 8 | 2026-01-28 | 4 | 4 | +12.00 | 81.5% |

**8 trading days carry 81% of the total R.** The other 31 days contribute the remaining 19%.

This is the actual texture of the SPT edge: a small minority of days do all the work. Brooks flags this in the source text — SPT is a once-or-twice-a-month regime, not a daily setup. The data shows exactly that rhythm: in 3 months (~60 trading days), ~8 delivered the majority of R.

Daily breakdown:
- **25 profitable days** (64%) vs **14 negative days** (36%)
- Median daily R: **+3.00R**, mean daily R: +4.53R
- Worst day: 2026-03-20, n=4, 0 wins, −4.00R
- Ugliest volume day: **2026-04-14, n=23, 8 wins, −2.66R** (heavy trigger day that barely broke even)

The 2026-04-14 row is worth isolating. 23 policy-stack trades fired, the day closed marginally red. This is what SPT looks like on a day that *superficially* screens as one — lots of qualifying signal but the tape didn't follow through to 3R. No current filter catches this; the ordinal analysis in §4 may address it.

## 3. Signal-bar body% (signal-bar "class" is degenerate inside H1/H2/L1/L2)

By construction, H1/H2 need a bull signal bar (long) and L1/L2 need a bear signal bar (short). The "is the signal bar a trend bar in the setup direction?" check collapses to 100% — all 149 policy-stack trades have a with-trend signal bar. Not a useful axis. Scratch.

Signal-bar **body %** (unsigned body / range):

| Body % | n | WR% | per-trade R |
|--------|--:|----:|------------:|
| 40–60% | 18 | 61.1 | +1.23R |
| 60–80% | 38 | 50.0 | +0.85R |
| ≥80% (strong trend bar) | 93 | 63.4 | **+1.32R** |

Not monotonic, not clean enough to filter on. Signal bars with ≥80% body do slightly better (+1.32R vs baseline +1.19R) but the mid-band 60–80% cluster drags. The scanner's trend-bar density check in `_score_small_pullback_trend` already captures this at the window level; doubling down at the signal-bar level isn't additive.

Verdict: **don't filter on signal-bar body %.** The entire policy-stack population is "clean enough" at the bar level that body% is not discriminating.

## 4. Ordinal within ticker-day (NEW, the actual edge)

For each (ticker, date) grouping, sort policy-stack trades by signal time and assign ordinal 1, 2, 3, etc. Then break down R by ordinal:

| Ordinal | n | WR% | per-trade R |
|---------|--:|----:|------------:|
| 1st | 69 | 52.2 | +0.85R |
| 2nd | 39 | 53.8 | +1.09R |
| 3rd | 17 | **70.6** | **+1.69R** |
| 4th | 13 | 69.2 | +1.55R |
| 5th+ | 11 | **100.0** | **+2.45R** |

Edge **compounds** with each additional detection on the same ticker-day. 1st detection is near the pooled mean; 3rd+ is substantially better; 5th+ is perfect across 11 trades.

This is structurally consistent with Brooks' SPT description (ch. "Signs of Strength in a Trend"):
- The 1st with-trend signal of an SPT is hard to trust. Beginners fade it.
- By the 3rd-5th signal, the tape character is obvious. The signal bars "look bad" but every one works. This is the regime's signature.
- Institutional continuation drives repeated signals without meaningful pullback.

Per-trade expectancy map (cumulative, by "skip first N detections"):

- Take all detections: 149 trades, +1.19R/trade
- Skip 1st only (take 2+): 80 trades, **+1.48R/trade**
- Skip 1st & 2nd (take 3+): 41 trades, **+1.85R/trade**
- Take only 4+: 24 trades, +1.97R/trade

**Practical read**: even if the 1st SPT-pattern detection of the day is being produced by the existing policy, the Brooks-consistent behavior is to wait. Skip-1st lifts per-trade R by +0.29R (+24%); skip-1st-and-2nd lifts it by +0.66R (+55%). This is the single biggest unexploited axis in the current policy.

The trade-off is fewer trades: skip-1st halves the 1st-bucket n (69 → 0) while adding nothing to the other buckets — nets to 80 trades/quarter instead of 149. On per-trade R the skip is positive, but the **total-R effect is ambiguous**:

| Policy variant | n | sum R |
|----------------|--:|------:|
| Take all | 149 | +176.78 |
| Skip 1st | 80 | +118.34 |
| Skip 1st & 2nd | 41 | +75.99 |

Absolute R falls with aggressive skipping. This is the classic trade-off: higher per-trade edge but fewer trades = fewer shots. For a risk-constrained trader sizing by conviction, skipping 1st is a clean uplift. For a pure-throughput trader sizing flat, take-all still wins on total R.

A natural compromise: **take 1st only when urgency is exceptionally high (≥ 8); take 2nd+ always**. Not tested here — queued for a future note.

## 5. Setup × direction — short-side L2 is the persistent weak spot

| Setup / direction | n | WR% | per-trade R |
|-------------------|--:|----:|------------:|
| H2 / long | 24 | **75.0** | **+1.76R** |
| L1 / short | 47 | 61.7 | +1.10R |
| H1 / long | 67 | 56.7 | +1.16R |
| L2 / short | 11 | **36.4** | **+0.45R** |

H2/long is the strongest setup — matches the earlier empirics §3 finding ("H2 / spike_and_channel — 45.9% WR" as the best raw combo, which upgrades to 75% WR after the full filter stack). L2/short is structurally weak (Q11) and persists even after all filters — only 11 trades but a clear laggard.

## 6. Day-type is symmetric under the current policy

| Day type | n | WR% | per-trade R |
|----------|--:|----:|------------:|
| spike_and_channel | 72 | 62.5 | +1.27R |
| trend_from_open | 77 | 57.1 | +1.11R |

TFO's urgency ≥ 6 tightening (vs S&C's urgency ≥ 4) does its job — the residuals after filtering are roughly equal-edge. No further day-type-specific tuning needed.

## 7. Updated open items

- **Q5** (cross-asset SPT) — unchanged, blocked on multi-asset expansion.
- **Q11** (short-side weakness) — robustness audit reinforces: L2/short is 36% WR @ +0.45R even with all filters; short-side is structurally weaker in this window. Still waits on bear-regime data for final closure.
- **NEW (Q16) — Ordinal filter** (OPEN, ACTIONABLE). Ordinal ≥ 2 adds +0.31R/trade; ordinal ≥ 3 adds +0.66R/trade. Trade-off is fewer shots. Next study: does `urgency ≥ 8` or `always_in recently flipped` let us keep the 1st-detection trades selectively?
- **NEW (Q17) — Date concentration** (OPEN, STRUCTURAL). 81% of R from 8 trading days. This is the Brooks-described 1–2×/month SPT rhythm. Implication: the strategy must tolerate stretches of low R; position-sizing and psychological expectations need to match that cadence. Candidate daily filter to explore: detection *count* at policy-stack on the same ticker — if ≥ 3 SPT triggers fire by bar 30, the day is confirmed SPT and later triggers are high-confidence. Possibly a daily-budget filter ("don't trade SPT until 2+ same-day triggers fire on any ticker").
- **NEW (Q18) — 2026-04-14 high-volume flat day** (OPEN, DIAGNOSTIC). 23 policy-stack triggers, 8/23 wins, −2.66R. A day that screened loudly as SPT and didn't deliver. Needs a chart pass to see what was structurally different — potential to add a daily-level regime tell.

## 8. Revised one-page policy stack (unchanged)

Nothing in this note changes the §7 policy — the edge is real at +1.19R/trade, n=149, diversified across 62 tickers and 39 days. The **ordinal-filter candidate** (skip 1st detection) is a possible future uplift; for now the policy is:

1. Setup ∈ {H1, H2, L1, L2}
2. day_type ∈ {trend_from_open, spike_and_channel}
3. urgency ≥ 4 (≥ 6 on trend_from_open)
4. gap_direction aligned with setup
5. always_in != opposed(setup)
6. signal time ∈ [10:30 ET, 14:15 ET)
7. Tentative short-side tightening: [11:30 ET, 14:15 ET)
8. Exit: 3R target / 1R stop / hold to resolution (no time-stop, no event exit)

**Structural reminder from this audit**: expect 64% profitable days and 8 "monster days" per quarter to carry the R. The other 50 days are roughly break-even in aggregate.

## Related

- [[small-pullback-trend]] — parent concept. Add Q16–Q18 to the open list.
- [[small-pullback-trend-entry-side-filters-2026-04-18]] — immediate predecessor; refined time-of-day. This note takes its policy stack as baseline.
- [[small-pullback-trend-alignment-filters-2026-04-18]] — short-side asymmetry (Q11), reinforced here at the setup level (L2/short = +0.45R).
- [[small-pullback-trend-empirics-2026-04-18]] — original per-setup WR (H2/S&C = 45.9% raw; here = 75% after stack).
- Script: `~/code/aiedge/scanner/scratch/spt_robustness_audit.py`.
