# Canonical real-time trend state — spec

Date: 2026-04-18
Companion to: `trend-classification-inventory.md` (inventory of the 13 existing classifiers)
Goal: define one schema the scanner can emit per bar-close, with explicit causality guarantees, so future work unifies against a target instead of adding a 14th classifier.

This note is spec only. **No code changes yet.** Review before wiring.

---

## The schema

```python
@dataclass(frozen=True)
class TrendState:
    direction: Literal["up", "down", "none"]   # "none" = no trend, not "unclear"
    strength:  float                           # [0.0, 1.0], normalized
    age_bars:  int                             # bars since current direction took hold; 0 if direction == "none"
    structure: Literal[                        # Brooks cycle-phase label
        "bull_spike", "bear_spike",
        "bull_channel", "bear_channel",
        "trading_range",
    ]
    confidence: float                          # [0.0, 1.0], meta-agreement across contributors
    contributors: dict[str, float]             # per-signal dir-signed score, for diagnostics
    bar_ts: pd.Timestamp                       # the close timestamp this state was computed at
    lag_bars: int                              # how many bars back the state is "confirmed" for (see Causality)
```

One state per bar close. Never mutated. `contributors` keys are the inventory IDs (`always_in`, `day_type`, `cycle_phase`, `session_shape`, `htf_alignment`, `trending_swings`, `majority_trend_bars`, `trending_everything`, `spike_quality`, `spike_duration`, `small_pullback_trend`, `bpa_trend_bar_density`) — each maps to a `[-1.0, +1.0]` directional score (negative = short bias, positive = long bias, zero = no opinion).

`strength` = magnitude of the weighted mean of contributors.
`direction` = sign of the weighted mean (zero-threshold in `tunables.py`).
`confidence` = 1 − dispersion(contributors), so high when contributors agree.

---

## Causality rules (the "no bias" part)

Every state MUST satisfy:

1. **Scoring-time causality.** At bar-close `t`, `TrendState(t)` may read bars `0..t` only. Never `t+1`, never an EOD statistic computed from bars past `t`.
2. **Swing-confirmation lag.** Single-bar swings (`features/swings.py`) require bar `i+1` to confirm bar `i`. So any contributor that depends on swings reports state for bar `t-1` at earliest; record that in `lag_bars`.
3. **No survivorship-filtered history.** Contributors that lean on rolling percentiles (`atr_percentile`, `realized_vol_tercile`) must draw from *all* prior sessions the scanner has ingested, not the subset that produced signals. Audit: `regime.py` uses raw close series → clean.
4. **No EOD leakage on intraday state.** `day_type` and `session_shape` are called with partial-session df and warmup guards (`session_minutes >= 60` for shape, `warmup 7 bars` for daytype). Their intraday output is *allowed* to change as the session evolves — but each emission at time `t` must only read bars `0..t`. Audit target: confirm both pass a replay-equivalence test (state at bar 40 from full session == state at bar 40 from df truncated to 40 bars).

Audit results (2026-04-18) for each of the 13 inventory items: **all pass rule #1** by construction (no `shift(-n)`, no `iloc[t+1]` outside bounded loops). The only natural lag is rule #2 — swing-based contributors. See `trend-classification-inventory.md` §References for line numbers.

---

## Replacements and semantics

| Schema field | Replaces / subsumes | Notes |
|---|---|---|
| `direction` | `always_in` | Rename `"unclear"` → `"none"`. A "none" trend is a real state, not missing data. |
| `strength` | the urgency sub-scores: `trending_swings`, `majority_trend_bars`, `trending_everything`, `spike_duration`, `small_pullback_trend` | These become contributors, not primary outputs. Ranking still reads their sum via `strength`. |
| `age_bars` | *new* | Counted as bars since last `direction` change. Surfaces Brooks "late channel" vs "early spike" without reintroducing spike_duration's open-only bias. |
| `structure` | `cycle_phase` argmax | `cycle_phase.probs` stays available on `contributors` for callers that want the full distribution. |
| `confidence` | *new* meta-agreement | Replaces the implicit "both classifiers said X" checks scattered through the pipeline. |

