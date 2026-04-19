# Small Pullback Trend (SPT)

The strongest type of trend day. A variant of trend-from-open where **every pullback** after the trend begins stays under ~20–30% of the recent ADR. Forms only once or twice a month.

## Brooks' diagnostic profile

**Structural markers** (Brooks, *Trading Price Action Trends*, ch. "Trend from the Open and Small Pullback Trends"):
- Opens within a few ticks of one extreme; closes within a few ticks of the opposite extreme.
- Trend from the open with all subsequent pullbacks ≤ 20–30% of recent ADR.
- Usually a relatively tight channel — not an impressive spike; price grinds.
- Market never really gets back to the moving average.
- Many **countertrend trend bars** and **weak with-trend signal bars**. Buy signals look bad; bear bars look compelling. This is diagnostic, not disqualifying.
- Beginners: see weak setups → short → trapped. Experienced: recognize tape → take every ugly with-trend setup.
- ~2/3 of SPT days produce a **larger pullback after 11:00 PST** (~14:00 ET) that is ~2× the biggest prior pullback. Often heralded by a climax-looking 1–2-bar burst in-trend — counter-intuitively a sell signal (bulls), not a breakout continuation.
- Channel is tight; do **not** move stops to breakeven too early — price will often test back near entry before extending.
- Follow-through often bleeds into the first 1–2 hours of the **next** session — look to enter with trend on pullbacks after the next open.

**The Brooks insight**: institutions had a big order to fill in that direction. They didn't want to create a climax that reverses, so they filled in pieces all day. The other side never gets a great entry, they chase, short-covers add to the drift. Result: relentless, low-emotion grind.

## Why SPT rewards what looks wrong

The core SPT paradox (Brooks):
1. Bulls see the trend, wait for a "real" pullback that never comes.
2. Bulls capitulate and start buying small at the market and on tiny pullbacks.
3. Bears don't get a good short, scale in too early on weak setups, get squeezed.
4. Short-covering + aggressive-bull market-buying → drift without pullback.
5. The very thing that looks weak (small bars, many bear trend bars, weak buy signals) is the strength signal.

Practical corollary from ch. 47 (*Signs of Strength*): in a confirmed SPT, **you don't need a setup to enter**. Market-at-any-time with a 2-pt money stop beats waiting for a perfect signal bar, because the signal bars will all look bad. Shorting below bars is a loser's strategy; if you short, only above bars (i.e. above highs, fading new highs).

## Five-bar "is this an SPT setup" checklist (Brooks-grounded)

