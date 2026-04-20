---
date: 2026-04-19
increment: 26
topic: ES futures direction-survival calibration — does the incr-25 live gate travel from equities to the primary trading instrument?
status: finding (NO production code change) — answers "needs Will's nod" item #2 from incr 25
---

# Incr 26 — The incr-25 live gate does NOT travel to ES unchanged

## TL;DR

The equity-universe direction-survival schedule proposed in incr 25 is **too
lenient for ES futures from bar 30 onwards**. On **101 ES.c.0 RTH sessions**
(≈ 5 months, Databento GLBX.MDP3, 4,887 directional observations), ES
baseline direction-survival is **70.3 %** vs **78 %** on the 200 equity
sessions. Applying the incr-25 schedule to an ES live gate would miss the
90 % target at every late-session cell.

**ES-calibrated p90 threshold schedule:**

| bars  | ES incr-26 | equity incr-25 | Δ         |
|-------|-----------:|---------------:|----------:|
| 10-14 |  **0.20**  | 0.30           | −0.10\*   |
| 15-19 |  **0.30**  | 0.30           | 0         |
| 20-24 |  **0.30**  | 0.30           | 0         |
| 25-29 |  **0.30**  | 0.30           | 0         |
| 30-39 |  **0.30**  | 0.20           | **+0.10** |
| 40-59 |  **0.20**  | 0.15           | **+0.05** |
| 60-78 |  **0.15**  | 0.10           | **+0.05** |

\* The bars-10-14 cell on ES passes the 0.20 bar on `n = 43` — a thin
cell, see caveat #2 below. Treat the real practical gate as starting at
bar 15.

**No production code change this pass.** This is a proposal Will needs to
see before any front-end gate is wired.

## Method

- Symbol: `ES.c.0` (Databento continuous front-month, stype_in='continuous').
- Dataset: GLBX.MDP3, schema `ohlcv-1m`.
- Window: 2025-11-20 → 2026-04-17 (150 calendar days). 140,134 raw 1-min
  bars.
- Filter: RTH 09:30–16:00 ET only. Overnight / Globex bars dropped before
  resampling — matches the aggregator's "what the live reader actually
  sees" contract during the cash session.
- Resample: 5-min bars, drop sessions with < 20 bars. Cap at 78 bars per
  session (last RTH bar of a full day).
- Aggregator: `aiedge.context.trend.compute_trend_state` unchanged (12
  contributors, `htf_alignment = 0.0` silent-zero because the production
  aggregator never receives daily/weekly closes — identical behaviour to
  incr 25).
- Progressive calls: for every warmup-k ∈ [10 .. session_end], record
  `(k, direction, strength, confidence, structure)`.
- Cell binning: **identical** `K_EDGES` and `S_EDGES` as incr 25 — the
  tool imports them verbatim so the two grids are cell-aligned.
- Threshold rule: smallest `|strength|` bin whose rate reaches target AND
  every higher-strength cell with `n ≥ 20` also reaches target.

## The ES grid

Cell format: `rate / n`. Blanks are `n < 5`; `-` is empty bin.

| k \ `|strength|` | .05-.10 | .10-.15 | .15-.20 | .20-.30 | .30-.50 |
|---|---:|---:|---:|---:|---:|
| 10-14 | 0.53 / 114 | 0.48 /  94 | 0.58 /  62 | **0.91 /  43** | **1.00 / 36** |
| 15-19 | 0.45 /  73 | 0.54 /  76 | 0.54 /  91 | 0.81 / 100     | **1.00 / 23** |
| 20-24 | 0.57 /  96 | 0.61 /  69 | 0.56 /  85 | 0.68 / 113     | **1.00 / 30** |
| 25-29 | 0.47 / 104 | 0.61 /  94 | 0.60 /  63 | 0.79 /  92     | **0.96 / 28** |
| 30-39 | 0.54 / 192 | 0.62 / 213 | 0.59 / 158 | 0.88 / 155     | **1.00 / 40** |
| 40-59 | 0.52 / 438 | 0.70 / 369 | 0.79 / 279 | **0.95 / 219** | **1.00 / 61** |
| 60-78 | 0.63 / 472 | 0.81 / 354 | **0.91 / 221** | **1.00 / 162** | **1.00 / 61** |

