# Trend classification inventory — real-time code path

Date: 2026-04-18
Goal: list every place the live scanner answers "is this a trend, and if so, which direction?" Unify later; inventory first.

**2026-04-18 follow-up**: spec for the unification target lives at `trend-state-canonical-spec.md`. Causality audit (no look-ahead bias) done against all 13 classifiers — all pass scoring-time causality; swing-based contributors have a 1-bar confirmation lag by construction, not bias.

**2026-04-19 wiring status (incr 14)**: 11 of 13 contributors wired into the canonical `aiedge/context/trend.py`: always_in, cycle_phase, trending_swings, majority_trend_bars, trending_everything, small_pullback_trend, spike_quality, spike_duration, bpa_trend_bar_density, day_type, session_shape. Remaining: htf_alignment (next, by-contract causal), regime family (atr_percentile, realized_vol_tercile — amplifiers, not direction voters; documented as not contributors). Headline structural-pair finding visible in `figures/structural_pair.png`.

**2026-04-19 wiring status (incr 15 — INVENTORY-COMPLETE)**: 12 of 12 **direction-voting** contributors wired. `htf_alignment` landed as the first regime-scale contributor (daily/weekly EMA cross, opt-in via `daily_closes` / `weekly_closes` kwargs; causality is by-contract, not chaos-tail). The regime family (`atr_percentile`, `realized_vol_tercile`) is **formally excluded** from the contributor schema — they are stratifiers/amplifiers, not direction voters, and their signed-score collapse would conflate "volatile" with "trending". Headline confluence finding visible in `figures/htf_confluence.png`. Capstone note: `trend-contributor-findings-2026-04-19-incr15-capstone.md`. Test suite: 140 classes / 845 subtests passing. **Still zero consumers** — dashboard-payload emission still needs Will's nod.

## Summary

The live scanner pipeline (`aiedge/signals/pipeline.py::score_gap_day`) computes **at least 11 distinct "trend-ish" values** per bar close, from 8 different modules. Each has its own lookback, its own bar-classification rule, and its own output semantics. No single canonical "trend state" exists — downstream code picks which classifier to trust per use-case.

This note is the raw inventory. A follow-up can propose a unified schema.

## Inventory

| # | Name | File | Function | Lookback | Output | Used by |
|---|------|------|----------|----------|--------|---------|
| 1 | `always_in` | `aiedge/signals/components.py:1251` | `_score_uncertainty` | last 5 bars (`STRONG_TREND_WINDOW`) | `"long" \| "short" \| "unclear"` | BPA gate, pipeline flip detection, live card |
| 2 | `day_type` | `aiedge/context/daytype.py:83` | `_classify_day_type` | full session, warmup 7 bars | one of {`trend_from_open`, `spike_and_channel`, `trending_tr`, `trading_range`, `tight_tr`, `undetermined`} + confidence + warning | day-type weight matrix, BPA gate, live card |
| 3 | `cycle_phase` | `aiedge/context/phase.py:219` | `classify_cycle_phase` | last 15 bars | softmax over {`bull_spike`, `bear_spike`, `bull_channel`, `bear_channel`, `trading_range`} | context only (not in signal math today) |
| 4 | `session_shape` | `aiedge/context/shape.py:274` | `classify_session_shape` | full session, direction-aware, 60-min live warmup | softmax over {`trend_from_open`, `spike_and_channel`, `trend_reversal`, `trend_resumption`, `opening_reversal`, `undetermined`} | live card (late session) |
| 5 | `htf_alignment` | `aiedge/context/htf.py:53` | `classify_htf_alignment` | daily + weekly EMA cross | `{daily_bias, weekly_bias, alignment ∈ {aligned, mixed, opposed, no_data}}` | aggregator (Phase 6 wiring) |
| 6 | `atr_percentile` / `realized_vol_tercile` | `aiedge/features/regime.py` | both | last 20 days | `float [0,1]` / `"low" \| "mid" \| "high"` | amplifier for trend bias in quiet/loud regimes; priors stratification |
| 7 | `trending_swings` | `aiedge/signals/components.py:983` | `_score_trending_swings` | all detected single-bar swings | raw score in {-1, 0, +1, +2} | urgency component |
| 8 | `majority_trend_bars` | `aiedge/signals/components.py:536` | `_score_majority_trend_bars` | all bars | raw score in {-1, 0, +1, +2} | urgency component |
| 9 | `trending_everything` | `aiedge/signals/components.py:606` | `_score_trending_everything` | all bars, linreg over closes/highs/lows/body-mids | raw score in {-1, 0, +1, +2} | urgency component |
| 10 | `spike_quality` + `spike_bars` | `aiedge/signals/components.py:160` | `_score_spike_quality` | from bar 0 forward until non-trend bar | spike bar count + 0-4 quality score | urgency + downstream (pullback, duration, BPA spike_channel) |
| 11 | `spike_duration` | `aiedge/signals/components.py:1043` | `_score_spike_duration` | from bar 0 forward | raw score in {-0.5, 0, +1, +2}; counts bars sustained with <25% single-bar retrace | urgency component |
| 12 | `small_pullback_trend` (SPT) | `aiedge/signals/components.py:715` | `_score_small_pullback_trend` | last 15 bars (`SPT_LOOKBACK_BARS`) | raw score 0-3 via weighted sum of 5 sub-checks | urgency component; ranking-sensitive |
| 13 | BPA per-bar `is_*_trend_bar` | `shared/bpa_detector.py:302`, `315` | `_is_bull_trend_bar`, `_is_bear_trend_bar` | single bar | bool | BPA detector internal (H1/L1, spike_channel, FL/FH structural checks) |