A trader's mental checklist, tuned to Brooks text + the aiedge scanner implementation:
1. **Open at an extreme** — open within ~20% of session high/low; strong first-bar trend bar in the eventual trend direction, or large gap in that direction.
2. **Density of with-trend trend bars** over the first 3–6 bars — majority trend bars with bodies ≥ 40% of range.
3. **Shallow pullbacks** — every pullback since the trend began ≤ 20–30% of ADR (Brooks' threshold). In the scanner's SPT component: `SPT_DEPTH_SHALLOW = 0.25`, `SPT_DEPTH_MODERATE = 0.40` of prior-leg height.
4. **MA untouched** — the 20EMA has not been touched in many bars (~2+ hours on 5-min = ~20 bars).
5. **No broken swings** — no unreclaimed break of the prior higher-low (bull) / lower-high (bear).

If all five line up inside the first hour, treat the tape as SPT and suppress the urge to fade.

## Scanner implementation (as-of 2026-04-18)

**Location**: `aiedge/signals/components.py` → `_score_small_pullback_trend(df, direction)` — returns 0.0 to 3.0 raw.

**Window**: last `SPT_LOOKBACK_BARS = 15` bars (~75 min on 5-min). Note: rolling — reflects CURRENT 15-bar window, not peak (confirmed by `scratch/test_spt_fix.py`).

**Five weighted sub-checks** (total raw max 3.0):
| # | Sub-check | Weight | Logic |
|---|-----------|-------:|-------|
| 1 | Trend-bar density | 0.8 | Fraction of 15-bar window that are trend bars in direction with body ≥ 40% of range |
| 2 | Pullback depth | 0.8 | Max/avg pullback depth vs prior leg height. Uses the 0.25 / 0.40 / 0.60 thresholds from Brooks |
| 3 | No broken swings | 0.6 | Breaks of prior HL (bull) / LH (bear); unreclaimed breaks → 0 |
| 4 | Higher-closes streak | 0.4 | Longest streak of closes > prior close (bull, inverted for bear); 6+ → 1.0 |
| 5 | Pullback-bar tail density | 0.4 | Of non-trend bars, fraction with deep bottom tail (bull) / top tail (bear); ≥ 60% → 1.0 |

**Hard gate**: if sub-check 1 (density) is zero, the window is chop — SPT returns 0 immediately. This is what prevents shallow-pullback noise in a range from leaking SPT points.

**Day-type weighting** (`daytype.py` DAY_TYPE_WEIGHTS):
| Day type | SPT weight | Rationale |
|----------|:----------:|-----------|
| trend_from_open | 1.5 | SPT's native habitat — amplify |
| spike_and_channel | 1.2 | Channel phase often exhibits SPT behavior |
| trending_tr | 1.0 | Neutral |
| trading_range | 0.3 | In a range, "shallow pullback" = trap — suppress |
| tight_tr | 0.1 | Same but harsher |
| undetermined | 1.0 | No signal either way |

## Empirical WR from Pattern Lab (n=3,368 detections, 2R cap)

**By day-type overall** (WIN/LOSS only, SCRATCH & INCOMPLETE excluded):
- `trend_from_open`: n=345, WR=25.5%, avg urgency=4.75 — highest urgency, lowest chop noise.
- `spike_and_channel`: n=276, WR=17.8%, avg urgency=2.52.
- `trading_range`: n=800, WR=22.3%, avg urgency=0.54 — high volume but low urgency signal strength.
- `trending_tr`: n=92, WR=6.5%.
- `undetermined`: n=91, WR=7.7%.

Day-type urgency differential is real: `trend_from_open` produces urgency ≈ 9× `trading_range`.

**Per-setup WR in SPT-adjacent day types** (n≥8, trend_from_open + spike_and_channel):

With-trend setups (H1/H2/L1/L2) in SPT regimes:
- H2 / spike_and_channel — **45.9% WR** (n=37) ← top combo.
- H1 / trend_from_open — 39.0% (n=59).
- H2 / trend_from_open — 35.7% (n=42).
- L2 / trend_from_open — 30.8% (n=26).
- L1 / trend_from_open — 23.8% (n=42).

Failing-side / reversal setups (FH/FL) in the same day types:
- FL2 / trend_from_open — 22.4% (n=49).
- FH2 / trend_from_open — 17.9% (n=28).
- FH2 / spike_and_channel — **3.1%** (n=32) ← Brooks' "shorting below bars is a loser's strategy" in data.
- FH1 / trend_from_open — **0.0%** (n=13).

**Interpretation**: empirical data confirms Brooks:
- In SPT-like days, with-trend H2 / L2 at or near the MA is the edge (~36–46% WR at 2R → positive expectancy).
- Countertrend FH/FL setups collapse to single-digit WR in the strongest trend types — exactly what Brooks describes as the trap.

## Open research questions

1. **SPT-score → WR calibration** (OPEN, blocked). The `_score_small_pullback_trend` component is wired into urgency and day-type weighted, but we haven't run a bucketed study of "detections with SPT-component ≥ 2.0 vs < 2.0." **Blocked by schema**: `detections` stores `day_type` but not per-component raw scores. Resolution path documented in [[small-pullback-trend-empirics-2026-04-18]] §4 — add `spt_component_raw` / `spt_component_weighted` columns at the next scanner migration.
2. **The 11-AM-PST / 2-PM-ET larger pullback** (CLOSED, confirmed). Brooks' claim held up: WR on with-trend setups in SPT day types collapses from ~36% (other hours) to **14.3%** in the 14:00 ET bucket. See [[small-pullback-trend-empirics-2026-04-18]] §1. Actionable: suppress new-entry urgency in 13:45–14:45 ET on `trend_from_open` / `spike_and_channel` days.
3. **Next-day follow-through** (DEFERRED). Not testable with current Pattern Lab data — only 12 ticker-days meet strict SPT threshold, and scanner warm-up means only 30 of 1,606 resolved detections land in the first hour. Revisit when backtest log extends further back or data source switches to `chart_json` replay. Full analysis in [[small-pullback-trend-stop-and-timing-2026-04-18]] §3.
4. **Breakeven-stop trap / stop-side** (CLOSED, inverted). Tested three stop widths (1R / 1.5R / 2R) against existing WIN `mae` and LOSS `mfe`. Widening stops *strictly decreases* expectancy (-0.12R → -0.55R → -0.86R per trade). The 5–8 winners rescued are swamped by the extra -0.5R–1R loss on 200+ losers whose `mae` already exceeded the wider stop. Brooks' directional advice (don't narrow to BE early) holds — 1R is already the sweet spot on the narrow side. The implied hypothesis that *wider* stops would help is wrong. The right optimization axis is target-side or filtering, not stop-side. See [[small-pullback-trend-stop-and-timing-2026-04-18]] §1.
5. **Cross-asset SPT** (OPEN). Brooks text is Emini-focused (12-pt ADR, 9-tick pullbacks). Scanner thresholds (`SPT_DEPTH_SHALLOW = 0.25`) are unitless fractions of leg height, so they should generalize. But ADR and pullback granularity differ for single-stock gap-ups vs indices; SPT component may need per-instrument calibration. Relevant for the multi-asset expansion.
6. **FL2 / `trend_from_open` session-timing edge** (CLOSED). The empirics note flagged FL2/TFO's 22.4% aggregate WR as "worth isolating." Resolution: WR is entirely driven by early-session (bars 7–18, pre-confirmation) entries at **50% WR, n=8, avg MFE 2.72R**; collapses to 12–25% after bar 18 and 0% at close. FL2/TFO is not a countertrend setup; it's an opening-fade before the trend's direction is confirmed. See [[small-pullback-trend-stop-and-timing-2026-04-18]] §2.
7. **Target-side optimization** (NEW, CLOSED). Raising the fixed target from 2R to 3R flips SPT with-trend expectancy from −0.12R to +0.08R/trade with no other change (robust to the BE-scratch assumption band). 90% of WINs already run past 3R. See [[small-pullback-trend-targets-and-urgency-2026-04-18]] §2.
8. **Urgency threshold** (NEW, CLOSED). Stratifying the 310-trade SPT with-trend population by `urgency` reveals 164 trades at urgency < 4 carrying all of the negative expectancy (−0.56R / trade at 2R, strictly worse at wider targets). Filter `urgency ≥ 4` + 3R target → **+0.77R / trade on 146 trades** (+113R over 3 months, vs −37R current). Urgency ≥ 4 maximizes total R; urgency ≥ 6 maximizes per-trade expectancy (+1.01R / trade on 99 trades). Does NOT generalize to FH/FL countertrend — those remain unprofitable at every urgency level. See [[small-pullback-trend-targets-and-urgency-2026-04-18]] §3.
9. **Urgency-gate generalization / instrument dispersion** (NEW, CLOSED). Urgency < 4 is universally unprofitable across every day_type. Urgency ≥ 4 is structurally impossible in `trading_range` (max 2.3) — the gate is a de-facto regime filter. Threshold is regime-dependent inside SPT: `spike_and_channel` clean at ≥ 4, `trend_from_open` needs ≥ 6 (4–6 band is a −0.33R trap). Single-names supply 49% urgency-≥4 detections; SPY/QQQ supply 7% — the edge lives in single-names; indices need a separate filter. The 14:00 ET collapse is NOT rescued by the urgency gate (WR 9.5%, −0.90R even at u≥4) — time-of-day is an independent additive guard. See [[small-pullback-trend-urgency-generalization-2026-04-18]].
10. **Alignment filters — `always_in` and `gap_direction`** (NEW, CLOSED). Gap-direction alignment is a strong additive filter: gap_opposed trades in SPT regimes lose 100% at urgency ≥ 4 (n=9). `always_in = opposed(setup)` is the same kind of "loud-right-before-failure" trap as the 14:00 ET bucket — highest avg urgency (4.74) but WR only 13.6%. Adding both filters removes small certain-loss pockets. See [[small-pullback-trend-alignment-filters-2026-04-18]] §§1–2.
11. **Short-side directional asymmetry** (NEW, OPEN). At urgency ≥ 4 in SPT regimes, longs run 52–64% WR / +1.11R to +1.56R per trade; shorts run 32–35% WR / +0.20 to +0.27R. ~20–30 pp WR gap. Likely bull-regime bias in the 2026-01-22 → 2026-04-17 window; possibly structural (short-covering squeezes bear setups). Until data spans a bear leg, treat short-side SPT as marginal — tentative rule: u≥6 across both regimes on shorts. See [[small-pullback-trend-alignment-filters-2026-04-18]] §3.
12. **Time-to-target glidepath** (NEW, CLOSED). Urgency-gated SPT winners hit 2R in median 10 bars (50 min); avg MFE 7.19R. CK-based R accumulation: +1.14R @ bar 5, +1.86R @ bar 10, +3.43R @ bar 20, +3.86R @ bar 30. Winners plateau around bar 20–25. A bar-25 time-stop was proposed as an experimental addition. See [[small-pullback-trend-alignment-filters-2026-04-18]] §4.
13. **Bar-N time-stop** (NEW, CLOSED — REJECTED). Simulation on n=153 urgency-gated SPT with-trend trades at 3R target: baseline (hold) = +0.91R/trade, bar-20 time-stop = +0.83R/trade (−12R/quarter), bar-30 = breakeven, bar-10 = catastrophic (−52R). 96% of winners run past 3R MFE; ≥91% hit 2R by bar 20. Conditional on "still open at bar 20," holding beats exit-at-close by +0.48R/trade. The upstream filters (urgency≥4, gap_aligned, always_in) already remove the chop trades the time-stop was meant to catch. Policy: **no time-stop**, hold to 3R target or 1R stop. See [[small-pullback-trend-time-stop-2026-04-18]].
14. **Event-triggered adverse exit** (NEW, CLOSED — REJECTED). Seven rule variants tested on n=153 trades (3R target): first adverse close, two consecutive bear bars, first bear trend bar, break below prior-bar low, close below 10-EMA, bear trend bar closing below 10-EMA (Brooks' exact hypothesis), bear trend bar closing below 20-EMA. **Every rule costs R** — range −24R/quarter (20-EMA variant) to −103R/quarter (`first_adverse_trend_bar`). When a rule fires, 59–65% of those trades would have been full 3R winners. The rule shoots winners; what it reads as "weakness" is the Brooks-diagnostic texture of an SPT channel. Third rejected exit-side refinement after the 2R-target and bar-N time-stop failures — all three fail because the urgency + gap + always_in filter stack already delivers a population that rewards holding. **Policy unchanged**: hold to 3R or 1R stop. See [[small-pullback-trend-adverse-event-2026-04-18]].
15. **Entry-side refinement — time-of-day window + ATR quality** (NEW, CLOSED). Tested on the same n=153 urgency-gated SPT with-trend population. (a) **Time-of-day**: replacing the narrow `suppress 13:45–14:45 ET` rule with `enter only 10:30–14:15 ET` adds **+0.35R per trade (+24R/quarter)** on the policy stack (TFO≥6, S&C≥4) — n drops from 115 to 102 but total R climbs from +109.41 to +133.04 (+1.30R/trade, 61.8% WR). The 14:45–15:30 ET bucket is actually the worst single band (7.7% WR, −0.69R, n=13); the current 13:45–14:45 rule barely helps because it misses that zone and the close-hour flats. Lunch (11:30–13:30 ET) is the single strongest window (+1.42R/trade, 63.1% WR, n=65) — do NOT fade SPT through lunch. (b) **Signal-bar ATR ratio**: U-shape (quiet <0.75 and explosive ≥2.0 both outperform mid) but too noisy to harden into a rule; ATR band filter subtracts R. (c) **Direction × window**: short-side weakness (Q11) is concentrated in the **opening hour only** — shorts in 10:30–11:30 ET run 12.5% WR while lunch shorts run 68% WR. Tentative short-side rule: tighten entry window to 11:30–14:15 ET. See [[small-pullback-trend-entry-side-filters-2026-04-18]].
16. **Ordinal-within-ticker-day filter** (NEW, CLOSED). Urgency does NOT rescue the 1st-detection bucket — counter-intuitively, high-urgency 1st detections are the worst cell in the whole policy stack (+0.19R at u≥8, strictly monotonic decrease in R as urgency threshold rises on the 1st bucket). Flip-streak proxy also fails (distribution degenerate). Action: **skip-1st** unconditionally → +1.410R/trade on n=84 (vs +1.156R baseline, n=153). Mechanism is Brooks: urgency measures bar texture, not regime maturity — a loud 1st bar is the climax that fails, not the start of the trend. See [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] §§1–2.
17. **Date concentration — market-wide daily gate** (NEW, CLOSED). Market-wide ordinal (`mkt_ord`) — rank of each trigger in the cross-ticker daily firing sequence — is a real filter. `mkt_ord 3–5` bucket prints +1.525R at 67.4% WR; `1–2` prints +1.012R at 50.8%. Stacking **skip-1st + mkt_ord > 2** compounds to **+1.519R/trade, 70.5% WR, n=61** (+0.36R/trade over baseline, −48% throughput). Absolute R falls — classic selectivity trade-off; matches SPT's "1–2×/month" cadence. See [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] §§3–4.
18. **Diagnostic — 2026-04-14 high-volume flat day** (NEW, PARTIAL). 23 policy-stack triggers, 17 different tickers, −2.66R. Day was a **market-wide breadth gap-up** that superficially screened SPT on many names simultaneously. 17/23 trades were each ticker's 1st detection — skip-1st rule removes most of the loss but takes only 6 trades. No tested filter cleanly saves it. Candidate: per-day trade cap (`market_ord ≤ N`) — blocked on more heavy-day data. See [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] §5.
19. **Symbol concentration — not fragile, but clustered** (NEW, CLOSED). 62 distinct tickers fire; top-10 deliver 77% of R, top-3 (CMCSA/MRNA/MSFT) deliver 41%. Leave-one-out: dropping any single top-5 ticker leaves +1.04R to +1.17R/trade; dropping top-3 drops to +0.84R/trade on n=125 — still positive. The policy is a pattern the data catches on gappy single-names, not a ticker-specific artifact. See [[small-pullback-trend-robustness-2026-04-18]] §1.
20. **Signal-bar body% filter** (NEW, CLOSED — REJECTED). Body% buckets show non-monotonic behavior: 40–60% = +1.23R, 60–80% = +0.85R, ≥80% = +1.32R. Scanner's window-level trend-bar density already captures this; adding a signal-bar-level cutoff isn't additive. Don't filter on signal-bar body%. See [[small-pullback-trend-robustness-2026-04-18]] §3.
21. **Max-trades-per-day cap** (NEW, CLOSED — wrong rule shape). Hard-cap at N trades/day is strictly worse than no cap (discards winners on good days). Correct rule is a **daily-confirmation gate**: after the first 3 policy-stack trades of the day, continue only if ≥ 2 won. On refreshed data: +1.34R/trade (vs +1.15R baseline), n=110; stacked with skip-1st + mkt_ord > 2 compounds to +2.00R/trade (n=33, 82% WR). Cleanly rejects 2026-04-14 (1/3 first-3 wins) while preserving 2026-04-15/16 (3/3). Closes Q21. See [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] §2.
22. **Short-side directional asymmetry — localized** (NEW, CLOSED). Q11 re-tested on refreshed data: shorts run 54% WR / +0.88R at full policy stack (up from 32% in pt 5's snapshot). Gap is almost entirely a **time-of-day artifact**: the 10:30–11:30 ET short bucket is 0/9 (−9R). Shorts from 11:30 ET onward run at long-side economics (68% WR, +1.44R). Not a structural regime bias — a counter-trend-squeeze window that maps to Brooks' text on ~11:00 ET / European-close bear entries. Rule 6 addendum: shorts hard-excluded before 11:30 ET. Closes Q11. See [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] §1.
23. **First-3-gate behavior on low-n days** (NEW, CLOSED). Low-n (< 4 triggers) days run +0.853R/trade vs +1.231R/trade on high-n — a real but non-actionable gap, because the damage is concentrated in singletons (n=1 days, 27 trades, 37% WR, +0.394R/trade), and rule 7 (skip-1st) mechanically excludes every singleton day by definition. n=2 days run +1.40R/trade and correctly pass through. Adding an explicit `day_n ≥ 2` filter is a no-op at the full PLAYBOOK stack (n=37 identical). Policy unchanged; Q22 closed. See [[small-pullback-trend-low-n-days-2026-04-18]].

## Sources

- Brooks, *Trading Price Action: Trends*, ch. "Trend from the Open and Small Pullback Trends" (extracted at `~/code/aiedge/brooks-source/extracted/trading-price-action-trends/57_trend-from-the-open-and-small-pullback-trends.md`).
- Brooks, same book, ch. "Signs of Strength in a Trend" (`47_signs-of-strength-in-a-trend.md`) — buy-at-market-in-strong-trend, weak-setup-is-diagnostic.
- Brooks, *Trading Price Action: Reversals*, ch. "The Best Trades Putting It All Together" (`51_the-best-trades-...md`) — trader's equation framing.
- Scanner component: `aiedge/signals/components.py:715` — `_score_small_pullback_trend`.
- Day-type weights: `aiedge/context/daytype.py` (DAY_TYPE_WEIGHTS matrix).
- SPT validation tool: `tools/spt_validate.py` (ground-truth day-type vs SPT score on 60 SPY trading days).
- Empirical WR: `~/code/aiedge/scanner/db/pattern_lab.sqlite` (`detections` table, WIN/LOSS outcomes at 2R cap).

## Related

- [[small-pullback-trend-empirics-2026-04-18]] — follow-up: time-of-day WR collapse at 14:00 ET (confirms Brooks' "11 PST larger pullback" claim), WIN MAE distribution, countertrend trap by setup.
- [[small-pullback-trend-stop-and-timing-2026-04-18]] — follow-up: stop-width widening HURTS (−0.12R → −0.86R at 2R stop), FL2/TFO is a session-timing opening-fade (not countertrend), next-day follow-through not testable.
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — follow-up: raising target 2R → 3R flips expectancy positive; urgency ≥ 4 filter removes 53% of trades and all negative expectancy; countertrend FH/FL does not respond to urgency gating.
- [[small-pullback-trend-urgency-generalization-2026-04-18]] — follow-up: urgency gate generalizes but threshold is regime-dependent (TFO needs ≥ 6, S&C ≥ 4); single-names dominate the edge, indices don't produce enough urgency signal; 14:00 ET suppression is additive to urgency gate, not redundant. Consolidated entry policy in §4.
- [[small-pullback-trend-alignment-filters-2026-04-18]] — follow-up: `always_in != opposed` and `gap_direction = aligned` are additive filters; short-side WR ~20–30pp below long-side in this window (bull-regime bias or structural — open); winners plateau at bar 20–25 (bar-25 time-stop candidate). Revised consolidated policy in §7.
- [[small-pullback-trend-time-stop-2026-04-18]] — follow-up: bar-N time-stop rejected (bar-20 costs 12R, bar-30 breakeven, bar-10 catastrophic). 96% of winners run past 3R MFE; holding dominates any mid-trade exit. Closes Q13.
- [[small-pullback-trend-adverse-event-2026-04-18]] — follow-up: seven event-triggered adverse-exit rules all worse than baseline (−24 to −103R / quarter). Rules fire on genuine SPT bar-texture, shooting 60–65% winners. Third rejected exit-refinement. Closes Q14.
- [[small-pullback-trend-entry-side-filters-2026-04-18]] — follow-up: refined time-of-day window `10:30–14:15 ET` adds +0.35R/trade (+24R/quarter) over current `suppress 13:45–14:45`. Lunch is the strongest zone (+1.42R/trade); 14:45–15:30 ET is the worst (−0.69R/trade). Signal-bar ATR ratio is not a filter. Short-side weakness (Q11) localized to opening hour. Closes Q15.
- [[small-pullback-trend-robustness-2026-04-18]] — follow-up: refreshed policy-stack (n=149, +1.19R/trade, 59.7% WR). Edge diversified across 62 tickers (leave-one-out stable) but concentrated in 8 of 39 days (81% R) — Brooks' "1–2×/month" rhythm in data. Ordinal effect: 3rd+ SPT detection on a ticker-day hits 70–100% WR; skip-1st lifts per-trade R by +24%. Opens Q16–Q18, closes Q19–Q20.
- [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] — follow-up: urgency-rescue of 1st detection REJECTED (high-urgency 1st is the *worst* cell, +0.19R at u≥8). `skip-1st` unconditional → +1.410R/trade. Market-wide ordinal (`mkt_ord`) gate adds: **skip-1st + mkt_ord > 2 → +1.519R/trade, 70.5% WR, n=61**. Closes Q16 & Q17, opens Q21 (max-trades-per-day cap). Adds rules 8–9 to policy stack.
- [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] — follow-up: Q11 short-side gap localized to 10:30–11:30 ET (9 losses in a row); Q21 reframed from hard-cap to **daily-confirmation gate** (first-3 ≥ 2 wins). Full compound stack: 82% WR, +2.00R/trade, n=33. Closes Q11 and Q21. Adds rule 9 (daily gate) and short-side ToD exclusion to rule 6.
- [[small-pullback-trend-low-n-days-2026-04-18]] — follow-up: Q22 resolution. 72% of trading days are low-n (< 4 triggers); damage concentrated in n=1 singletons (27 trades, 37% WR, +0.39R). Rule 7 (skip-1st) mechanically excludes every singleton. `day_n ≥ 2` is a no-op at full PLAYBOOK stack. Policy unchanged; Q22 closed.
- [[small-pullback-trend-PLAYBOOK]] — one-page consolidated trading playbook with the full 10-rule stack and expected economics.
- [[small-pullback-trend-INDEX]] — reading order for the 12-note research series with question-resolution map.
- [[always-in-direction]] — SPT direction = always-in direction, usually clear by bar 2–3.
- [[failed-breakout]] — the failing-side setups (FH1/FL1) that look compelling in SPT but lose.
- Scanner doc: `~/code/aiedge/scanner/DAYTYPE_SPEC.md`.
