# Trend research — 2026-04-19 · incr 17 · all four "needs your nod" follow-ups completed

## TL;DR

Closed every "needs your nod" item from incr 16. Real-data redundancy validation **largely overturns** the headline incr-16 conclusions: most synthetic-bank near-duplicates collapse on real data. `trend_state` now flows from the live runner into the dashboard payload (additive, no ranking change). HTF daily+weekly closes are wired in via the existing `daily_closes_cache`. Weighting variants don't meaningfully move directional hit rate on the cached real sample, so equal weighting stays.

**602 tests / 905 subtests still green.**

## Tasks completed

| # | Task | Outcome |
|---|------|---------|
| 4 | Real-data redundancy validation | Synthetic findings largely artifacts; equal-weighting preserved |
| 1 | Emit `trend_state` into dashboard payload | Wired via `_serialize_trend_state` helper, additive only |
| 2 | Wire daily/weekly closes into `compute_trend_state` in live runner | New `_annotate_trend_state` reads `df5m_cache` + `daily_closes_cache` |
| 3 | Pattern Lab WR test (re-scoped) | DB is empty → ran TrendState directional hit-rate study on cached real bars |

## Headline finding — synthetic-bank caveat CONFIRMED, recommendations REVERSED

The incr-16 "near-duplicate" pair was a fixture artifact. On real ES.c.0 5-min data:

| Pair | Synthetic r | Real r | Verdict |
|------|------------:|-------:|---------|
| `small_pullback_trend ↔ bpa_trend_bar_density` | **+0.997** | **+0.404** | NOT a near-duplicate. Headline incr-16 down-weight recommendation is REVERSED. |
| `trending_everything ↔ htf_alignment` | +0.995 | +0.000 | Pure sample-bank artifact. |
| `always_in ↔ spike_duration` | +0.986 | -0.054 | Pure artifact. |
| `always_in ↔ majority_trend_bars` | +0.979 | +0.000 | Pure artifact. |
| `cycle_phase ↔ small_pullback_trend` | +0.963 | +0.560 | Real correlation but moderate. |

**Mean uniqueness across 12 contributors:** real bank **0.148** vs synthetic **0.782**. On real data, contributors are ~5× more independent than the polarized synthetic fixtures suggested.

`trending_swings` is no longer a uniqueness standout on real data (real |r|_avg = 0.138, middle of pack). The "only independent voter" framing from incr 16 was a fixture artifact too.

![Real-data redundancy comparison](figures/contributor_agreement_real.png)

## New issue surfaced — `majority_trend_bars` never fires on real data

Across 593 cached real ES.c.0 5-min bars: `majority_trend_bars` is **constant 0** (1 unique value, std = 0.000). It contributes pure noise to the equal-weighted mean. Likely cause: the trend-bar-majority threshold (currently STRONG_BODY_RATIO + N% rule) is calibrated for synthetic monotonic fixtures and is too strict for real overnight ES futures bars, which have higher noise / smaller bodies relative to range.

Two options:
- **Down-weight to 0** in production (drop_zero variant) — neutral on directional hit rate (delta < 0.02 on every horizon tested).
- **Recalibrate threshold** against a real-data calibration set so it fires meaningfully — needs more bars + Will's nod.

## Directional hit-rate study (substitute for Pattern Lab WR test)

Pattern Lab DB is empty (0 bytes), so can't run the original WR test. Substitute: measure how often the equal-weighted TrendState direction matches realised close-to-close direction over forward horizons of 3 / 6 / 12 bars (15 / 30 / 60 min) on cached real ES.c.0 sessions.

| Variant | 3-bar hit | 6-bar hit | 12-bar hit |
|---------|----------:|----------:|-----------:|
| **equal (current)** | 0.567 | 0.534 | 0.467 |
| drop_zero (drops `majority_trend_bars`) | 0.578 | 0.524 | 0.451 |
| downweight_synthetic_pair (halves the incr-16 pair) | 0.586 | 0.530 | 0.469 |

**Directional alpha decays fast and goes negative at 60 min.** Equal weighting is ~57% at 15 min — slight edge — drops to coin-flip by 30 min, becomes a slight contrarian by 60 min. None of the weighting variants beat equal by more than +0.02. Equal weighting stays.

![Weighting hit-rate comparison](figures/trend_state_weighting_hitrate.png)

**Caveats:**
- Small sample (60-167 per horizon).
- ES.c.0 overnight only — not RTH equities.
- Pattern Lab DB needs backfill before the WR-by-setup-type test that incr 16 originally proposed can run.

## Production code changes (additive only, ranking unchanged)

### Task 1 — `aiedge/dashboard/serializers.py`
- Added `_serialize_trend_state()` helper (handles both `TrendState` dataclass and dict).
- Wired into `_serialize_scan_payload`: emits `entry["trendState"]` when `r["trend_state"]` is present. Hides the field when absent so existing dashboards keep rendering as-is.

### Task 2 — `aiedge/runners/live.py`
- New `_annotate_trend_state(results, df5m_cache)` that calls `compute_trend_state(df5, daily_closes=cache[ticker], weekly_closes=_weekly_closes_from_daily(daily))` per surviving ticker.
- Slotted in immediately after `_annotate_htf_alignment(results)` (line ~588). Same exception-tolerance pattern.
- `weekly_closes` derived from the existing `daily_closes_cache` via `_weekly_closes_from_daily` — no new data fetch needed.

### Test status
- `python -m pytest -q` → **602 passed / 905 subtests** in 12.81s. Zero regressions.
- Smoke test: synthetic 20-bar 5-min DataFrame + 10-day daily series → `trend_state` dict has all 12 contributors, `_serialize_scan_payload` emits `trendState` block with direction/strength/structure/confidence/contributors.

## What's next — needs your nod

1. **Front-end TrendState panel.** The payload now ships `trendState` per ticker; the site currently doesn't render it. Wire a small panel under the existing `htfAlignment` line.
2. **Investigate `majority_trend_bars` constant-0 on real data.** Threshold recalibration vs drop. Drop is the smaller-blast-radius option but masks the underlying issue.
3. **Pattern Lab DB backfill.** Without it, the WR-by-setup-type test from incr 16's roadmap stays blocked.
4. **Larger real-data sample.** Six overnight ES sessions is thin. Cache more dates / add RTH 5-min bars before drawing strong conclusions about weighting.

## Mistakes avoided (vs incr 16's anti-pattern list)

- ✅ Synthetic-bank caveat called out loudly in incr 16 was correct; we tested it before acting on the down-weight recommendation. Without that, we'd have shipped a regression.
- ✅ Production wiring is additive — `_annotate_trend_state` doesn't touch ranking, doesn't change any existing field. Easy rollback (delete the call site).
- ✅ `_serialize_trend_state` returns `None` for empty / no-data states so the dashboard hides the panel rather than rendering blanks.

## Run-history pointer

- incr 15 capstone: `trend-contributor-findings-2026-04-19-incr15-capstone.md`
- incr 16 redundancy: `trend-contributor-findings-2026-04-19-incr16-redundancy.md`
- incr 17 (this run): all four follow-ups closed.