## Per-bar "trend bar" definitions used across modules

The definition of a single "trend bar" varies by module:

| Module | File:line | Rule | Shared constant? |
|--------|-----------|------|------------------|
| `always_in` | `components.py:1258` | `_is_bull(row) and _body_ratio(row) > STRONG_BODY_RATIO` (0.60) | yes (`daytype.STRONG_BODY_RATIO`) |
| `day_type` warmup gate | `daytype.py:99` | same rule as above | yes |
| `majority_trend_bars` | `components.py:551` | `_is_bull(row) and _body_ratio(row) > 0.50` — **hard-coded 0.50, not `STRONG_BODY_RATIO`** | no |
| SPT trend bar | `components.py:755` | signed body ≠ 0 AND `body_ratio >= SPT_TREND_BODY_RATIO` (0.40) | no |
| `spike_quality` is_trend_bar | `components.py:165` | `_is_bull` or `_is_bear` — no body-ratio gate at all | no |
| BPA trend bar | `bpa_detector.py:302-315` | own canonical Brooks definition (body dominates, close near extreme, not shared) | no |
| `cycle_phase` bull_channel | `phase.py:127` | peak fit at body_ratio = 0.45 | no |

**Finding**: the scanner has at least **six** different body-ratio thresholds for "trend bar" — 0.40, 0.45, 0.50, 0.60, plus two unconstrained (`is_bull`/`is_bear`) and the BPA canonical. They were tuned independently.

## Output-scale summary

| Output | Scale | Direction? |
|--------|-------|------------|
| `always_in` | categorical | yes (long/short/unclear) |
| `day_type` | categorical | implicit via `gap_direction` input |
| `cycle_phase` | softmax distribution | yes (bull/bear explicit in label) |
| `session_shape` | softmax distribution | direction given as input |
| `htf_alignment` | categorical | alignment-based |
| trending_swings / majority / everything / spike_duration / spt | integer/real score | direction given as input |
| BPA trend-bar | bool per bar | yes (bull vs bear) |

No classifier produces a normalized **`trend_strength ∈ [0, 1]`** scalar. Urgency-subsystem values are raw deltas that sum to `URGENCY_RAW_MAX = 29.0` before normalization.

No classifier produces a **`trend_age`** (how many bars the current trend has lasted). `spike_duration` measures the opening spike only, not a trend that started mid-session.

No classifier emits an explicit **`trend_break`** event. A flip is implicit: `always_in` changes value bar-to-bar and the pipeline reads that directly (`pipeline.py:267,273`).

## Known overlaps and conflicts

1. **`day_type == trend_from_open`** requires `open_position < 0.15` (bull) and `trend_pct > 0.50`. **`cycle_phase == bull_channel`** requires `drift >= 50% of range` and higher-closes fraction. These can disagree on the same bar because day_type uses full-session structure and cycle_phase uses last 15 bars only — no cross-check.
2. **`session_shape`** duplicates four of six `day_type` labels (`trend_from_open`, `spike_and_channel`, plus `trend_reversal` / `opening_reversal` / `trend_resumption` that day_type lacks). They're computed independently with different math.
3. **`always_in == "short"` on a `day_type == trend_from_open` (gap-up) bar** is the "bear flip on a bull day" case. The pipeline detects it (`pipeline.py:264-269`) and overrides R/R direction, but neither classifier is updated — the flip is only visible in the combined reading.
4. **SPT vs `trending_swings`** both claim to measure "structure intact, shallow pullbacks", but SPT uses a 15-bar window and 5-sub-check weighted sum, while `trending_swings` just counts consecutive HH/HL on detected single-bar swings. They disagree regularly per backfill samples (not measured here — would need to correlate).

## Gaps worth closing

- **Canonical trend state.** A single dict: `{direction: up/down/none, strength: 0-1, age_bars: N, structure: spike/channel/tr/reversal, confidence: 0-1}`. All existing classifiers become contributors to that state, not parallel outputs.
- **Unified "trend bar" definition.** Pick one body-ratio cutoff or make it an explicit tunable per caller with a documented default.
- **Trend-age tracker.** Count bars since last `always_in` flip; expose as a feature. Surfaces the "late channel" vs "early spike" distinction Brooks leans on.
- **Trend-break event.** Emit when `always_in` flips OR when swing-structure breaks (HH/HL chain broken) OR when price breaks a `cycle_phase` channel line. Today only the first is observable.
- **Cross-classifier agreement score.** A single number answering "how many of the 11 signals agree on direction right now". Useful as a meta-confidence for Pattern Lab stratification.

## Open questions for a unification pass

1. Should `always_in` subsume `majority_trend_bars` / `trending_swings` / `trending_everything`, or are those three "how much" signals that complement a "which direction" signal?
2. Is `cycle_phase` actually load-bearing anywhere? Grep shows no consumer in signal math. If it's only on the live card, consider collapsing it into `day_type` (they share four of five labels).
3. SPT was added as an urgency component *specifically* to catch calm-drift low-ADR tech days the other signals miss (`components.py:99-104`). If we unify, SPT's role is probably "trend quality on low-vol tapes" — a regime-stratified refinement, not a replacement.

## References

- Pipeline orchestration: `aiedge/signals/pipeline.py:140-300`
- BPA integration: `aiedge/signals/bpa.py:60-130`
- Shared body-ratio constant: `aiedge/context/daytype.py:36` (`STRONG_BODY_RATIO = 0.60`)
- Brooks source-of-truth for patterns: `vault/Brooks PA/patterns/*.md`
