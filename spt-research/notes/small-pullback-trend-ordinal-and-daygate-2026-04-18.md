# SPT — Ordinal Rescue + Market-Wide Daily Gate (2026-04-18, pt 10)

Eleventh follow-up to [[small-pullback-trend]]. Directly addresses the two open/actionable items from [[small-pullback-trend-robustness-2026-04-18]]:

- **Q16** — can the 1st ticker-day SPT detection be rescued selectively (urgency ≥ 8, or `always_in` recently flipped)?
- **Q17** — does a market-wide "wait for N triggers to fire before taking new entries" daily gate add edge?
- **Q18** (bonus) — what caught 2026-04-14 out and would any filter have saved it?

Script: `~/code/aiedge/scanner/scratch/spt_ordinal_and_daygate_2026_04_18.py`.
Population (refreshed): n=153 after the full §7 policy stack. Slightly larger than the n=149 from the robustness note (more April 17-18 detections have resolved since).

## 0. Refreshed baseline

| | n | WR% | sum R | per trade |
|---|--:|----:|------:|----------:|
| Policy stack (robustness §7) | 153 | 59.5 | +176.85 | +1.156R |
| 1st ticker-day detection only | 69 | 52.2 | +58.44 | **+0.847R** |
| 2nd+ ticker-day (skip-1st) | 84 | 65.5 | +118.41 | **+1.410R** |

The 1st-vs-2nd+ gap is sharp: **+0.56R/trade**. Skip-1st alone already matches the robustness note's read.

## 1. Urgency-rescue of the 1st detection — REJECTED (Q16)

The natural hypothesis: the 1st detection of the day is weak on average, but the subset with very high urgency should be the true-pattern-starts. Wrong, and surprisingly so. Raising the urgency floor on the 1st bucket:

| 1st filter | n | WR% | per trade |
|---|--:|----:|---------:|
| 1st urgency ≥ 5 | 63 | 52.4 | +0.832R |
| 1st urgency ≥ 6 | 50 | 50.0 | +0.791R |
| 1st urgency ≥ 7 | 33 | 48.5 | +0.629R |
| **1st urgency ≥ 8** | **15** | **33.3** | **+0.190R** |
| 1st urgency ≥ 9 | 7 | 28.6 | +0.143R |

**Per-trade R strictly *decreases* as the urgency threshold rises on the 1st-detection bucket.** High-urgency 1st detections are the single worst cell in the dataset (+0.19R at u≥8).

Stacked (keep 1st iff urgency ≥ N, plus all 2nd+):

| Rule | n | per trade |
|---|--:|---------:|
| take-all baseline | 153 | +1.156R |
| 1st-u≥8 ∪ 2nd+ | 99 | +1.225R |
| 1st-u≥9 ∪ 2nd+ | 91 | +1.312R |
| **skip-1st (2nd+ only)** | **84** | **+1.410R** |

No urgency threshold beats simply skipping the 1st-detection bucket. Every rescue variant falls short of `skip-1st`. **Closed: urgency does not rescue the 1st detection.**

### Why — the Brooks reading

This is structurally consistent with Brooks ch. 57 ("Trend from the Open and Small Pullback Trends"):

> In the first hour, the strongest-looking bars are often the ones bulls chase and bears sell into. The tape character of an SPT only reveals itself after 2–3 failed reversal attempts.

A high-urgency *1st* SPT detection is produced at the exact moment when the scanner sees the most emotionally loud bars (big bodies, strong closes). Those are the bars Brooks says get trapped. The regime hasn't yet confirmed — it's proposing SPT, not delivering SPT. By the 2nd+ detection, the market has already chewed through one pullback-that-stayed-shallow cycle, which is the actual SPT diagnostic.

**Practical read**: when the scanner lights up loudly at 10:35 ET on day one of a would-be SPT, the temptation to enter immediately is the beginner's trap the Brooks text describes in plain language. The data confirms it.

## 2. Flip-streak rescue of the 1st detection — REJECTED (Q16)

Proxy `flip_streak` = count of consecutive setup-direction closes prior to the signal bar. `always_in` is not stored per-bar so this is the best available stand-in.

Distribution on the 1st-detection bucket is degenerate: 99% of firsts have streak 1-3, with streak ≤ 3 covering 97% of the population. No discriminating power.

