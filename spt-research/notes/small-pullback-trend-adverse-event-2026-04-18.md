# SPT — Event-Triggered Adverse Exits (2026-04-18, pt 7)

Eighth follow-up to [[small-pullback-trend]]. Closes the "New — untested" item from [[small-pullback-trend-time-stop-2026-04-18]] §7:

> conditional exit on *adverse* event (e.g. first bear trend bar closing below bar-10 EMA on a long). Distinct from a pure time-stop.

Result: **rejected — every event rule costs R vs baseline hold.** The Brooks SPT paradox shows up cleanly: the very bars the rule treats as "adverse" are exactly the "countertrend trend bars" and "weak with-trend signal bars" Brooks says are diagnostic of strength, not weakness.

Population: same urgency-gated SPT policy stack — H1/H2/L1/L2 in `trend_from_open` + `spike_and_channel`, `urgency >= 4`, `gap_direction` aligned with setup. **n=153 resolved.** Baseline: 3R target, 1R stop, hold.

Snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite` @ 2026-04-18 afternoon.
Script: `~/code/aiedge/scanner/scratch/spt_adverse_event_analysis.py`.

## 1. The rules tested

Each rule walks the post-signal bars (chart_json, up to 20 bars forward). At each bar, the stop (1R) and target (3R) are checked intrabar first. If neither fires and the rule's trigger condition is met at that bar's close, the trade exits at that close. Chart-exhausted trades fall back to the stored mfe/mae baseline.

| Rule | Trigger condition (long; invert for short) |
|------|--------------------------------------------|
| `first_adverse_close` | First bar with `close < entry` |
| `two_adverse_closes` | Two consecutive bars with `close < open` (bear bars) |
| `first_adverse_trend_bar` | First bear trend bar (body ≥ 40% of range) |
| `break_prior_swing` | First bar with `close < prior bar's low` |
| `close_beyond_10ema` | First bar with `close < 10-EMA` |
| `trend_bar_beyond_10ema` | Brooks' exact hypothesis: bear trend bar closing below 10-EMA |
| `trend_bar_beyond_20ema` | Same, looser MA |

Grace period: 1 bar (first trigger eligible is `bar_index + 1`).

## 2. Full-population results (n=153, 3R target)

| Rule | WR% | Sum R | Per-trade | Δ vs baseline |
|------|----:|------:|----------:|--------------:|
| **baseline_hold** | **52.9%** | **+139.51** | **+0.91R** | — |
| trend_bar_beyond_20ema | 49.0% | +115.34 | +0.75R | −24.17R |
| close_beyond_10ema | 45.1% | +76.28 | +0.50R | −63.23R |
| trend_bar_beyond_10ema | 45.8% | +75.23 | +0.49R | −64.28R |
| break_prior_swing | 45.8% | +64.88 | +0.42R | −74.63R |
| two_adverse_closes | 43.8% | +60.44 | +0.40R | −79.08R |
| first_adverse_close | 24.8% | +45.37 | +0.30R | −94.14R |
| first_adverse_trend_bar | 45.1% | +36.54 | +0.24R | **−102.98R** |

Observations:

- **No rule beats baseline.** The best (20-EMA variant) gives back −24R / quarter; the worst (first adverse trend bar) gives back −103R.
- Tighter rules are worse. The more sensitive the adverse-event trigger (first adverse *close*, first adverse *trend bar*), the more R is destroyed. Looser rules (20-EMA vs 10-EMA) leave more money on the table.
- The tightest rule (`first_adverse_close`) collapses WR to **24.8%** and clips expectancy by −0.61R/trade. It fires on 81/153 trades — essentially any normal within-channel pullback trips it.

## 3. Conditional-lift view — what the rule trades

The cleanest read is: *on the subset of trades where the rule fires, what was the baseline outcome?* If the rule is identifying real reversals, the baseline WR on fire-subset should be low (baseline would have given back R on those). If baseline WR is high, the rule is shooting winners.

| Rule | Fires | Base sum on fire-subset | Rule sum | Δ | Base WR when fired |
|------|------:|------------------------:|---------:|---:|-------------------:|
| first_adverse_close | 81 | +71.07 | −23.07 | −94.14 | 53.1% |
| two_adverse_closes | 77 | +90.37 | +11.30 | −79.08 | **63.6%** |
| first_adverse_trend_bar | 96 | +123.91 | +20.93 | −102.98 | **64.6%** |
| break_prior_swing | 79 | +88.91 | +14.27 | −74.63 | 62.0% |
| close_beyond_10ema | 67 | +62.31 | −0.92 | −63.23 | 59.7% |
| trend_bar_beyond_10ema | 65 | +62.37 | −1.91 | −64.28 | 60.0% |
| trend_bar_beyond_20ema | 29 | +21.41 | −2.76 | −24.17 | 58.6% |

**The rule is shooting winners.** 60–65% of trades that trip the adverse-event rule would have resolved as full 3R winners. The Brooks dictum verbatim: in an SPT channel, bear trend bars and 10-EMA breaks are features of the grind, not reversal signals.

The `first_adverse_trend_bar` row is the starkest — it fires on 96 trades with a 64.6% baseline win rate. Shooting winners 62 times in exchange for a few pennies of loss protection on the losers.

## 4. Direction and day-type splits

**By direction** (3R target):

