# Trend research — 2026-04-20 · incr 28

**Forward-return validation of the live gate on ES futures.** Direction-survival
(incr 23/26) measured LABEL persistence; this pass measures whether the gate
actually moves PRICE. Headline: gate produces **+0.09R at 10-bar horizon
(57.1% hit) vs +0.04R baseline (50.3%)** on **968 gated reads / 4,887
directional observations across 101 ES.c.0 sessions**. Edge grows with horizon
through 20 bars then collapses to **−0.10R at EoD** (mean reversion). The live
gate is a genuine alpha for ~25–100 min holds; **do not hold to close**.

## Headline numbers

- **Sample**: 101 RTH ES.c.0 sessions (cache hit on incr 26 fetch). 4,887
  directional `compute_trend_state` observations after warmup k=10. Gate
  passes 968 (19.8%).
- **Mean directed R (gate vs random-sign baseline)**:
  - H = 5b (25 min): **+0.062R** (53.4%) vs **+0.031R** (49.5%) → **Δ +0.032R**
  - H = 10b (50 min): **+0.090R** (57.1%) vs **+0.039R** (50.3%) → **Δ +0.051R**
  - H = 20b (100 min): **+0.198R** (58.6%) vs **+0.116R** (50.6%) → **Δ +0.082R**
  - H = EoD: **−0.103R** (45.2%) vs **+0.071R** (51.0%) → **Δ −0.175R**
- **Edge curve is hump-shaped**: rises monotonically from 5b → 20b, collapses
  by EoD. The directional label persists (incr 23/26), but PRICE mean-reverts
  by close. The gate is a **momentum signal at ~25–100 min**, not a hold-to-close
  classifier.

## Where the edge concentrates

**By bar-k bucket at 10-bar horizon** (gated reads only):

| k bucket | n | mean R |
|---|---|---|
| 10–14 | 79 | **+0.47R** (huge — opening read) |
| 15–19 | 123 | +0.31R |
| **20–29** | **263** | **+0.19R** ← production gate's first valid window |
| 30–39 | 192 | +0.16R |
| 40–59 | 280 | +0.10R |
| **60–78** | **114** | **−0.27R** ← late-session gate is anti-edge |

The k≥20 production cut already filters out the strongest cells (k=10–14 at
+0.47R) by design — those reads are too early per incr 23 (reversibility risk).
**Within the production gate, edge tapers monotonically with bar-k**, going
strongly negative in the last 90 minutes of session (k=60–78). Late-session
mean reversion is the dominant force.

**By |strength| bucket at 10-bar horizon** (no k filter):

| |strength| | n | mean R | hit |
|---|---|---|---|
| 0.05–0.10 | 1,213 | +0.12R | 49.1% |
| 0.10–0.15 | 1,057 | +0.06R | 54.5% |
| 0.15–0.20 | 846 | **−0.07R** | 53.2% |
| **0.20–0.30** | **792** | **+0.16R** | **58.2%** |
| 0.30–0.50 | 259 | +0.09R | 57.9% |

The **|s| 0.15–0.20 bucket is anti-edge** (−0.07R); the gate threshold jumps
to **+0.16R at |s| 0.20–0.30**. **The ES |s| ≥ 0.20 cut chosen by incr 26 is
validated by forward returns, not just label persistence.** This is independent
evidence the asset-class threshold split (0.15 equity / 0.20 ES) is real.

The non-monotonicity in the lowest bucket (+0.12R at |s|=0.05–0.10) is a
sample-bank artifact: that bucket is dominated by long, mature mid-session
reads where direction is locked but strength has decayed. It is NOT a "low
strength is also informative" finding — separate stratification by k confirms
the low-strength bucket reads concentrate in k=20–60 where baseline forward
return is also positive.

## Headline figures

1. **`forward_return_gate_vs_baseline.png`** — mean R + hit rate by horizon,
   gate vs random-sign baseline. **Hump-shape: positive 5–20b, negative at
   EoD.**
2. **`forward_return_horizon_curve.png`** — mean R by horizon stratified by
   |strength| bucket. The |s|≥0.20 lines (teal/green) sit cleanly above the
   |s|<0.20 lines for H=10b/20b.