Things the schema does **not** include, deliberately:
- `day_type` / `session_shape` stay separate. They classify the *session*, not the *current trend*, and they emit post-hoc labels. A mid-session bull pullback inside a bull trend_from_open day has `direction=up` but the day-type is a different concept.
- `htf_alignment` stays a filter on setups, not a mutation of current-TF trend state.

---

## The unified "trend bar" question

Inventory flagged six body-ratio thresholds (0.40, 0.45, 0.50, 0.60, plus two unconstrained) across modules. The spec does **not** pick one. Reason: each threshold is tuned to its own use. Forcing a single cutoff would retune six scorers at once — output impact would cross the "ask before changing" line in `CLAUDE.md`.

Instead: expose them as named module-level constants with explanatory docstrings so the divergence is visible. Proposed rename map (refactor only, no value changes):

| Current location | New constant | Value |
|---|---|---|
| `components.py:551` literal `0.50` | `MAJORITY_TREND_BAR_BODY_RATIO` | 0.50 |
| `components.py:755` `SPT_TREND_BODY_RATIO` | keep | 0.40 |
| `components.py:165` (unconstrained) | `SPIKE_TREND_BAR_BODY_RATIO = 0.0` | 0.0 |
| `phase.py:127` magic `0.45` | `CYCLE_CHANNEL_BODY_PEAK` | 0.45 |
| `daytype.py:36` `STRONG_BODY_RATIO` | keep (canonical) | 0.60 |

Pure refactor — grep-and-rename. Zero output delta. Safe for an autonomous next session.

---

## Proposed first increments (non-breaking)

Pick these one at a time; each can land independently without touching ranking:

1. **Add `TrendState` dataclass** to `aiedge/context/trend.py` (new file). Emit it from a new function `compute_trend_state(df, *, session_ts)` that *calls the existing classifiers* and wraps their outputs. Don't consume it anywhere yet. Snapshot the output into the dashboard payload under a new key `trend_state` for inspection. No existing caller reads it.
2. **Add the body-ratio constant renames** listed above. Pure refactor.
3. **Add a replay-equivalence test** for `classify_day_type` and `classify_session_shape`: build a full-session df, truncate to length `k` for each `k ∈ [warmup, n]`, assert state at bar `k` from the truncated df matches state at bar `k` from the full df. This is the `rule #4` audit. If either fails, flag before any further unification.

Increments 4+ (consumers switching to `trend_state`) are a separate decision — ask Will.

---

## Status (2026-04-18 autonomous pass)

Increments 1–3 landed in `~/code/aiedge/scanner/`. Test suite 471 passed (was 451), zero regressions.

**Increment 1 — `TrendState` dataclass** (`aiedge/context/trend.py`):
- Frozen dataclass per the schema above (direction / strength / age_bars / structure / confidence / contributors / bar_ts / lag_bars).
- `compute_trend_state(df)` is wired to **only one contributor** — the always-in detector (mirrored from `signals/components.py::_score_uncertainty` lines 1257–1288). Future slices add the remaining 12 inventory items.
- **Zero consumers.** Not imported anywhere outside its own test file. Does not touch the dashboard payload yet (that step is borderline scanner-output-affecting and was deferred; pick it up next time after a quick check with Will).