| Direction | n | Baseline | Best alt rule | Δ |
|-----------|--:|---------:|--------------:|--:|
| long | 92 | +1.18R/tr | trend_bar_beyond_20ema +1.01R | −0.17R |
| short | 61 | +0.51R/tr | close_beyond_10ema +0.38R | −0.13R |

Direction-agnostic verdict — rules hurt roughly equally on each side, proportional to the underlying baseline.

**By day-type** (3R target):

| Day-type | n | Baseline | Best alt rule | Δ |
|----------|--:|---------:|--------------:|--:|
| trend_from_open | 107 | +0.86R/tr | trend_bar_beyond_20ema +0.79R | −0.07R |
| spike_and_channel | 46 | +1.03R/tr | trend_bar_beyond_20ema +0.68R | −0.35R |

S&C is hurt more (−0.35R/trade) than TFO (−0.07R). S&C channel-phase winners tend to thrust harder and through more noise — the 20-EMA break often coincides with the deep pullback that later delivers the biggest R. Clipping those is costly.

## 5. Why every rule fails (mechanism)

Going back to Brooks, *Trading Price Action: Trends*, ch. 57 and ch. 47:

> Many countertrend trend bars and weak with-trend signal bars. Buy signals look bad; bear bars look compelling. This is diagnostic, not disqualifying.

> Beginners: see weak setups → short → trapped. Experienced: recognize tape → take every ugly with-trend setup.

An event-triggered adverse exit is the mechanical embodiment of the beginner trap. It interprets the normal texture of an SPT day — bear trend bars on long side, 10-EMA taps, bars that close beyond prior lows — as evidence of failure. Brooks' text says the opposite: these are the *signature* of the regime, and the correct response is to keep holding.

The 2R target confusion ([[small-pullback-trend-targets-and-urgency-2026-04-18]]) and the bar-N time-stop failure ([[small-pullback-trend-time-stop-2026-04-18]]) are two earlier versions of the same mistake. This is the third: every "exit early when things look wrong" rule fails because **things look wrong on purpose** in an SPT regime, and the urgency-gated filter already removed the trades where it means something.

## 6. Exit-reason distributions

For the two Brooks-anchored rules:

`trend_bar_beyond_10ema` (n=153):
- rule fires: 65 (42%)
- chart exhausts with no rule / stop / target: 39 (26%)
- target first: 32 (21%)
- stop first: 17 (11%)

`trend_bar_beyond_20ema`:
- chart exhausts: 51 (33%)
- target first: 43 (28%)
- stop first: 30 (20%)
- rule fires: 29 (19%)

The tighter 10-EMA rule triggers on 42% of trades — 60% of which would have been winners. The looser 20-EMA rule only triggers on 19% — but still shoots 59% winners.

## 7. Verdict on the "New — untested" item

**CLOSED — rejected.** No event-triggered adverse-exit rule beats baseline hold on urgency-gated SPT with-trend trades. The three best (20-EMA, 10-EMA variants, close_beyond_10ema) lose −24 to −64R per quarter; the worst (`first_adverse_trend_bar`, `first_adverse_close`) lose −94 to −103R per quarter.

This is the third rejected "exit early" policy on the SPT population:
1. Narrow 2R target vs 3R: costs +0.43R/trade ([[small-pullback-trend-targets-and-urgency-2026-04-18]] Q7).
2. Bar-N time-stop: costs up to −52R / quarter depending on N ([[small-pullback-trend-time-stop-2026-04-18]] Q13).
3. Event-triggered adverse exit: costs −24 to −103R / quarter (this note, Q14).

All three fail for the same reason: the upstream urgency + gap + always_in filter stack is already doing the hard work. What remains are genuinely SPT-native trades, and those reward holding.

**Consolidated policy unchanged** (from [[small-pullback-trend-urgency-generalization-2026-04-18]] §4 + [[small-pullback-trend-alignment-filters-2026-04-18]] §7):
- `urgency ≥ 4` (≥ 6 on pure TFO), with-trend only, `gap_direction` aligned, `always_in != opposed`.
- 3R fixed target, 1R stop, **hold to resolution**.
- Suppress entries 13:45–14:45 ET.
- No bar-N time-stop. No event-triggered adverse exit.

## 8. What remains open

- **Q1 (blocked)** — SPT-component-raw bucketed WR. Unchanged.
- **Q5 (open)** — cross-asset SPT. Unchanged; blocked on multi-asset expansion.
- **Q11 (open)** — short-side weakness. Unchanged; revisit when data spans a bear leg.
- **NEW — not worth pursuing**: the space of exit-improvement rules (time-stop, event-stop, target-widening already done) seems fully explored within this population. Further gains likely come from *entry-side* filtering — e.g., a volatility / ATR quality check or a time-of-day bucket finer than the 14:00 ET band — rather than mid-trade exit refinements.

## Related

- [[small-pullback-trend]] — parent concept, research queue (Q14 added).
- [[small-pullback-trend-time-stop-2026-04-18]] — bar-N time-stop rejected; this note's predecessor and where Q14 was proposed.
- [[small-pullback-trend-alignment-filters-2026-04-18]] — `gap_aligned` + `always_in` filters, bar-20 winner plateau observation that motivated the original time-stop idea.
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — 3R > 2R target finding, `urgency ≥ 4` derivation.
- Analysis: `~/code/aiedge/scanner/scratch/spt_adverse_event_analysis.py`.