| 1st filter | n | per trade |
|---|--:|---------:|
| 1st streak ≤ 1 (fresh flip) | 32 | +0.849R |
| 1st streak ≤ 2 | 55 | +0.940R |
| 1st streak ≤ 3 | 67 | +0.842R |

The slight edge at streak ≤ 2 (+0.940R vs baseline 1st +0.847R) is within sampling noise. **Closed: flip-streak proxy is not a useful rescue axis.** If per-bar `always_in` history were stored, a proper test could be redone — flagged for a schema upgrade, not urgent.

## 3. Market-wide daily gate — PROMISING (Q17)

For each trading date, sort ALL policy-stack triggers across all tickers by signal time, assign market-wide ordinal 1, 2, 3, … N. Then gate: "only take trades once K triggers have already fired today anywhere in the market."

| Rule | n | WR% | sum R | per trade |
|---|--:|----:|------:|---------:|
| take-all (mkt_ord ≥ 1) | 153 | 59.5 | +176.85 | +1.156R |
| mkt_ord > 1 | 114 | 62.3 | +137.87 | +1.209R |
| **mkt_ord > 2** | **90** | **65.6** | **+113.10** | **+1.257R** |
| mkt_ord > 3 | 73 | 64.4 | +84.20 | +1.153R |
| mkt_ord > 5 | 47 | 63.8 | +47.50 | +1.011R |

Sweet spot at `mkt_ord > 2`: per-trade R rises from +1.156R → +1.257R (+9%), but total R falls 36% (+176.85 → +113.10) due to pruning.

Bucket view clarifies where the edge lives:

| Bucket | n | WR% | per trade |
|---|--:|----:|---------:|
| market_ord 1–2 | 63 | 50.8 | +1.012R |
| **market_ord 3–5** | **43** | **67.4** | **+1.525R** |
| market_ord 6–10 | 25 | 72.0 | +1.080R |
| market_ord 11–20 | 19 | 57.9 | +1.099R |
| market_ord 21+ | 3 | 33.3 | −0.132R |

The **3–5 bucket is the cleanest single cell**: by the time 3–5 triggers have fired across the market, the day's "SPT-ness" has proved itself; by 20+ the day is showing too many triggers and starts to look like 2026-04-14 (see §5).

**The gate is a daily-confirmation proxy** — it's functionally similar to "wait for the market-wide SPT score to confirm before acting" that the parent doc proposes. Promising, but the per-trade uplift standalone is modest.

## 4. Combined: skip-1st × market-gate — THE ACTUAL EDGE

Stacking skip-1st (Q16) with market_ord > 2 (Q17) compounds:

| Rule | n | WR% | sum R | per trade |
|---|--:|----:|------:|---------:|
| baseline | 153 | 59.5 | +176.85 | +1.156R |
| skip-1st | 84 | 65.5 | +118.41 | +1.410R |
| skip-1st + mkt_ord > 2 | **61** | **70.5** | +92.63 | **+1.519R** |
| skip-1st + mkt_ord > 3 | 49 | 71.4 | +71.85 | +1.466R |
| skip-1st + mkt_ord > 5 | 32 | 71.9 | +43.29 | +1.353R |

**+0.36R/trade uplift over baseline (+31%)** on the best cell (`skip-1st + mkt_ord > 2`). n drops to 40% of baseline. 70.5% WR.

Trade-off is throughput vs expectancy. For Will's risk-per-trade sizing convention:
- Take-all: 153 trades × 1R risk → +176.85R over 3 months
- Stacked filter: 61 trades × 1R risk → +92.63R over 3 months

Per-trade economics are much better but absolute R drops. Fits the "patient + selective" trader profile better than "high-throughput" — which matches SPT's Brooks-native cadence of 1–2×/month.

## 5. 2026-04-14 diagnostic — NOT cleanly solvable (Q18)

The worst heavy-volume day (n=23, 8 wins, −2.66R) deserves a closer read:

- **17 tickers, 17 different 1st-detection triggers** — no ticker fired more than twice. This is a market-wide breadth fire, not a concentrated pattern.
- Urgency distribution spans 4–9 normally, so urgency gating doesn't catch it.
- 17/23 trades were the 1st ticker-day detection on their ticker. Only 5 were 2nd and 1 was 3rd. **The day is entirely 1st-detection trades.**

| Filter on 04-14 | n | WR% | sum R |
|---|--:|----:|------:|
| take all | 23 | 34.8 | −2.66 |
| skip-1st | 6 | 16.7 | −1.98 |
| 1st-u≥8 ∪ 2nd+ | 9 | 33.3 | +0.09 |
| mkt_ord > 5 | 18 | 38.9 | **+1.23** |

