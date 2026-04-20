# Trend research — 2026-04-20 · incr 29

## Equity forward-return validation: the EoD reversal is ES-specific

### TL;DR

Incr 28 found the live gate (|strength| ≥ 0.20, k ≥ 20) on **ES futures** has a **hump-shaped edge curve**: +0.20R at 20 bars, then **−0.10R at end-of-session**. It left open whether that reversal is universal or asset-specific. This pass repeats the same analysis on **120 US equity RTH sessions / 58 symbols / 4,478 directional reads** from the scanner's 1-min cache. **Answer: equities do NOT have the reversal.** The equity edge rises **monotonically** from +0.40R at 5b → +0.77R at 10b → +1.63R at 20b → **+1.71R at EoD**. Gate hit rate climbs to **73% at 20b and stays at 67% at EoD**. **The incr 28 "suppress gate at k ≥ 60" rule is ES-specific and should NOT be applied to equities.**

---

### What changed vs incr 28

Only the data source. Same `compute_trend_state`, same warmup k=10, same ATR-normalised directed-R entry (close[k-1] = last close in the read), same 5b/10b/20b/EoD horizons, same random-sign matched-k baseline, same strength and k-bucket stratifications. The *only* substantive difference: **US equity 1-min bars from the scanner cache** (120 sessions · 58 distinct symbols · capped at 4 sessions/symbol to avoid mega-cap domination), not ES.c.0.

Two gates reported this time for the travelability test:
- `|s| ≥ 0.15` — the equity-calibrated threshold from incr 23 (incr 26 established this is the equity-side knob)
- `|s| ≥ 0.20` — the ES knob, applied unchanged to equities (does it travel?)

---

### Headline numbers — the shape is clean

**Equity gate `|s| ≥ 0.15 AND k ≥ 20`** (n = 1,804 gated reads across 4,478 directional):

| horizon | n (gate) | mean R | median R | hit rate | baseline R |
|---:|---:|---:|---:|---:|---:|
| 5 bars (25 min) | 1,605 | **+0.396R** | +0.352R | **60.1 %** | −0.030R |
| 10 bars (50 min) | 1,418 | **+0.765R** | +0.631R | **63.3 %** | −0.057R |
| 20 bars (100 min) | 1,032 | **+1.628R** | +1.454R | **73.2 %** | −0.047R |
| **End of session** | 1,762 | **+1.708R** | +1.475R | **67.0 %** | −0.039R |

**ES incr 28 — same table for comparison** (|s|≥0.20, 101 sessions):

| horizon | mean R | hit rate |
|---:|---:|---:|
| 5b | +0.062R | 53.4 % |
| 10b | +0.090R | 57.1 % |
| 20b | +0.198R | 58.6 % |
| **EoD** | **−0.103R** | **45.2 %** |

Key contrast: ES mean at EoD is **negative**; equity mean at EoD is **+1.71R with 67% hit rate**. Equity hit rate climbs monotonically (60 → 63 → 73 %) and only softens to 67 % at close — still well above coin-flip.

The stricter `|s| ≥ 0.20` gate on equities is EVEN better than the `|s| ≥ 0.15` gate:

- H=10b: +0.88R (66.5%) vs +0.77R (63.3%)
- H=20b: +1.76R (78.3%) vs +1.63R (73.2%)
- H=EoD: **+2.10R (72.7%)** vs +1.71R (67.0%)

So the ES threshold knob (|s|≥0.20) DOES travel to equities — it's STRICTER and gives higher expectancy per trade. The open-loop question "does the ES knob work on equities" is yes.

---

### Strength gradient — rising through mid-band

At H=10b (gated reads only), stratified by `|strength|` bucket:

| \|strength\| bucket | n | mean R | hit rate |
|---:|---:|---:|---:|
| 0.05–0.10 | 880 | +0.17R | 53.6 % |
| 0.10–0.15 | 903 | +0.36R | 57.1 % |
| 0.15–0.20 | 707 | +0.53R | 57.9 % |
| 0.20–0.30 | 764 | +0.58R | 59.3 % |
| 0.30–0.50 | 339 | +0.66R | 66.1 % |
| 0.50–1.01 | 6 | −0.53R | 33.3 % ← too thin (n=6), ignore |

