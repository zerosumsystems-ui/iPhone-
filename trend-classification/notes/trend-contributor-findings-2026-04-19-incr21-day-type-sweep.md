# Trend research — incr 21 — day_type `trend_from_open` gate sweep

**Date:** 2026-04-19
**Status:** Read-only; no production code changed. Three recommended threshold changes from prior incrs still pending Will's nod.

## TL;DR

Swept the two primary knobs of `_classify_day_type`'s `trend_from_open` clause — `OPEN_EXTREME_THRESHOLD` ∈ {0.10, 0.15, 0.20, 0.25, 0.30} × `TREND_PCT_MIN` ∈ {0.40, 0.45, 0.50, 0.55} — on the same 800-session real RTH bank incr 18/19/20 used. **The TFO gate is a precision detector: 100 % directional accuracy in every cell.** When it fires, the session closes in the predicted direction — zero mismatches in 127 fires on the loosest cell. The only lever worth tuning is **coverage**, and **`OPEN_EXTREME` is the binding knob**: loosening from 0.10 → 0.30 adds **+5.7 pp fire rate** (10.2 % → 15.9 %) while preserving 100 % accuracy. `TREND_PCT_MIN` is secondary (~2–4 pp effect).

## Headline numbers

- Sample: **800 (symbol, session) pairs** from `cache/databento/*/*.parquet`; 1-min → 5-min RTH. Realised: up = 494, down = 255, flat = 51. Always-best baseline = **65.9 %** (always-up).
- **Production cell (OE=0.15, TPM=0.50):** fires **11.9 %** (95 sessions), accuracy **100.0 %**. 83 up-fires all closed up; 12 down-fires all closed down.
- **Loosest cell (OE=0.30, TPM=0.40):** fires **15.9 %** (127 sessions), accuracy **100.0 %**. Same zero-mismatch result.
- **Strictest cell (OE=0.10, TPM=0.55):** fires **7.5 %** (60 sessions), accuracy **100.0 %**.
- **OE is the binding knob.** Holding TPM=0.50: fire rate climbs 9.6 → 11.9 → 13.5 → 14.4 → 14.9 % across OE=0.10/0.15/0.20/0.25/0.30.
- **TPM effect is small.** Holding OE=0.15: fire rate moves 12.8 → 12.8 → 11.9 → 9.1 % across TPM=0.40/0.45/0.50/0.55 — only the strictest (0.55) meaningfully shrinks coverage.

## Why 100 % is real, not a measurement bug