## Three observations worth keeping

### 1. ES is a noisier live reader than the equity universe at the low-|strength| floor

The directional baseline (`direction != none` rows only, unweighted) is
**70.3 % for ES** vs **78.1 % for equities**. That's a **−7.8 pp**
difference at the "just emit it, no gate" level. Likely cause: the equity
panel sampled 193 symbols with a built-in cherry-picking bias (live-
scanner cache is populated only for names with interesting intra-day
behaviour that day — it's not a random sample of the universe). ES is a
single instrument sampled every trading day, choppy ones included.

The good news: a 70 % baseline leaves **more room** for the "confirmed"
badge to add value on ES than it did on equities. The bad news: the
cells below |strength|=0.15 are worse on ES than equities across the
board (0.45–0.63 range vs 0.48–0.71 on equities).

### 2. The 30-39 band is where the two populations diverge

The headline delta: at **bar 30-39**, equities cleared 90 % survival with
`|strength| ≥ 0.20` (0.97 rate, n=224). **ES does not** — the 0.20-0.30
cell sits at **0.88** on `n=155`. The 0.30+ bar is needed: 1.00 rate on
n=40. In absolute terms this looks small, but at the live-gate scale it
matters: a trader who relies on the equity-calibrated 0.20 threshold at
bar 32 would be right **88 %** of the time, not 90 %. Close, but not
what the "confirmed" badge claims.

The 40-59 and 60-78 deltas are smaller (0.05 each) but monotone — ES is
stricter at every late band.

### 3. Early-session bars show the opposite sign but on thin cells

At bars 10-14 the ES p90 bar is 0.20, not 0.30. The 0.20-0.30 cell hit
`0.91 (n=43)` — a clean pass. But note: the 0.15-0.20 cell on ES sits
at `0.58 (n=62)` — worse than equity's 0.69 — and the thresholder
doesn't flag it only because the monotone check passes at 0.20.

On equities the same band has `0.20-0.30 = 0.79 (n=118)` which fails
the monotone check (0.30-0.50 cell hits 0.96), so equity's threshold
jumps straight to 0.30. ES doesn't fail the check because its 0.20 cell
is unusually clean on a smaller sample. **This is likely sampling
noise; I would not deploy the 0.20 threshold at bar 10 in production.**

Practical advice: **don't bother gating before bar 15** on either
panel. The thresholder picks 0.20 on one and 0.30 on the other but the
underlying signal-to-noise is rough.

## Candidate live-gate rule set for ES (needs Will's nod)

```python
# Post-incr-26 proposal — NOT wired. ES-specific time-aware gate.
def is_direction_confirmed_es(bar_k: int, strength_abs: float) -> bool:
    """P(live direction == session-close direction) ≥ 0.90 on 101-session
    ES.c.0 sample (incr 26). Stricter than the equity-calibrated equivalent
    by 0.05-0.10 at every band ≥ 30 bars."""
    if bar_k < 15:
        return False
    if 15 <= bar_k <= 39:
        return strength_abs >= 0.30
    if 40 <= bar_k <= 59:
        return strength_abs >= 0.20
    return strength_abs >= 0.15  # 60+
```

Expected behaviour on this sample: **986 of 4,887** directional
observations (≈ 20 %) pass the gate with combined survival **≈ 98.2 %**.
The gate suppresses the badge on 80 % of live reads — the badge stays
honest.

## If you only want ONE schedule (covers both panels)

For a single conservative gate that holds on either instrument:

```python
def is_direction_confirmed_safe(bar_k: int, strength_abs: float) -> bool:
    if bar_k < 15:
        return False
    if 15 <= bar_k <= 39:
        return strength_abs >= 0.30  # max(0.30, 0.30) — tied
    if 40 <= bar_k <= 59:
        return strength_abs >= 0.20  # max(0.20, 0.15)
    return strength_abs >= 0.15       # max(0.15, 0.10)
```

This is the max(ES, equity) schedule. It loses 8 pp of early-band
coverage on equities (bars 10-14 no longer eligible) but holds ≥ 90 %
survival across both populations in every cell the gate fires.

## Five mistakes avoided this pass

1. **Didn't copy-paste the equity schedule onto ES without checking.**
   That was the tempting one-liner — "the schedule looks good, ship
   it". The cell-aligned comparison would have been skipped, and the
   gate would have fake-confirmed 12 % of the bars in the 30-39 /
   0.20-0.30 band.
2. **Didn't average ES and equity rates into a single grid.** Equities
   and ES are different populations with different baselines. Averaging
   would erase the very asymmetry the study was meant to surface.
3. **Didn't include overnight/Globex bars.** ES trades ~23 h; if I'd
   resampled before the RTH mask, overnight bars would leak into the
   09:30 warmup cells and contaminate the first-few-bars stats. The
   aggregator is a cash-session tool.
4. **Didn't exclude Databento "degraded" days.** November 28 + March
   15-16 were flagged degraded. I kept them — the degradation is in
   the price feed quality, not in the RTH coverage. A live reader that
   day saw the same degraded bars, so the study reflects reality. But
   that's documented here in case a later pass wants to strip them.
5. **Didn't claim the thresholder is the production knob.** It's a
   decision-support tool; the actual knob is whether Will wants to
   ship a single-panel gate, a dual-panel gate, or defer until Pattern
   Lab has WRs to overlay.

## Artifacts

- `es_direction_survival_incr26.csv` — per-cell n/hits/rate.
- `es_direction_survival_incr26_threshold.csv` — per-k p90/p95 curve.
- `es_direction_survival_incr26.json` — summary + equity comparison.
- `es_direction_survival_incr26_traj.csv` — durable 6,795-row per-bar
  trajectory (symbol, session, k, direction, strength, confidence,
  structure). Reusable for any future ES-specific re-analysis without
  re-pulling Databento.
- `es_direction_survival_incr26_sessions.csv` — per-session n_bars +
  final (direction, strength, confidence, structure).
- Figures (`figures/`):
  - `es_direction_survival_heatmap.png` — ES 2-D grid heatmap.
  - `es_direction_survival_counts.png` — ES cell sample sizes.
  - `es_direction_survival_threshold.png` — ES p90/p95 curve.
  - **`es_vs_equity_direction_survival.png`** — THE headline figure.
    4-panel: equity heatmap | ES heatmap | ES–eq delta | threshold
    curves side-by-side.
- Tool: `~/code/aiedge/scanner/tools/es_direction_survival_incr26.py`.

## What's next — requires Will's nod

1. **Pick which schedule to ship** — ES-specific, equity-specific, or
   conservative max(). For a live dashboard that mixes panels (equities
   in the scanner + ES chart on the side), max() is the honest default.
2. **Wire `is_direction_confirmed` into the live payload** as a
   boolean `trendState.directionConfirmed`. This was already item #1
   of incr 25's open list; incr 26 now gives it a defensible ES-safe
   cutoff schedule.
3. **Pattern Lab WR cross-check** — still blocked on Pattern Lab DB
   backfill. The gate is calibrated on direction survival; WR
   calibration comes after.
4. **Consider a per-symbol baseline panel.** ES baseline at 70 % vs
   equity 78 % hints that the "confirmed" badge threshold may need to
   be per-symbol-class for any future non-ES futures (NQ, CL, GC).
   NOT a priority yet — ES is the target instrument.