3. **`forward_return_by_k.png`** — mean R at H=10b by bar-k bucket, all reads
   vs gated. Shows the k=60–78 cliff visually.
4. **`forward_return_distribution.png`** — directed-R density at H=10b.
   Gate distribution is right-shifted of baseline; mean +0.09R vs +0.04R.

## Interpretation

The live gate is not a "label classifier" — it is a **forward-momentum
indicator with a horizon-dependent edge**. Three independent properties hold:

1. **Label persists** (incr 23/26): direction(k≥20, |s|≥0.20) survives to EoD
   with ~90% probability on ES.
2. **Price moves in label direction at short-mid horizons**: +0.09 to +0.20R
   at 5–20 bars, hit rate 53–59% (this pass).
3. **Price reverts by close**: −0.10R at EoD, hit rate 45.2%. Late-session
   reversal is structural, not noise.

**(1) and (3) coexist** because the label can persist while price chops within
the directional cluster, then reverts before close. This is consistent with the
Brooks "channel after spike → final test" reading: trend continues, then
profit-taking flushes back through the entry.

**Practical takeaway for any future live consumer of the gate**: treat the
gate as a 5–20 bar (25–100 min) momentum signal. Exit at horizon, not at
close. Late-session reads (k≥60) should be down-weighted or suppressed.

## Mistakes avoided this pass

1. **Didn't conflate label persistence with profitability.** The previous
   passes' "P(direction survives) ≈ 90% at gate" is a STATISTIC about the
   classifier; this pass measured what the trader actually banks. They are
   different objects and could have decoupled (a stable-but-flat label
   produces zero R despite 100% survival).

2. **Didn't quote raw points.** ES point value swings 4× across the sample
   (ATR 3.86 → 12.84). Per-session R-normalisation is the only honest unit
   for cross-session aggregation.

3. **Used the conservative entry assumption** (`close[k-1]` = the last close
   IN the read, not `open[k]` = the next bar). A live consumer can fill at
   the post-bar open without slippage; using `close[k-1]` understates real
   tradable edge by ~half a bar.

4. **Random-sign baseline at matched k-distribution.** A naive baseline (e.g.
   "always long") would conflate gate edge with the symbol's drift. Sign
   randomisation cancels drift; matched k-distribution cancels intraday
   timing-of-day effects.

5. **Reported all four horizons.** The hump-shape edge curve only appears
   when 5/10/20/EoD are all visible. Single-horizon reporting (e.g. EoD only)
   would have shown the gate as net-negative — which is wrong for the
   intended use case of an intraday signal.

## What's next — needs Will's nod

- **Add the 5–20 bar horizon as a display field on the dashboard card.** When
  the gate fires, show "expected R over next 50 min: +0.09R historical" as
  context. Don't wire it into ranking yet.
- **Suppress the gate at k ≥ 60 in any future live consumer.** Late-session
  reads are anti-edge by 0.27R at H=10b — actively misleading.
- **Repeat on equities.** The 200-session equity panel (incr 23) has the
  trend trajectory but not prices — would need a re-fetch via Databento +
  the same directed-R aggregation. Confirms whether the EoD reversal is
  ES-specific or universal.
- **Still pending from prior incrs:** ALWAYS_IN_WINDOW 5→10 (incr 20),
  MAJORITY_TREND_BAR_FLOOR 0.40→0.25 (incr 18), OPEN_EXTREME_THRESHOLD
  0.15→0.25 (incr 21), htf_alignment silent-zero docstring (incr 19),
  simplified live gate `|s|≥0.15 AND k≥20` with asset-class strength knob
  (incr 27).

## Data artifacts

- `forward_return_incr28_obs.csv` — 4,887-row per-(session,k) observation
  table with strength, structure, atr, entry close, and forward R/hit at all
  four horizons.
- `forward_return_incr28_baseline.csv` — 1,912-row random-sign baseline.
- `forward_return_incr28.csv` — per-cell aggregation (gate vs baseline ×
  horizon, plus per-strength bucket at H=10b).
- `forward_return_incr28.json` — summary with by-strength and by-k breakdowns.
- Tool: `scanner/tools/forward_return_incr28.py`.
- PDF: `pdfs/trend-research-2026-04-20-incr28.pdf`.