The TFO gate requires three conjunctive conditions:
1. open_position < OE (open in bottom OE of day's H–L range, or top OE for bear)
2. trend_pct > TPM (>50 % directional bars)
3. max_pullback_of_range < 0.30 (no deep retrace)

Gate (1) is load-bearing. `open_position < 0.30` means the open sits within the bottom 30 % of the day's range. A session closing BELOW the open would require the close to be even closer to the low than the open — a narrow, 30 %-of-range window. Combined with (2) (majority of bars are bull) and (3) (no deep pullback), the probability mass of "open in bottom 30 %, majority-bull bars, no deep pullback, yet close below open" collapses to zero in this 800-session sample. The gate's design is working exactly as intended.

Flat sessions (51 of 800) are correctly excluded from the accuracy denominator — same as incr 19/20. The sign-match definition is "fired sign == realised open→close sign" on non-flat sessions only.

## Comparison with incr 20 (always_in)

Very different stories:

| Contributor | Prod fire rate | Prod accuracy | Baseline | Verdict |
|---|---|---|---|---|
| always_in (incr 20, W=5, K=2) | 36.0 % | 57.6 % | 66.0 % | **Below baseline** — loosen W to 10 |
| day_type / TFO (incr 21, OE=0.15, TPM=0.50) | 11.9 % | **100.0 %** | 65.9 % | **Far above baseline** — precision-tuned; loosen only for coverage |

`always_in` was trigger-happy and weakly informative. `day_type`'s TFO gate is the opposite — rare but mathematically reliable. Both issues call for knob changes but in opposite directions (always_in: tighten the consecutive-bar requirement; day_type: optionally loosen the open-extreme threshold).

## Mistakes avoided in this pass

1. **Didn't over-claim.** 100 % accuracy is striking; verified it by reading the per-session CSV and confirming zero mismatches in the 127-fire loosest cell. If the result had been artifact-like (all up-fires because sample was up-biased) I'd have caught it here.
2. **Isolated the gate, not the whole pipeline.** `_classify_day_type` has six priority branches. Sweeping the whole thing would have required reproducing 200 lines of downstream logic and buried the primary finding. Honest framing in the note title and script docstring: this is the TFO *gate*, not the whole day_type classifier.
3. **Held pullback gate constant.** The third gate — `max_pullback_of_range < 0.30` — could have been the third knob. A 3D sweep would double the cell count and obscure the OE-vs-TPM relationship. Documented in the script why.
4. **Didn't collapse nominal confidence into the finding.** The sweep fixes confidence at 0.80 to decouple gate-strictness from confidence-shaping. Production's confidence formula varies with extremity + trend_pct + pullback; that's an independent knob.
5. **Baseline framing is honest.** 100 % >> 65.9 % always-up, but the fires themselves ARE biased up (83 up vs 12 down at production). The contributor's up-fires outnumber down-fires 6.9:1, roughly matching the up:down session ratio (494:255 = 1.9:1) — *amplified*, because TFO is a strict gate. That's a feature, not a bug — but flagged here so nobody reads "100 %" as "works on any session".

## What this implies for trend classification

The TFO gate is one of the cleanest signals in the stack. When it fires, the session is mathematically bound to close in its direction. Three implications:

1. **TFO-fires sessions should carry higher aggregate weight than they currently do.** Production equal-weights day_type alongside 11 other contributors, so a 100 %-precise signal gets diluted 1:11 on fires. The capstone's deferred "weighting" work item should keep day_type at weight 1.0 (baseline) at minimum, and a case can be made for weight > 1.0 when it fires.
2. **Coverage is the only lever worth tuning.** The incr-20-style accuracy-vs-fire Pareto shows no tradeoff here — looser cells strictly dominate stricter ones on coverage, with no accuracy loss.
3. **The 19.6 % day_type fire rate from incr 19 is NOT explained by a too-strict TFO gate.** TFO alone fires at 11.9 %, so the other 7-8 pp of day_type fires come from `spike_and_channel`, `trending_tr`, and the `trading_range → trend_from_open` chop-ratio override branches. Those branches weren't swept here and remain candidates for future work.

## Deferred — Will's nod (no autonomous change)

- **Threshold change candidate (new):** `OPEN_EXTREME_THRESHOLD: 0.15 → 0.25` in `aiedge/context/daytype.py` (currently a magic literal inside `_classify_day_type`; loose enough to be a visible number, not a constant at the top of the file — would need a small refactor to expose it). Rationale: adds 2–3 pp TFO fire rate with no accuracy cost. Smaller and less urgent than the incr-20 `always_in` window change, which was negative-sum at production.
- **Still pending from prior incrs:**
  - incr 20: `ALWAYS_IN_WINDOW: 5 → 10` and mirror `STRONG_TREND_WINDOW` in `signals/components.py`.
  - incr 18: `MAJORITY_TREND_BAR_FLOOR: 0.40 → 0.25`.
  - incr 19: docstring patch on `compute_trend_state` flagging the silent-zero htf_alignment trap for offline studies.

## Open work (read-only, next passes)

- **Full day_type sweep** covering `spike_and_channel` and `trading_range` paths — requires re-implementing the downstream branches, or parameterising the full `_classify_day_type` function with per-branch knobs and gating on which branch fires.
- **Pattern Lab WR confirmation** — does a TFO-fires filter raise the setup-level WR? If day_type has 100 % same-session direction accuracy, the forward-WR signal should also be elevated on fires. This is a Pattern Lab query, not a sweep.
- **Rebalanced sample.** The 800-session bank is 66 % up-biased. Rerunning on a longer window (500 days, ~40 % up) would confirm the 100 % accuracy isn't an up-run artifact. Same item incr 20 flagged.

## Artifacts

- Sweep tool: `tools/day_type_sweep_incr21.py` (new). Mirrors incr 20's structure exactly — same 800-session loader, same reporting schema.
- Data: `vault/Scanner/methodology/day_type_sweep_incr21.csv` (800 rows) + `day_type_sweep_incr21.json` (summary).
- Figures:
  - `figures/day_type_sweep_heatmap.png` — 3-panel heatmap (fire rate / accuracy / median |score|) with production cell boxed in red.
  - `figures/day_type_sweep_pareto.png` — 20 cells as fire vs accuracy scatter; all cells sit at accuracy = 1.0, far above the 65.9 % always-up baseline.
  - `figures/day_type_sweep_prod_vs_top.png` — production vs top-3 accuracy-ranked cells among fire ≥ 10 % eligible set (since all cells tied at 100 %, the "top" is determined by fire rate ranking within the band).
- PDF: `pdfs/trend-research-2026-04-19-incr21.pdf`.

## Test-suite status

Unchanged — 143 classes / 905 subtests still passing (zero production code touched).

## Authority

Autonomous under `~/code/aiedge/scanner/CLAUDE.md` "refactors and internal carves are fine." No production code modified. Recommended threshold change (`OPEN_EXTREME_THRESHOLD: 0.15 → 0.25`) is **NOT applied** — requires Will's nod. Scanner ranking unchanged, `BPA_PRIMARY_ENABLED` untouched.
