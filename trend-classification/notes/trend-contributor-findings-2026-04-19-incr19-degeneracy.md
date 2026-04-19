---
title: Trend research — incr 19 · contributor degeneracy + directional-accuracy hierarchy on 800 real RTH sessions
date: 2026-04-19
slice: silent-fail audit across all 12 contributors on the same 800-session real RTH equity sample used in incr 18; surfaces a second silent-fail contributor and a sharp directional-accuracy hierarchy across the bank
---

# Increment 19 — contributor degeneracy + directional-accuracy hierarchy

## What this increment is

Incr 17 caught `majority_trend_bars` as constant 0 on real ES.c.0 overnight bars. Incr 18 confirmed the failure persists on 800 real RTH equity sessions and recommended lowering the majority floor to 0.25 (still pending Will's nod).

This increment asks the natural follow-up: **are other contributors silently failing the same way**, and how much directional information does each one actually carry per session? Run on the same 800-session real RTH equity sample (387 symbols, 9 trading dates, 2026-04-09 → 2026-04-19) so results are directly comparable to incr 18.

For each of the 12 contributors, measure:
1. **Fire rate** — fraction of sessions where `|score| > 1e-9`.
2. **Unique-value count** — distinct outputs across the sample (resolution).
3. **Output entropy** (Shannon, 8 bins on [-1, +1]) — informativeness; max = 3.0 bits.
4. **Saturation** — fraction of fires that land at the extremes (`|score| ≥ 0.9`).
5. **Directional accuracy** — when fired, does sign match realised open → close session move?
6. **Disagreement with majority** — sign opposed to equal-weighted mean of the other 11.

Read-only with respect to the scanner. No production code modified, no flags flipped, no thresholds touched.

## TL;DR

- **Two silent-fail contributors** out of 12: `htf_alignment` (0/800 fires — dormant by design when daily/weekly close history isn't passed in) and `majority_trend_bars` (1/800 fires — confirms incr 18).
- **A sharp directional-accuracy hierarchy** emerges. When the contributors fire on a session, accuracy ranges from **57.6% (always_in)** to **97.9% (trending_everything)** vs the realised open → close direction. Equal weighting silently averages a 98% predictor with a 58% predictor.
- **`session_shape` is the workhorse** — fires on **100%** of sessions, **644 unique** outputs, entropy **2.37 bits / 3.0 max**, **96.3%** directional accuracy. Most informative single contributor in the bank.
- **`trending_everything` is the conviction voter** — only fires on 49.6% of sessions, but **88.7% saturation** (when it fires it goes hard) and **97.9%** directional accuracy. The closest thing to a binary high-confidence signal.
- **`always_in` is surprisingly weak** — 36% fires, **57.6%** directional accuracy. Barely better than coin flip when it fires. Worth investigating whether the 5-bar window is too short on RTH bars.
- **`cycle_phase` fires every session** (99.8%) with the second-highest unique-value count (443) but only **65.6%** directional accuracy. Anchors structure, not direction.

## The headline figure — fire rate, ranked

![Fire rate ranking](figures/contributor_degeneracy_fire_rate.png)

Two contributors sit below the 5% silent-fail line: `htf_alignment` (0/800) and `majority_trend_bars` (1/800). Eight contributors fire on at least 36% of sessions. Three (`cycle_phase`, `bpa_trend_bar_density`, `session_shape`) fire on > 90% of sessions.

## The second headline figure — output entropy, ranked

![Entropy ranking](figures/contributor_degeneracy_entropy.png)

Informativeness in bits — how much of the [-1, +1] output space does each contributor actually use? `session_shape` (2.37), `spike_quality` (2.43), and `small_pullback_trend` (2.01) lead the pack. `htf_alignment` (-0.00, single value) and `majority_trend_bars` (0.01) are degenerate.

## The third headline figure — per-contributor distributions

![Distributions](figures/contributor_degeneracy_distribution.png)

Twelve panels (log y-axis), one per contributor. Spike-at-zero = the contributor is silent on most sessions. Wide distributions = the contributor uses its output range. The asymmetric `trending_everything` distribution (mass at +0.5 / +1.0 vs zero) is the saturation behaviour quantified in the table below.

## Per-contributor diagnostic table (sorted by fire rate)

| contributor | fire | uniq | ent (bits) | sat | dir acc | disagree |
|:---|---:|---:|---:|---:|---:|---:|
| `htf_alignment` | **0.0%** | 1 | -0.00 | n/a | n/a | n/a |
| `majority_trend_bars` | **0.1%** | 2 | 0.01 | 0% | 0.0% | 100.0% |
| `trending_swings` | 12.6% | 5 | 0.74 | 14% | 75.0% | 28.7% |
| `spike_duration` | 13.5% | 5 | 0.82 | 71% | **85.7%** | 18.5% |
| `day_type` | 19.6% | 84 | 0.97 | 4% | 75.2% | 26.8% |
| `always_in` | 36.0% | 8 | 1.56 | 0% | **57.6%** | 37.8% |
| `trending_everything` | 49.6% | 5 | 1.66 | **89%** | **97.9%** | 15.9% |
| `small_pullback_trend` | 68.0% | 123 | 2.01 | 0% | 68.8% | 27.2% |
| `spike_quality` | 92.1% | 32 | 2.43 | 3% | 64.1% | 43.7% |
| `bpa_trend_bar_density` | 93.8% | 117 | 1.04 | 0% | 81.2% | 19.2% |
| `cycle_phase` | 99.8% | 443 | 1.68 | 0% | 65.6% | 28.9% |
| `session_shape` | **100.0%** | 644 | 2.37 | 0% | **96.3%** | 18.5% |

**Key:** `fire` = % of 800 sessions where signed score ≠ 0 · `uniq` = distinct rounded values · `ent` = Shannon entropy (8 bins, max 3.0 bits) · `sat` = fraction of fires at |score| ≥ 0.9 · `dir acc` = sign agreement with realised open → close move (flats excluded) · `disagree` = sign opposed to equal-weighted mean of the OTHER 11 contributors.

## Findings

### 1. Second silent-fail contributor — `htf_alignment` is fully dormant in the offline study

`htf_alignment` is opt-in: it scores 0.0 unless `daily_closes` and `weekly_closes` are passed to `compute_trend_state`. The cached parquet store has no daily/weekly history attached, so this study calls it with `None` — by-design 0/800 fires.

The live runner DOES wire daily/weekly closes via `_annotate_trend_state` (incr 17 task #2), so production isn't fully silent on this contributor. But the offline test path used here, the Pattern Lab backfill path, and any future research that loads from cache without ALSO loading the daily_closes_cache **will silently get a zero contribution from htf_alignment**. That's a hidden trap for any future contributor study.

**Action:** documentation patch — note in `compute_trend_state`'s docstring that callers passing `None` for HTF closes will get a zero contribution from `htf_alignment` and that this is silent. The current docstring describes the behaviour but doesn't flag it as a study trap. (Zero-prod-change. Will's nod not required for a docstring change but I am flagging it here rather than acting unilaterally per the standing direction.)

### 2. `majority_trend_bars` confirmed broken on the same sample

Independent reproduction of incr 18 from a fresh code path: 1/800 fires (0.12%). The `disagree_vs_others = 100%` column is the single fire being directionally wrong (the `dir_acc = 0%` cell). Incr 18's recommendation to lower the floor to 0.25 stands on this evidence too.

### 3. Sharp directional-accuracy hierarchy — equal weighting is mixing tiers

When the 12 contributors fire on a session, the sign-matches-realised-direction rate spans from 57.6% (`always_in`) to 97.9% (`trending_everything`). Equal weighting averages all of them with the same vote. **Tier breakdown:**

| tier | dir acc | contributors |
|:---:|:---:|:---|
| A | ≥ 90% | `trending_everything` (97.9%), `session_shape` (96.3%) |
| B | 75-90% | `spike_duration` (85.7%), `bpa_trend_bar_density` (81.2%), `day_type` (75.2%), `trending_swings` (75.0%) |
| C | 60-75% | `small_pullback_trend` (68.8%), `cycle_phase` (65.6%), `spike_quality` (64.1%) |
| D | 50-60% | `always_in` (57.6%) |
| F (silent) | n/a | `majority_trend_bars`, `htf_alignment` |

**Important caveat — this is labelling fidelity, not forward predictive value.** "Directional accuracy" here is the contributor's sign vs the realised same-session open → close. A contributor that fires LATE in a session has the easier task of agreeing with the move that already happened. A pure forward-prediction study (incr 17's drop_zero / downweight study used 3/6/12-bar forward horizons and showed equal weighting holds up there) is a different question.

This tier list is a snapshot of how much each contributor's vote reflects what the session did, not how much each contributor predicts the next 30 minutes.

### 4. `trending_everything` is a binary high-conviction signal

89% of its fires saturate at |score| ≥ 0.9, and those fires are 97.9% directionally accurate. It only votes when 4 of 4 (or 0 of 4) regression slopes agree across closes / highs / lows / body-mids — by construction, it's a "the session has trended on every measurable axis" detector. The +1 / -1 / 0 trichotomy makes it more like a categorical vote than a graded score.

### 5. `always_in` is the weakest informative contributor

36% fires, 57.6% accuracy when fired — barely above coin-flip on the realised same-session move (which is itself ~66% up-biased on this 9-day window, so 57.6% accuracy actually UNDER-PERFORMS the always-predict-up baseline of 65.95% from incr 18). The 5-bar consecutive-strong-body window appears to overfit to short bursts that don't translate into the rest of the session.

**Worth a follow-up incr-18-style sweep:** vary `ALWAYS_IN_WINDOW` (current: 5) and `DIRECTION_MIN_CONSEC` (current: 2) to see if a longer window or stricter consecutive-bar requirement raises directional accuracy without collapsing fire rate.

### 6. `cycle_phase` is a structure detector, not a direction predictor

99.8% fires, 443 unique values (highest after `session_shape`), but only 65.6% directional accuracy. Cycle phase is the source of `TrendState.structure` (the `bull_spike` / `bear_channel` / etc. label), so its job is structural classification. The contributor is doing the right job; the wrapper just shouldn't be expected to carry equal directional weight as `session_shape` or `trending_everything`.

## Cross-reference — disagreement-vs-majority

The `disagree_vs_others` column reveals a separate story: which contributors most often vote AGAINST the equal-weighted majority of the other 11.

- `spike_quality` (43.7% disagreement) and `always_in` (37.8% disagreement) are the most heterodox voters. Both fire often AND vote against the majority on a high fraction of those fires. Combined with their lower directional accuracy (64.1% / 57.6%), this is consistent with both being noisy on a same-session-direction read.
- `trending_everything` (15.9% disagreement) and `session_shape` (18.5%) are the most majority-aligned voters. They drive the majority more than they oppose it.

## Action items (gated on Will's nod, per scanner CLAUDE.md)

1. **DOCUMENTATION (zero blast radius):** add a one-line note to `compute_trend_state` docstring flagging that callers passing `None` for `daily_closes`/`weekly_closes` get a silent zero contribution from `htf_alignment`. This would prevent the same study trap that hit this run.
2. **PRIMARY (still pending from incr 18):** lower `MAJORITY_TREND_BAR_FLOOR` to 0.25. Re-baseline replay-equivalence tests. Re-run this same degeneracy study to confirm `majority_trend_bars` exits the silent-fail zone.
3. **NEXT STUDY:** incr-18-style two-axis sweep on `always_in` (`ALWAYS_IN_WINDOW`, `DIRECTION_MIN_CONSEC`) to see if longer / stricter requirements raise directional accuracy. The contributor is structurally similar to `majority_trend_bars` (consecutive-strong-body counting) so the same recalibration logic applies.
4. **NEXT STUDY:** incr-18-style sweep on the `_DAY_TYPE_MAGNITUDE` table or the strict `trend_pct > 0.50` gate — `day_type` only fires on 19.6% of sessions despite covering a structural read every session has by definition.
5. **OPEN QUESTION on weighting:** the directional-accuracy tier list is a same-session label-agreement signal, not forward predictive value. Pattern Lab WR-driven weighting (still blocked on DB backfill) remains the right path to actually update equal weighting in production. Don't act on this tier list as a weighting recommendation — but do use it to prioritise which contributors deserve the next deep-dive sweep.

## Mistakes avoided / lessons baked in

1. **Asked a single broad question on the same sample.** Rather than bespoke recalibration sweeps per contributor, this run runs ONE diagnostic across the whole bank. That surfaces the comparative picture (where does each contributor sit on fire rate vs entropy vs accuracy) which a per-contributor sweep would miss.
2. **Same sample, same loader as incr 18.** Direct comparability — the `majority_trend_bars` row in this table is the SAME 1/800 fire rate that the incr-18 floor sweep produced from a different code path. That's a useful sanity check on the loader.
3. **Decomposed accuracy excluded flats.** Same convention as incr 18 — 51 of 800 sessions had `|move_pct| < 0.001` and were excluded from the directional accuracy denominator, preserving the asymmetric base-rate caveat (66% up-biased on this 9-day window).
4. **`htf_alignment` reported honestly as zero, not masked.** A "smarter" study could have loaded daily/weekly history and given htf_alignment a real signal — but then this study would mask the silent-fail trap that ANY offline study hitting `compute_trend_state(df, daily_closes=None)` would face. Reporting the zero is the more useful research output.
5. **Sample bias caveat carries over.** Same 9-trading-day window as incr 18, same 387 US equity universe. Multi-month / multi-asset extension is the natural next step.

## Where things live

- **New tool:** `~/code/aiedge/scanner/tools/contributor_degeneracy_incr19.py`
- **New figures (3):**
  - `figures/contributor_degeneracy_fire_rate.png`
  - `figures/contributor_degeneracy_entropy.png`
  - `figures/contributor_degeneracy_distribution.png`
- **New data artifacts:**
  - `contributor_degeneracy_incr19.csv` (per-session score for all 12 contributors + realised direction)
  - `contributor_degeneracy_summary_incr19.csv` (per-contributor diagnostic stats)
  - `contributor_degeneracy_incr19.json` (machine-readable summary)
- **This note:** `trend-contributor-findings-2026-04-19-incr19-degeneracy.md`
- **Read first:**
  - `trend-contributor-findings-2026-04-19-incr18-majority-floor.md`
  - `trend-contributor-findings-2026-04-19-incr17-followups.md`
  - `trend-contributor-findings-2026-04-19-incr15-capstone.md`

## Authority

Read-only research under scanner CLAUDE.md "refactors and internal carves are fine." Pattern Lab and replay-equivalence tests are byte-stable. The recommended docstring patch and the threshold/window changes flagged above are **not** applied — those require Will's explicit nod per the scanner's standing direction on detector thresholds.
