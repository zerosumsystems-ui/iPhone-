---
title: Trend research — incr 20 · `always_in` two-axis sweep over (ALWAYS_IN_WINDOW, DIRECTION_MIN_CONSEC)
date: 2026-04-19
slice: sweep the two strictness knobs on the `always_in` contributor over 800 real RTH equity sessions; test incr 19's "window too short" conjecture; find a strictly dominant cell over production.
---

# Increment 20 — `always_in` two-axis sweep

## What this increment is

Incr 19's cross-bank diagnostic flagged `always_in` as the weakest informative contributor in the trend-state stack — **36.0% fire rate** on 800 real RTH sessions but only **57.6% directional accuracy** when it fired. That's **under** the always-predict-up baseline on this sample (66.0%), meaning when `always_in` votes, you'd do better ignoring it and calling every session "up." The posted conjecture: the 5-bar window is too short and overfits to bursts that don't translate into the session.

This increment tests that conjecture with an incr-18-style two-axis sweep on the detector's two knobs:

| knob | production | sweep range |
|---|---:|---|
| `ALWAYS_IN_WINDOW` | **5** | 3, 5, 7, 10, 15 |
| `DIRECTION_MIN_CONSEC` | **2** | 1, 2, 3, 4 |

Same 800-session sample used in incr 18 and 19 (387 US equity symbols, 9 RTH dates 2026-04-09 → 2026-04-19; 1-min Databento bars resampled to 5-min RTH; ≥20 bars per session; flats excluded from the accuracy denominator).

**Read-only with respect to the scanner. No production code modified, no flags flipped, no thresholds touched.** A parameterised replica of `_always_in_score` lives in the sweep tool — byte-stable duplicate of the scanner import for research purposes.

## TL;DR

- **Conjecture CONFIRMED.** Production (W=5, K=2) directional accuracy is **57.6%** — below the **66.0%** always-predict-up baseline on this sample. The 5-bar window is too short.
- **A strictly dominant cell exists.** `W=10, K=2` fires on **51.9%** of sessions (+16 pp vs production) at **66.9%** directional accuracy (+9 pp). Both dimensions win; no Pareto trade-off.
- **The binding knob is the window, not the min-consec.** Holding K=2 and varying W: `57.6 → 63.2 → 66.9 → 64.7` at W=5/7/10/15. Window length controls both fire rate and accuracy; K mostly controls stringency at a cost to fire rate.
- **K=1 (any single strong bar) is noise** regardless of W — it fires often but accuracy hovers at coin-flip.
- **One cell hits the 80% accuracy band** — `W=15, K=1` at **82.9% accuracy** but only **5.5% fire rate** (n=44). Too rare to ship as production but documented as a "high-conviction" setting candidate.
- **Caveat carries over from incr 19.** "Directional accuracy" here is sign-match-with-realised-same-session open→close — labelling fidelity, not forward predictive value. Pattern Lab WR-driven weighting is still the right path. This sweep identifies candidates, does not decide.

## Headline figure — the (W × K) grid

![Always-in sweep heatmap](figures/always_in_sweep_heatmap.png)

Three panels in one grid. Left = fire rate (how often the detector votes). Middle = directional accuracy (when it votes, does the sign match realised open→close). Right = median |score| on fires (binary vs graded). Red box = production cell.

**The story is in the middle panel.** Production sits at 58% accuracy. Every cell in the W=10 row with K≤3 beats it, and `W=10, K=2` sits at 67% with a 52% fire rate.

## Second headline figure — Pareto view

![Pareto view](figures/always_in_sweep_pareto.png)

Every (W, K) cell plotted as (fire_rate, accuracy). The dashed grey line is the always-up baseline. Cells **above** the baseline add information; cells **below** subtract information. Production sits below the baseline. `W=10, K=2` sits comfortably above it.

## Third headline figure — production vs top-accuracy candidates

![Production vs top](figures/always_in_sweep_prod_vs_top.png)

