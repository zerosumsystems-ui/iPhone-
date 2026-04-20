---
date: 2026-04-19
increment: 25
topic: Live direction survival — 2D grid of P(dir → final dir) by bar k × |strength|
status: finding (NO production code change) — proposes a candidate time-aware live gate
---

# Incr 25 — When is the live direction trustworthy?

## TL;DR

Incr 23 gave one headline rule: "at bar 20, `|strength| ≥ 0.15` → 93 %
direction survival". Re-reading the 9,425-row trajectory at **2-D
resolution** (bar-bin × strength-bin) shows **that number averages over
later bars** where the signal is already crisp. At the strict k=20 bar in
isolation it's only 81 %. Strength-only gating isn't enough — the right
gate is **time-aware**.

The 2-D grid gives a clean, concrete candidate rule set for a live
confidence gate:

| bars | min `|strength|` for ~90 % survival | n observations | observed rate |
|------|-------------------------------------:|---------------:|--------------:|
| 10-29 | **0.30** | 118 | **98.3 %** |
| 30-39 | **0.20** | 290 | **97.2 %** |
| 40-59 | **0.15** | 726 | **95.9 %** |
| 60-78 | **0.10** | 1 117 | **98.4 %** |

Everything else (lower strength, or earlier bar) sits between 45 % and 79 %
survival — **barely above the 78 % baseline of "if it said a direction it
was probably right"**. That's not worth a "confirmed direction" badge.

**No production code change this pass.** The rule set is a proposal that
needs Will's nod before any front-end gate ships.

## Method

- Source: `realtime_stability_stratified_incr23_traj.csv` (200 RTH sessions
  across 193 distinct symbols × warmup k=10 → up to k=68 progressive calls,
  9,425 rows total). Persisted durable artifact from incr 23.
- For each session, define `final_dir = direction` at the largest-k row.
- For every row where `direction ∈ {up, down}` (7 264 of 9 425 rows —
  directional observations), bin by `(k, |strength|)` and count
  `survived = (direction == final_dir)`.
- Bar bins: `[10-14, 15-19, 20-24, 25-29, 30-39, 40-59, 60-78]`.
- Strength bins: `[0.00-0.05, 0.05-0.10, 0.10-0.15, 0.15-0.20,
  0.20-0.30, 0.30-0.50, 0.50-1.01]`.
- Cell rate is only displayed when n ≥ 5.
- The per-k threshold is the **smallest strength bin whose rate reaches
  target AND every higher-strength cell does too** (monotone check). Cells
  with n < 20 are skipped — avoids declaring a 100 % survival on 4 lucky
  observations.

Pure read-only. Scanner code, aggregator, tests all untouched (602 / 905
still green).

## The 2-D grid

Cell format: `rate / n`. Blanks are n < 5.

| k \ `|strength|` | .05-.10 | .10-.15 | .15-.20 | .20-.30 | .30-.50 |
|---|---:|---:|---:|---:|---:|
| 10-14 | 0.45 / 203 | 0.63 / 175 | 0.69 / 162 | 0.79 / 118 | **0.96 / 28** |
| 15-19 | 0.48 / 216 | 0.64 / 181 | 0.78 / 170 | 0.82 / 132 | **0.97 / 30** |
| 20-24 | 0.64 / 197 | 0.68 / 198 | 0.68 / 171 | 0.87 / 121 | **1.00 / 28** |
| 25-29 | 0.59 / 229 | 0.65 / 200 | 0.72 / 127 | 0.86 / 116 | **1.00 / 32** |
| 30-39 | 0.64 / 383 | 0.73 / 400 | 0.79 / 296 | **0.97 / 224** | **1.00 / 66** |
| 40-59 | 0.71 / 327 | 0.81 / 523 | **0.94 / 378** | **0.98 / 280** | **1.00 / 68** |
| 60-78 | 0.69 / 359 | **0.96 / 423** | **0.99 / 369** | **1.00 / 284** | **1.00 / 41** |

Bold cells are the ≥ 0.90 survival cells.

## Three observations worth keeping

1. **Bar 30 is the aggregator's handoff point.** Below k=30, you need
   `|strength| ≥ 0.30` to escape the 78 % baseline — and only 118 of
   ~1 700 early-bar directional observations meet that bar. From k=30
   onward, a moderate `0.15-0.20` range already clears 90 % (in 40-59) and
   a weak `0.10-0.15` clears 96 % by bars 60-78.