**Increment 2 — body-ratio constant renames**:
- `components.py`: added `MAJORITY_TREND_BAR_BODY_RATIO = 0.50`, swapped the literal in `_score_majority_trend_bars`. Comment notes the divergence from `STRONG_BODY_RATIO`.
- `phase.py`: added `CYCLE_CHANNEL_BODY_PEAK = 0.45`, swapped the literal in both `_cycle_bull_channel_raw` and `_cycle_bear_channel_raw`.
- Skipped: `components.py:165` (the inventory's "unconstrained spike trend bar" was a `_is_bull`/`_is_bear` color check, not a body-ratio threshold — adding a `= 0.0` constant would be dead code; the actual body-ratio gate inside `_score_spike_quality` already uses `STRONG_BODY_RATIO` at line 184). `SPT_TREND_BODY_RATIO` and `STRONG_BODY_RATIO` were already named, no work needed.
- **Zero output delta** — pure rename, all 451 prior tests still pass byte-identically.

**Increment 3 — replay-equivalence tests** (`tests/context/test_causality.py`):
- 6 new TestCase classes covering `_classify_day_type`, `classify_session_shape`, and the new `compute_trend_state`.
- Strategy: build a deterministic 40-bar synthetic session, then for each truncation point `k`, compare classifier output between (a) `full_df.iloc[:k]` and (b) `extended_df.iloc[:k]` where `extended_df = full_df ++ 25 chaos bars`. If a classifier ever reads bars after `k`, A != B and the subTest fails loudly.
- Result: all three classifiers pass. Rule #4 from the spec is now mechanically enforced — any future regression that reads forward will be caught by this test.

## Status (2026-04-18 autonomous pass — increment 4)

Replay-equivalence harness extended to the remaining intraday contributors. 16 new test classes added to `tests/context/test_causality.py`:

| Contributor | File | Result |
|---|---|---|
| `classify_cycle_phase` | `context/phase.py` | causal ✓ |
| `_score_majority_trend_bars` | `signals/components.py` | causal ✓ |
| `_score_trending_everything` | `signals/components.py` | causal ✓ |
| `_score_trending_swings` | `signals/components.py` | causal ✓ (1-bar swing-confirmation lag is not bias — see spec rule #2) |
| `_score_small_pullback_trend` | `signals/components.py` | causal ✓ |
| `_score_spike_quality` | `signals/components.py` | causal ✓ (also pins spike_bars equivalence) |
| `_score_spike_duration` | `signals/components.py` | causal ✓ (chained on spike_bars from prefix) |
| `_is_bull_trend_bar` / `_is_bear_trend_bar` | `shared/bpa_detector.py` | causal ✓ (per-row predicates, structurally incapable of look-ahead — confirmed by per-bar prefix iteration) |

Test counts: 487 passed / 810 subtests (was 471 / 198). Zero regressions.

**Not covered, by design:**
- `regime.py::atr_percentile` and `realized_vol_tercile` consume prior-day history, not intraday df. The "no look-ahead" promise is a contract on the caller (today's value excluded from the history series). Inventory audit clean.
- `htf.py::classify_htf_alignment` is a daily/weekly EMA cross — same reasoning, not an intraday df classifier.

Rule #4 of this spec is now mechanically enforced for **every intraday-df contributor in the inventory** (8 of 13). Any future regression that adds a `df.iloc[t+1:]` read or a full-session statistic computed on the suffix will fail loudly in CI.

## Status (2026-04-18 autonomous pass — increment 5)

`cycle_phase` wired in as the second contributor to `compute_trend_state`. Still zero consumers — purely an internal carve (scanner output unchanged).

**Changes in `aiedge/context/trend.py`:**
- New helper `_always_in_score(df) -> (signed_score, direction, age)` extracts the first-slice logic behind a clean boundary.
- New helper `_cycle_phase_score(cp_dict) -> float` maps the softmax probs to a [-1, +1] directional score: `P(bull_spike) + P(bull_channel) − P(bear_spike) − P(bear_channel)`. Trading-range mass contributes zero. Returns 0.0 when the classifier is disabled or df is below its 3-bar warmup.
- New helper `_structure_from_cycle_phase(cp, direction, strength)` prefers `cycle_phase.top` as the authoritative `structure`, and falls back to the first-slice strength-based heuristic only when cycle_phase is disabled or returns no distribution (df shorter than `CYCLE_PHASE_LOOKBACK_BARS=15`).
- New helper `_confidence_from_contributors(scores)` computes meta-agreement = `1 − mean-absolute-deviation`. Two contributors at opposite extremes → confidence 0; two that agree → confidence 1. Single-contributor fallback preserves the first-slice `|score|` behavior.
- `compute_trend_state` now aggregates both contributors via an equal-weighted mean. `direction` is sign of the mean with a small zero-threshold (`DIRECTION_ZERO_THRESHOLD = 0.05`). `age_bars` still walks back from the tail, matching the chosen direction even when it disagrees with the always-in standalone call.

**Test additions (`tests/context/test_causality.py`):**
- New `TrendStateCyclePhaseContributor` class with 6 pins: `cycle_phase` key present, score bounded in [-1, 1], bull session → positive score, structure ∈ bull phases, confidence ≥ 0.5 when contributors agree, short-df graceful fallback.
- The existing `TrendStateReplayEquivalence` still passes — cycle_phase was already proven causal (increment 4), so combining it into `compute_trend_state` inherits that property.

**Test suite:** 493 passed / 810 subtests (was 487 / 810). Zero regressions.

**Why this was safe autonomously:** trend.py has zero consumers in the scanner — no pipeline or payload reads it. Per `~/code/aiedge/scanner/CLAUDE.md` "refactors and internal carves are fine autonomously." The dashboard-payload emission is the step that *does* affect scanner output; that still needs Will's nod.

## Status (2026-04-18 autonomous pass — increment 6)

`trending_swings` wired in as the third contributor to `compute_trend_state`. Still zero consumers — scanner output unchanged.

**Changes in `aiedge/context/trend.py`:**
- New helper `_trending_swings_score(df) -> float` maps `_score_trending_swings` to a signed score in [-1, +1]. Because the existing function is direction-conditioned (returns `{-1, 0, +1, +2}` w.r.t. an input `gap_direction`), we call it twice (up + down) and take `(max(0, up_raw) - max(0, down_raw)) / 2`. Broken-chain returns (-1) collapse to 0 on their own side so the other direction's signal still carries.
- `compute_trend_state` now aggregates three contributors: `always_in`, `cycle_phase`, `trending_swings`. Equal-weighted mean unchanged. Confidence still = 1 − MAD across contributors; with three contributors, the MAD-based metric now captures genuine three-way agreement.
- `lag_bars` is now set to 1 whenever `trending_swings` emits non-zero signal, reflecting the 1-bar swing-confirmation lag (spec rule #2). Zero when swings are silent.

**Test additions (`tests/context/test_causality.py`):**
- New fixtures `make_bull_with_pullbacks(n)` and `make_bear_with_pullbacks(n)` — strictly monotonic bull/bear sessions have no swings by construction, so dedicated zig-zag fixtures are needed to exercise the contributor. Each pullback bar has a deep lower wick (c − 2.0) producing a strict swing low; each up bar is bracketed by pullbacks producing a strict swing high. Result: clean HH/HL chain.
- New `TrendStateTrendingSwingsContributor` class with 10 pins: key present, score bounded in [-1, 1], monotonic bull returns 0 (documented behaviour), pullback fixtures classify correctly, `lag_bars == 1` when swings fire and 0 when silent, three-way agreement keeps confidence above the mid-line, bear session classifies down.
- Replay-equivalence inherited from increment 4 (`TrendingSwingsReplayEquivalence`) — the helper only wraps the already-tested function.

**Test suite:** 503 passed / 810 subtests (was 493 / 810). Zero regressions.

**Why this was safe autonomously:** trend.py still has zero consumers in the scanner — no pipeline or payload reads it. Per `~/code/aiedge/scanner/CLAUDE.md` "refactors and internal carves are fine autonomously." The dashboard-payload emission is the step that *does* affect scanner output; that still needs Will's nod.

## Status (2026-04-19 autonomous pass — increment 7)

`majority_trend_bars` wired in as the fourth contributor to `compute_trend_state`. Still zero consumers — scanner output unchanged.

**Changes in `aiedge/context/trend.py`:**
- New helper `_majority_trend_bars_score(df) -> float` maps `_score_majority_trend_bars` to a signed [-1, +1] score using the same direction-conditioned double-call trick as `_trending_swings_score`: call the underlying function twice (gap_direction="up" and "down"), clip negatives, and take `(up_pos - down_pos) / 2`. The raw function returns {-1, 0, +1, +2}; the `/2` keeps the signed result bounded in [-1, +1] after the max-clip.
- `compute_trend_state` now aggregates four contributors: `always_in`, `cycle_phase`, `trending_swings`, `majority_trend_bars`. Equal-weighted mean unchanged. Confidence = 1 − MAD across contributors; with four contributors the dispersion metric now captures agreement across two "which direction" signals (always_in, cycle_phase) and two "how much" signals (trending_swings, majority_trend_bars).
- No change to `lag_bars` — unlike `trending_swings`, majority-bar counting has no swing-confirmation lag. Bar-count statistics use bars 0..t inclusive and require no future confirmation.

**Test additions (`tests/context/test_causality.py`):**
- New `TrendStateMajorityTrendBarsContributor` class with 8 pins: key present; score bounded in [-1, +1] across all fixtures; monotonic bull → +1.0 signed (exactly, because >70% strong-body bull bars maps to +2 raw); monotonic bear → -1.0 signed; choppy 50/50 session → exactly 0.0 (both sides fall in the 40–55% neutral bucket); short df (< 3 bars) → 0.0; bear session has three negative + one non-positive contributor; bull-with-pullbacks confidence ≥ 0.3.
- The 50/50 choppy result is a design-intentional neutral: neither direction crosses the 55% majority threshold. The bull-with-pullbacks fixture also lands at exactly 0 on this contributor (15 up bars / 30 total = 50%, pullback bars don't qualify as bear trend bars because their body_ratio is 0.20). That's fine — `majority_trend_bars` only fires when density is decisively one-sided, which is the original urgency-scorer's design.
- Replay-equivalence inherited from increment 4 (`MajorityTrendBarsReplayEquivalence`) — the helper only wraps the already-tested function.

**Test suite:** 511 passed / 810 subtests (was 503 / 810). Zero regressions.

**Why this was safe autonomously:** trend.py still has zero consumers in the scanner — no pipeline or payload reads it. Per `~/code/aiedge/scanner/CLAUDE.md` "refactors and internal carves are fine autonomously." The dashboard-payload emission still needs Will's nod.

## Next non-breaking increments (still need Will's nod)

- **Wire `trend_state` into the dashboard payload** under a new key. Adds output but no consumer reads it. Allows visual review of the canonical state alongside the existing 13 classifiers. *Requires Will's nod — it's additive but visible in the JSON payload.*
- **Add fifth contributor** to `compute_trend_state`. Best next candidate: `trending_everything` (linreg over closes/highs/lows/body-mids; same direction-conditioned double-call pattern as `trending_swings` and `majority_trend_bars`). Remaining autonomous-OK candidates: `small_pullback_trend` (SPT; 15-bar lookback, 5-sub-check weighted sum), `spike_duration` (from bar 0 forward). All replay-tested. Autonomous if trend.py stays isolated.
- **Weighting decision** (open question #1): equal weights still. With four contributors — two "which direction" (always_in, cycle_phase) and two "how much" (trending_swings, majority_trend_bars) — the natural next step once Pattern Lab WRs are available is to down-weight the "how much" scorers when their magnitude disagrees with the direction scorers (a "does the crowd agree on direction?" prior).

---

## Open questions (defer until step 1 lands)

- Weighting of contributors in the weighted mean: equal? tuned to Pattern Lab WRs? Regime-stratified?
- Does `confidence` need a warmup period like `session_shape.show_on_live_card`?
- Should `age_bars` reset on a "pause" (direction == "none") or only on a flip (up ↔ down)?

None of these block step 1 — the schema accommodates any answer.

---

## References

- Inventory and causality audit: `trend-classification-inventory.md`
- Schema target file (to be created): `aiedge/context/trend.py`
- Shared body-ratio constant today: `aiedge/context/daytype.py:36`
- `always_in` as it exists: `aiedge/signals/components.py:1250-1281`