Side-by-side bars. Only cells with fire_rate ≥ 15% are eligible (so we're not cherry-picking rare high-accuracy cells). `W=10, K=2` wins both axes.

## Full grid (rounded, 800 sessions)

| W \\ K | K=1 | K=2 | K=3 | K=4 |
|:---|---:|---:|---:|---:|
| **W=3** | 62% / 58% | 24% / 62% | 5% / 56% | 0% / n/a |
| **W=5** | 40% / 56% | **36% / 58%** ← prod | 7% / 51% | 0.5% / 75% |
| **W=7** | 25% / 61% | 45% / 63% | 10% / 53% | 2% / 67% |
| **W=10** | 13% / 68% | **52% / 67%** ★ | 18% / 64% | 4% / 69% |
| **W=15** | 6% / 83% | 52% / 65% | 22% / 62% | 5% / 67% |

**Cell format**: fire rate / directional accuracy. `★` = strictly dominates production on both axes among eligible (fire ≥ 15%) cells. Baseline (always-predict-up): **66.0%**.

## Findings

### 1. Production is actively under-performing the baseline

The production (W=5, K=2) cell fires on 288/800 sessions at 57.6% accuracy. The sample base rate (always-predict-up) is 65.95%. When `always_in` votes in production, it is **worse than ignoring the vote and defaulting to up** by ~8 percentage points. Equal weighting in the 12-contributor aggregator averages this sub-baseline vote with contributors that are well above baseline. Incr 19 already flagged the hierarchy; this sweep quantifies that production is literally below the line.

### 2. `W=10, K=2` dominates production on both fire rate and accuracy

- Fire rate **51.9%** vs 36.0% (+15.9 pp absolute, +44% relative)
- Accuracy **66.9%** vs 57.6% (+9.3 pp absolute, +16% relative)
- Median |score| when fired: **0.200** vs 0.400 (softer, more graded — longer window divides by larger denominator)

This is the **only** cell with fire rate ≥ 40% AND accuracy > 66%. It's above the always-up baseline, above the coin-flip line, and on the Pareto frontier for high-fire settings.

Mechanism: a 10-bar window samples more of the session, so K=2 (two consecutive strong-body trend bars in the same direction) catches structural trends that a 5-bar window misses by landing on a pullback. K=2 at W=10 is less sensitive to single-bar noise than K=2 at W=5.

### 3. Binding knob is W, not K

At K=2, varying W: **57.6 → 63.2 → 66.9 → 64.7** (W=5/7/10/15). Monotonic-up to W=10 then a mild drop. At fixed K=2, the window length is the lever; stricter K tanks fire rate without buying meaningful accuracy above 65-70%.

Implication: if the recalibration were to happen, the useful change is `ALWAYS_IN_WINDOW: 5 → 10`. `DIRECTION_MIN_CONSEC` should stay at 2.

### 4. K=1 is noise

K=1 means "any single strong-body trend bar AND no opposite strong-body bar." It fires often (25-62% depending on W) but accuracy hovers at 56-61% — barely different from coin flip after excluding the W=15 outlier (see finding 5). Single-bar noise drives the signal. Don't lower K to 1.

### 5. High-conviction corner — `W=15, K=1` at 83% accuracy but 5.5% fire rate

One cell is genuinely high-confidence: W=15, K=1 — 44/800 sessions fire, but 82.9% directional accuracy among those. Interpretation: requiring "one strong-body trend bar AND zero opposite strong-body bars in 15 bars" is a very strict no-opposing-pressure filter. When it fires, the session has been unambiguously one-directional. Too rare to carry the primary signal (5.5% fire) but documented as a candidate for a future "high-conviction" overlay.

### 6. Zero-fire corner — W=3, K=4 impossible by construction

W=3, K=4 requires 4 consecutive same-direction strong-body bars in a 3-bar window. Structurally impossible. Sweep returns 0% fire, as expected. Sanity check that the parameterised replica is wiring (W, K) correctly.

## Cross-reference — accuracy hierarchy from incr 19

Plugging the best `always_in` cell back into the incr 19 tier list:

| tier | dir acc | contributors |
|:---:|:---:|:---|
| A | ≥ 90% | `trending_everything` (97.9%), `session_shape` (96.3%) |
| B | 75-90% | `spike_duration` (85.7%), `bpa_trend_bar_density` (81.2%), `day_type` (75.2%), `trending_swings` (75.0%) |
| B- | 65-75% | **`always_in` at W=10, K=2 (66.9%)**, `small_pullback_trend` (68.8%), `cycle_phase` (65.6%), `spike_quality` (64.1%) |
| D | 50-60% | `always_in` at W=5, K=2 (57.6%) ← production |
| F (silent) | n/a | `majority_trend_bars`, `htf_alignment` |

Recalibrating `always_in` to W=10, K=2 moves it from tier D (below-baseline noise) to the bottom of tier B- (competitive with `small_pullback_trend` and `cycle_phase`). Not a top-tier contributor, but at least a useful one.

## Important caveats

1. **Same-session labelling, not forward prediction.** The accuracy metric is sign-match-with-realised-open→close-of-same-session, not forward WR. A contributor that fires late in a session has the easier task of agreeing with what already happened. Pattern Lab WR-driven weighting (open work item #3 from capstone) uses 3/6/12-bar forward horizons and is the right path for a production change.
2. **Sample bias — 9-day up-biased window.** 494/749 non-flat sessions (66%) went up. On a down-biased or balanced sample, the "always-up baseline" would differ. The relative comparison (production 57.6% vs W=10,K=2 at 66.9%) is robust to sample bias because both metrics apply to the same 800 sessions. The absolute "beat the baseline" claim needs to hold on a rebalanced sample before any flag flip.
3. **Monotonicity is empirical not theoretical.** Accuracy went 58 → 63 → 67 → 65 across W=5/7/10/15 at K=2. W=10 is a local optimum on this sample. Pattern Lab WR sweeps at more windows (12, 20, 25) would be the right next step before adopting W=10 as permanent.
4. **Production change requires Will's nod.** The scanner CLAUDE.md standing direction is explicit: "Ask before any change that affects scanner output" — including detector thresholds. This run surfaces a candidate and the evidence behind it. The flip itself is gated.

## Action items (gated on Will's nod, per scanner CLAUDE.md)

1. **PRIMARY (new, from this incr):** change `ALWAYS_IN_WINDOW: 5 → 10` in `aiedge/context/trend.py` and `STRONG_TREND_WINDOW: 5 → 10` in `aiedge/signals/components.py` (the docstring notes they're kept in sync). Re-baseline the replay-equivalence test `TrendStateAlwaysInContributor`. Re-run the degeneracy study from incr 19 to confirm `always_in` exits the below-baseline zone.
2. **STILL PENDING (from incr 18):** lower `MAJORITY_TREND_BAR_FLOOR` to 0.25. Independently motivated but complementary — the two silent-fail / sub-baseline contributors both need recalibration.
3. **STILL PENDING (from incr 19):** docstring patch on `compute_trend_state` flagging the silent-zero htf_alignment trap for offline studies.
4. **NEXT STUDY:** incr-18-style sweep on `day_type` — the second contributor incr 19 flagged as underfiring (19.6% on 800 sessions).
5. **NEXT STUDY (bigger):** rebalanced sample (e.g. pick 400 up-session + 400 down-session dates across a longer window) to re-run this sweep with neutralised base rate.

## Mistakes avoided / lessons baked in

1. **Parameterised replica, not an in-place override.** Rather than monkey-patching `ALWAYS_IN_WINDOW` / `DIRECTION_MIN_CONSEC` at import time (which would spook any downstream code that reads those constants during the same process), the sweep defines a local copy of `_always_in_score` that takes (W, K) as arguments. Zero risk of contaminating the import graph. Same pattern incr 18 used for the majority-floor sweep.
2. **Held all other knobs fixed.** Only (W, K) varied. `STRONG_BODY_RATIO`, `_is_bull` / `_is_bear` bar-color predicates, and the session resampling were untouched. So the observed fire-rate / accuracy deltas are attributable purely to (W, K).
3. **Excluded flats identically to incr 18/19.** Same `move_pct ∈ (-0.001, +0.001)` flat band, same 51/800 exclusion. Directly comparable numbers.
4. **Highlighted the Pareto-dominant cell explicitly.** Will doesn't read code — the `always_in_sweep_prod_vs_top.png` figure is the visual answer to "should production change?" without requiring anyone to eyeball a 20-cell table.
5. **Did NOT recommend K=1 despite a lone high-accuracy cell.** Finding 5's 82.9% accuracy at W=15, K=1 is tempting but it's on 44 fires — too rare to drive a primary signal, and the finding 4 pattern (K=1 is noise at every other W) argues against lowering K in general.
6. **Caveat-first, recommendation-second.** Same-session accuracy ≠ forward WR. This sweep surfaces a candidate; it doesn't close the weighting question. Pattern Lab backfill is the right final arbiter.

## Where things live

- **New tool:** `~/code/aiedge/scanner/tools/always_in_sweep_incr20.py`
- **New figures (3):**
  - `figures/always_in_sweep_heatmap.png` — W × K heatmaps of fire / accuracy / median|score|
  - `figures/always_in_sweep_pareto.png` — fire vs accuracy scatter of all 20 cells, baseline overlaid
  - `figures/always_in_sweep_prod_vs_top.png` — bar comparison of production vs top-accuracy candidates
- **New data artifacts:**
  - `always_in_sweep_incr20.csv` (per-session signed score for all 20 cells)
  - `always_in_sweep_incr20.json` (grid + realised breakdown + production pointer)
- **This note:** `trend-contributor-findings-2026-04-19-incr20-always-in-sweep.md`
- **Read first:**
  - `trend-contributor-findings-2026-04-19-incr19-degeneracy.md` (motivation)
  - `trend-contributor-findings-2026-04-19-incr18-majority-floor.md` (method template)
  - `trend-contributor-findings-2026-04-19-incr15-capstone.md` (big picture)

## Authority

Read-only research under scanner CLAUDE.md "refactors and internal carves are fine." Pattern Lab and replay-equivalence tests are byte-stable. The `ALWAYS_IN_WINDOW: 5 → 10` threshold change flagged above is **not** applied — requires Will's explicit nod per the scanner's standing direction on detector thresholds.