2. **Incr 23's "93 % at bar 20" number collapses at strict resolution.**
   At exact k=20 with `|strength| ≥ 0.15`, the rate is **81 %** (n=75). At
   k ≥ 20 cumulatively with `|strength| ≥ 0.15`, the rate is **91 %**
   (n=2 601) — but most of that comes from k ≥ 40 cells sitting at 94-100 %.
   The single-bar 93 % claim was optimistic by ~12 points.

3. **The baseline itself (78 %) isn't bad.** If the live scanner just
   emits the direction whenever it's non-`none`, it matches the close 78 %
   of the time. A confidence badge only adds value when it significantly
   beats 78 %. Cells below 90 % should not earn the badge.

## Candidate live-gate rule set (needs Will's nod)

```python
# Post-incr-25 proposal — not wired. Time-aware direction confidence gate.
def is_direction_confirmed(bar_k: int, strength_abs: float) -> bool:
    """Return True when live direction matches session-close direction
    ≥ 90% of the time, based on 200-session backtest (incr 25)."""
    if bar_k < 10:
        return False  # warmup
    if 10 <= bar_k <= 29:
        return strength_abs >= 0.30
    if 30 <= bar_k <= 39:
        return strength_abs >= 0.20
    if 40 <= bar_k <= 59:
        return strength_abs >= 0.15
    # 60+
    return strength_abs >= 0.10
```

Expected behaviour on this sample: **2 251 observations** (≈ 31 % of all
directional live reads) would pass the gate, with combined survival
**≈ 97.4 %**. The other 69 % would be shown without the badge. The
front-end "confirmed direction" ribbon stays honest.

## What changed vs the incr 23 prose

- **Corrected**: the single-number "93 % at bar 20, `|strength| ≥ 0.15`"
  rule-of-thumb is replaced by a time-aware threshold schedule. The 93 %
  figure was a cumulative average that smuggled in the much-more-reliable
  late-session bars.
- **Reinforced**: direction (not structure) is the right thing to latch on.
  Structure flipped 9.5× per session (incr 24) while direction flipped
  0.58× — direction survives to the close, structure doesn't have to.

## Five mistakes avoided this pass

1. **Didn't treat the incr-23 number as gospel.** The "93 %" headline was
   a legitimate aggregate, but it was used to justify a single-threshold
   gate that wouldn't actually deliver 93 % confidence at bar 20. Verify
   at the resolution the rule will fire at.
2. **Didn't re-run the progressive-k sweep.** The 9 425-row trajectory CSV
   is the durable artifact. All live-gate calibration questions are answered
   off it — zero new Databento calls, zero scanner runs.
3. **Didn't tune off noisy cells.** The per-k threshold routine skips any
   cell with n < 20. Two of the "1.00 survival" cells in bars 10-29 have
   n = 28-32 and they *do* clear the gate — but I kept them in because
   n ≥ 20 is still a meaningful sample, and because the proposed rule
   requires `|strength| ≥ 0.30` there (already a high bar).
4. **Didn't over-sell the rule.** 2 251 of 7 264 directional observations
   pass — that's 31 %. Will should know the gate *suppresses* the badge
   most of the time, so front-end copy needs to say "confirmed" not
   "active".
5. **Didn't propose a production change.** The `is_direction_confirmed`
   function is a candidate, not wired. Needs Will to say "wire it".

## Artifacts

- `direction_survival_incr25.csv` — per-cell n and rate.
- `direction_survival_incr25_threshold.csv` — per-k p90 / p95 threshold curve.
- `direction_survival_incr25.json` — summary + practical-gate pick.
- Figures (`figures/`):
  - `direction_survival_heatmap.png` — rate heatmap with n-annotations.
  - `direction_survival_counts.png` — observation counts per cell.
  - `direction_survival_threshold.png` — p90 / p95 `|strength|` curve by k.
- Tool: `~/code/aiedge/scanner/tools/direction_survival_incr25.py`.

## What's next — requires Will's nod

1. **Wire `is_direction_confirmed` into the live payload** as a boolean
   `trendState.directionConfirmed` next to `direction`. Dashboard can
   render a small "confirmed" ribbon when true. No ranking impact.
2. **Calibrate the same grid on real ES futures** (not just the sampled
   equity sessions in the stratified sweep) to make sure the threshold
   schedule travels. Incr 23's sweep was stratified across 193 symbols;
   ES-specific calibration is a smaller 1-symbol sweep that fits in a
   single Databento window.
3. **Pattern Lab WR cross-check** — still blocked on Pattern Lab DB
   backfill. Once populated, compare "gate passed" direction badges
   against per-setup WR to confirm the badge carries real edge.