Strength IS informative for forward returns on equities. Monotonic climb through the 0.30–0.50 bucket (66 % hit rate). The 0.50+ tail is under-sampled — the aggregator structurally damps to that magnitude only in extreme sessions. **The equity-specific `|s|≥0.15` cut is defensible by forward-R** (the 0.10–0.15 bucket at +0.36R is the lowest band that's meaningfully positive; below that, the 0.05–0.10 bucket at +0.17R is still positive but close to noise).

---

### The headline figure

See `figures/forward_return_eq_vs_es_horizon.png` — equity hump (which isn't a hump, it's a ramp) vs ES hump (which IS a hump) side-by-side across horizons. The two curves diverge at H=20b and stay diverged through EoD.

---

### Why the magnitude differs (8–10× ES) — the honest caveat

Equity gate mean R (+1.71R at EoD) is an order of magnitude larger than ES gate mean R (+0.20R at 20b / −0.10R at EoD). This is real in the sample, but comes with two caveats:

1. **Scanner-selection bias.** The equity 1-min cache is populated by the live scanner during trading hours. It doesn't mirror the universe — it mirrors the sessions the scanner engaged with. Those are disproportionately sessions with unusual momentum or volatility. So "average equity session" is over-represented by "interesting equity session" here. ES, by contrast, is every RTH session in the 150-day window — unbiased.

2. **Per-session ATR normalisation interacts with low-priced stocks.** A handful of biotech / small-cap names in the sample have ATR < $0.01. When ATR is tiny, even small post-gate moves yield huge R. Filtering sessions to `atr ≥ 0.10` barely moves the mean (+1.628 → +1.622 at 20b, +1.708 → +1.677 at EoD) — so outliers aren't driving the magnitude — but the magnitude itself is inflated by the fact that the corpus over-represents volatile names.

**Bottom line on magnitudes**: The ABSOLUTE equity magnitudes don't travel without a representative-universe re-sample. The SHAPE of the curve — rising monotonically through EoD rather than humping and reverting — is the portable finding.

---

### Interpretation

The **hump-shape on ES is futures-specific**. Most likely drivers:

- ES end-of-day flows: profit-taking, VWAP reversion, hedging unwinds. Index futures have a disproportionate share of institutional roll / hedge activity that concentrates into the close.
- Equity end-of-day flows go the OTHER way: late-session retail chasing, momentum ETF rebalancing into names that moved, passive-index buying on the close auction — these reinforce the direction rather than reverting it.
- Or: the ES sample (150 calendar days) includes more mean-reverting session shapes than the equity cache (scanner-biased toward trending sessions).

Practical implication for any consumer that shows the live gate on BOTH asset classes:

- **Don't apply a single "suppress at k ≥ 60" rule globally.** It's defensible on ES and actively harmful on equities (where the late-session gate is the STRONGEST-expectancy window).
- The asset-class strength split (incr 26: 0.15 equity / 0.20 ES) should stay.
- The asset-class horizon guidance is NEW: *ES* wants a ~25–100 min holding window; *equities* tolerate hold-to-close.

---

### Mistakes avoided this pass

1. **Didn't copy-paste the ES schedule onto equities.** The incr 28 recommendation to suppress the gate at k ≥ 60 was calibrated on ES; applying it unchanged to equities would erase the strongest expectancy window on that asset class. Verified the k-bucket figure and confirmed the late-session equity reads are fine.

2. **Didn't treat mean as sufficient — also checked median and P5/P95.** Mean (+1.63R) and median (+1.45R) at 20b are nearly identical; the distribution is shifted right, not driven by outliers. The ATR-filter sanity check confirmed low-ATR biotech outliers aren't inflating the number.

3. **Didn't report magnitudes without the selection-bias caveat.** The equity cache is not a random sample of the universe — it's "sessions the scanner cared about." The SHAPE is portable; the MAGNITUDE is not without a representative re-sample. Documented prominently.

4. **Reported BOTH the equity-native gate (|s|≥0.15) AND the ES knob ported to equities (|s|≥0.20).** Single-gate reports would have hidden the fact that the stricter ES knob actually works fine on equities — it's stricter (only 1,053 vs 1,762 EoD reads) but higher-expectancy per trade (+2.10R vs +1.71R). Both knobs work on equities.

5. **Didn't conflate "edge grows to EoD" with "hold-to-close is the right strategy."** The per-trade variance is enormous at EoD (P5 to P95 span: −5.6R to +9.5R). A risk-adjusted exit at 20b (+1.63R, P95 = +6.79R) captures ~95% of the EoD expectancy at half the variance. The trader's choice, but the note shows both.

---

### What's next — needs Will's nod

1. **Asset-class-aware late-session rule.** ES suppresses gate at k ≥ 60 (incr 28); equities do NOT. Any dashboard showing the live gate across asset classes needs an `asset_class` axis for this rule, not a single threshold.

2. **Representative equity re-fetch.** This pass uses the scanner-cache population. To measure the TRUE magnitude on an unbiased universe, re-fetch 1-min equity bars for a fixed 150-day window on a fixed symbol list (e.g., S&P 100 constituents) via Databento's XNAS.ITCH, rerun the analysis, and compare equity mean R before claiming +1.71R at EoD is production-bankable.

3. **Horizon-aware display (ES + equity).** The incr 28 rec to show "expected R over next 50 min" on the live card now has two regimes: ES caps at 25–100 min, equities scale through EoD. Needs a copy change if both asset classes surface.

4. **Carried forward (no change):** all five prior threshold recs (incr 18/19/20/21 + simplified live gate from incr 27) remain pending.

---

### Files touched

- **Tool:** `scanner/tools/forward_return_equity_incr29.py` (new, ~440 LOC)
- **Data:** `vault/Scanner/methodology/forward_return_eq_incr29.{csv,json}` + observation CSV (4,478 rows) + baseline CSV (3,560 rows)
- **Figures:** 4 PNGs in `vault/Scanner/methodology/figures/`
  - `forward_return_eq_vs_es_horizon.png` ← HEADLINE (equity ramp vs ES hump, side-by-side)
  - `forward_return_eq_gate_vs_baseline.png` — 4-horizon bar pair
  - `forward_return_eq_horizon_curve.png` — stratified by |s| bucket
  - `forward_return_eq_by_k.png` — stratified by bar-k bucket
- **Scanner code:** zero changes. Aggregator, schema, tests byte-stable.
- **Test suite:** unchanged (602 classes / 905 subtests still passing — no scanner production code touched).

---

*Findings site: https://github.com/zerosumsystems-ui/iPhone-/tree/main/trend-classification*