`skip-1st` made the day *worse* — because the few 2nd+ triggers happened to be losers too. `1st-u≥8 ∪ 2nd+` made it flat (+0.09R) but at the cost of only taking 9 of 23 trades. `mkt_ord > 5` is the best day-filter (+1.23R) but only because by trigger 6+ the day was already signaling breadth-failure.

**What was structurally different about 2026-04-14**: not a single-ticker SPT day but a "breadth gap-up" day. Every gappy single-name screened as TFO/SaC and fired a 1st trigger. With-trend H1/H2 entries across 17 different tickers shared the same fate — the *market* was in range, not any individual name. No filter here discriminates between "one real SPT ticker" and "17 tickers all looking like SPT."

**Deferred hypothesis**: `market_ord 21+` bucket prints −0.132R across all dates. 2026-04-14 is responsible for most of that bucket. A hard cap at "max N policy-stack trades per day, market-wide" might be the right instrument — not tested here because only one date has enough trades to populate it. Revisit when the backtest extends.

## 6. Updated one-page policy stack

Two actionable additions over the robustness §7 policy:

1. Setup ∈ {H1, H2, L1, L2}
2. day_type ∈ {trend_from_open, spike_and_channel}
3. urgency ≥ 4 (≥ 6 on trend_from_open)
4. gap_direction aligned with setup
5. always_in != opposed(setup)
6. signal time ∈ [10:30 ET, 14:15 ET)
7. Tentative short-side tightening: [11:30 ET, 14:15 ET)
8. **NEW — skip the 1st ticker-day detection** (+0.25R/trade uplift)
9. **NEW — require market_ord > 2** (compounds to +0.36R/trade over baseline)
10. Exit: 3R target / 1R stop / hold to resolution

**Projected expectancy on the refreshed window**:
- Old policy (1–7): +1.156R/trade, 153 trades, +176.85R / 3 months
- New policy (1–9): +1.519R/trade, 61 trades, +92.63R / 3 months, 70.5% WR

## 7. Updated open items

- **Q16 (ordinal filter)** — CLOSED. `skip-1st` is the right action. Urgency and flip-streak both fail to rescue the 1st bucket. Data contradicts the intuitive rescue hypothesis.
- **Q17 (date concentration / daily gate)** — CLOSED as "use market_ord > 2 in combination with skip-1st". Standalone weaker than stacked.
- **Q18 (2026-04-14 diagnostic)** — PARTIAL. The day is a market-wide breadth fire of 17 single-ticker 1st triggers. None of the tested filters cleanly rescue it. Candidate: **max-N-trades-per-day market-wide cap**. Not testable yet (only one day populates the relevant bucket).
- **NEW (Q21) — max-trades-per-day cap** (OPEN). The `market_ord 21+` bucket is almost entirely 2026-04-14. Until more heavy days populate the tail, the right rule-shape can't be fit. Revisit with a longer backtest.

## 8. What surprised me

**The single most surprising finding**: high-urgency 1st detections are the *worst* cell in the entire policy-stack universe (+0.19R at u≥8). I expected the 1st-detection bucket to be split between "real early entries" (high urgency) and "chop" (low urgency), with the high-urgency cell carrying the edge. It's the opposite — high-urgency 1st detections are systematically worse than low-urgency 1st detections.

This matches Brooks' explicit text about the opening hour of an SPT but is easy to forget in practice because the scanner's urgency score *feels* like a confidence proxy. Reminder for future work: urgency measures *bar texture*, not *regime maturity*. A loud 1st bar can be the climax that fails, not the start of the trend.

## Related

- [[small-pullback-trend]] — parent concept. This note adds rule 8 and 9 to the policy stack and closes Q16/Q17. Add Q21.
- [[small-pullback-trend-robustness-2026-04-18]] — immediate predecessor. This note takes its n=149 policy stack as baseline (refreshed to n=153) and closes its two open items.
- [[small-pullback-trend-entry-side-filters-2026-04-18]] — time-of-day rule from §7 carries through unchanged.
- Script: `~/code/aiedge/scanner/scratch/spt_ordinal_and_daygate_2026_04_18.py`.
- Source: Brooks, *Trading Price Action: Trends*, ch. "Trend from the Open and Small Pullback Trends" — the "weak-looking signal bars are diagnostic" principle.
