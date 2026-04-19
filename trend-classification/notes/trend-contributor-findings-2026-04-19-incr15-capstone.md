---
title: Trend contributor findings — increment 15 capstone (12 of 12 direction-voting contributors wired)
date: 2026-04-19
slice: 12 of 12 direction-voting contributors wired; regime family documented as out-of-scope
---

# Increment 15 capstone — the 12-contributor `TrendState` is inventory-complete

## TL;DR

- **`aiedge/context/trend.py` now aggregates 12 contributors** covering
  every direction-voting classifier in the scanner inventory.
- **The 13th inventory entry — the regime family** (`atr_percentile`,
  `realized_vol_tercile`) — is **explicitly NOT a contributor**. It's
  an amplifier / stratifier, not a direction voter, and does not fit
  the `contributor: score ∈ [-1, +1]` schema.
- **Zero consumers today.** The dashboard payload does not read
  `TrendState` yet. That wiring still needs Will's nod — it's an
  additive output but visible.
- **Test suite: 140 classes / 845 subtests passing.** Every intraday
  contributor is replay-equivalence pinned under the chaos-tail
  harness. `htf_alignment` is by-contract causal (input is
  pre-session daily/weekly history; doesn't read intraday df).
- **Weighting is still equal.** Spec open-question #1 is now well-
  characterised but deliberately deferred until Pattern Lab WRs
  justify a specific rule.

## The five-family taxonomy, fully populated

| Family | Contributors | Role | How it reads |
|---|---|---|---|
| **directional** (2) | `always_in`, `cycle_phase` | Which way is price going **right now**? | Last 5–15 bars |
| **magnitude** (5) | `trending_swings`, `majority_trend_bars`, `trending_everything`, `small_pullback_trend`, `bpa_trend_bar_density` | How much trend is in the session so far? | Session-wide density / linreg / swing-chain |
| **session-memory** (2) | `spike_quality`, `spike_duration` | How clean was the opening / how long did the opening run? | Bar 0 forward; saturates once opening ends |
| **structural** (2) | `day_type`, `session_shape` | What shape is this session? Strict bucket vs soft softmax. | Full prefix; bar-0 anchor evolves with new extremes |
| **regime** (1) | `htf_alignment` | How does the setup align with daily/weekly EMA cross? | Pre-session close series; never reads df |

Every family holds at least one member. The inventory is closed.

## The important findings — at a glance

### Finding 1: blind-spot share is BOUNDED by design, not by oversight

| Fixture | Firing / 12 | Blind contributors | Why |
|---|---|---|---|
| `bull_trend` | 11/12 | `trending_swings` | strictly monotonic → no detectable swings (by construction) |
| `bull_w_pullbacks` | 8/12 | `always_in`, `majority_trend_bars`, `spike_duration`, `day_type` | strict / streak scorers correctly fail on alternating bars |
| `choppy` | 1/12 | everything except a tiny `htf_alignment` flat read | **negative control** — nothing should fire |
| `bear_w_pullbacks` | 8/12 | (symmetric with bull_w_pullbacks) | (same, mirrored) |
| `bear_trend` | 11/12 | `trending_swings` | (mirrored) |

Two useful reads here:

1. **Every blind spot is justified.** `always_in` needs consecutive
   strong bars; pullback fixtures alternate so it can't count. Same
   for `majority_trend_bars` (sits at exactly 50 %, doesn't clear the
   55 % bucket) and `spike_duration` (1-bar opening spike doesn't hit
   the 3-bar threshold). `day_type`'s strict `trend_pct > 0.50`
   threshold fails at exactly 0.50 — deliberately conservative.
2. **8 of 12 firing on pullback fixtures is the *honest* floor.**
   Before the structural pair landed we had 6 of 10 firing; adding
   `day_type` actually dropped the firing ratio (67 % → 60 %) because
   it's stricter than the magnitude family. `session_shape` then
   restored headroom by firing where `day_type` couldn't (60 % → 64 %
   at incr 14 → 67 % at incr 15 with `htf_alignment` adding a
   confluent vote).

See `blind_spot_count.png` for the per-fixture firing vs blind stack.

### Finding 2: equal-weighting drag on bull-to-bear reversal is now 3 of 12

On the bull-opens-then-flips-bear reversal fixture, three contributor
families stay pinned to the opening / HTF bias even after the recency-
aware family has fully flipped bear:

- `spike_quality` + `spike_duration` (session-memory, 2 of 12)
- `htf_alignment` (regime, 1 of 12)
- `day_type` + `session_shape` soften but don't saturate negative

The **equal-weighted all-12 mean settles at ≈ -0.13** post-flip, vs
the **recency-only mean at ≈ -0.55**. That gap is the structural cost
of equal weighting with persistent-opening contributors.

Two candidate remedies remain documented but deferred:

1. Down-weight memory + regime when recency disagrees.
2. Fade memory weight as `len(df)` grows past the spike window.

See `contributor_recency.png` for the bar-by-bar resolution.

### Finding 3: the structural pair is deliberately complementary

`day_type` uses strict buckets (`trend_pct > 0.50` STRICT for
`trend_from_open`). On exactly-50 % sessions it collapses to
`trading_range` and emits 0. `session_shape` uses a softmax over 6
Brooks shapes with no hard threshold — it fires at ≈ ±0.43 on the
same fixtures.

**Aggregating them keeps the conservative bucket without losing soft
signal entirely.** See `structural_pair.png` for the cleanest view.

### Finding 4: HTF is a confirmation, not a veto

`htf_alignment` uses **asymmetric magnitudes** by design:

- `aligned` (both daily and weekly agree with setup direction): **+0.5**
- `opposed`: **-0.2**
- `mixed` / `no_data`: **0**

A bull intraday session with confluent daily+weekly up scores
`htf_alignment ≈ +0.7`; with opposed daily+weekly down it scores
**-0.7** — but the direction label **stays UP** because the 11
intraday contributors still agree. HTF lifts strength on confluence
more than it drops strength on divergence.

**Pattern Lab WRs should determine whether the +0.5 / -0.2 asymmetry
is the right shape.** Until then, this matches Brooks' "trade with
the daily, but don't refuse setups against it" intuition.

See `htf_confluence.png` for the bull-intraday × three-HTF-backdrop view.

### Finding 5: the regime family is correctly NOT a contributor

`atr_percentile` and `realized_vol_tercile` (from
`aiedge/features/regime.py`) answer "how loud is the tape?" — a
**stratifier**, not a direction vote. Forcing them into the
contributor schema would require either:

- collapsing a tercile into a signed score (e.g., high vol = +0.5)
  which conflates "volatile" with "trending", or
- inventing a per-regime weight which belongs in a downstream
  amplifier stage, not in the `TrendState` unification.

Documenting them as **non-contributors** is the honest read. They
remain in the priors-stratification path unchanged.

## Mistakes to avoid (learned the hard way)

1. **Adding a firing contributor ≠ reducing blind spots.**
   Increment 12 (`bpa_trend_bar_density`) caught a first-draft
   over-claim: "new contributor fills blind spot" was wrong. The
   original blind spots stayed — the new contributor added a *firing*
   vote alongside them. The honest metric is `firing / total`, not
   "did we fill the blind set?".
2. **Strict thresholds can make the aggregate weaker, not stronger.**
   Increment 13 (`day_type`) dropped the firing ratio on pullback
   fixtures from 67 % → 60 %. That's not a regression — it's
   `day_type` correctly saying "this is a trading range, not a
   trend". Keep the strict contributor; let the soft `session_shape`
   fire alongside.
3. **Session-memory is not "slow recency" — it's categorically different.**
   `spike_quality` saturates once the opening ends and **never evolves**.
   Equal weighting + 2 of 12 memory contributors → aggregate can lag
   the recency consensus. This is a structural property, not a bug.
4. **By-contract causality is the right tool for `htf_alignment`.**
   The chaos-tail replay harness extends intraday df only — it can't
   prove `htf_alignment` is causal because the function never reads
   df. The correct argument is: input is pre-session history, fully
   observed before session start, no look-ahead surface. Pinned by
   `test_htf_score_independent_of_intraday_chaos`.
5. **`None` default for HTF history is load-bearing.**
   `daily_closes=None, weekly_closes=None` in `compute_trend_state`
   makes `htf_alignment` score exactly 0.0 (`no_data`). The opt-in
   behaviour keeps the existing aggregate stable for every test
   fixture that doesn't supply HTF — essential for the "zero
   consumers, zero payload changes" property.

## Headline visuals (regenerated 2026-04-19)

Eight figures in `vault/Scanner/methodology/figures/`:

1. `contributor_matrix.png` — 5 fixtures × 12 contributors heatmap.
   Reads like a control panel for the whole study.
2. `contributor_differentiation.png` — bull_trend vs bull_w_pullbacks
   vs choppy grouped bars. Annotates the 4 structural blind spots
   and the 8 firing contributors on pullbacks.
3. `trend_state_resolution.png` — bar-by-bar evolution of the
   12 contributors + aggregate direction/strength/confidence on the
   bull-with-pullbacks fixture.
4. `contributor_recency.png` — bull-to-bear reversal, all 12
   contributors traced. Recency flips bear; memory + structural +
   regime stay pinned positive. Aggregate mean settles at -0.13, not
   -0.55.
5. `contributor_family_grid.png` — per-family means across 5
   fixtures. Taxonomy view, reads where families disagree at a
   glance.
6. `blind_spot_count.png` — firing vs blind stacked bars per fixture.
   Quantifies the 8-of-12 floor on pullback fixtures.
7. `day_type_strictness.png` — `day_type` vs `bpa_trend_bar_density`.
   Strict-vs-smooth divergence on pullback fixtures.
8. `structural_pair.png` — `day_type` vs `session_shape`. The
   complementary strict-vs-soft structural pair.
9. `htf_confluence.png` — same bull intraday session, three HTF
   backdrops. Only `htf_alignment` moves (opt-in invariant).
   Direction stays UP across all three.

Regenerate with `python3 tools/visualize_trend_contributors.py` from
the scanner repo.

## What's next (NOT taken autonomously)

1. **Emit `trend_state` into the dashboard payload.** Additive but
   visible output change. Needs Will's nod per
   `~/code/aiedge/scanner/CLAUDE.md`.
2. **Wire real HTF history in the pipeline.** Today nothing passes
   `daily_closes` / `weekly_closes` to `compute_trend_state`, so the
   `htf_alignment` contributor is dormant outside this research
   script. Non-trivial plumbing — needs a deliberate pass.
3. **Pattern Lab WRs → weighting decision.** Open-question #1 has
   enough empirical characterisation now. The next research pass
   should test weight variants on backtest WRs, not more contributors.
4. **Real-data redundancy validation** *(added 2026-04-19, increment 16
   follow-up).* Re-run the contributor pairwise-Pearson study on a
   backtest sample of real intraday sessions. Confirms whether the
   `trending_swings` standout (Finding 7) and the
   `small_pullback_trend` ↔ `bpa_trend_bar_density` near-duplicate
   hold in production, and rules out the polarized-fixture
   artifacts.

## Increment 16 addendum — empirical contributor redundancy

Pure addition: pairwise Pearson r study across the 5-fixture × 26-
prefix per-bar bank. No production code changed. New figure
`figures/contributor_agreement.png`; full discussion in
`trend-contributor-findings-2026-04-19-incr16-redundancy.md`.

**Headline rank** (mean |r| with the other 11 contributors,
**lower = more independent signal**):

```
trending_swings        |r|_avg = 0.335   ← standout, by 2× margin
spike_duration         |r|_avg = 0.770
always_in              |r|_avg = 0.775
session_shape          |r|_avg = 0.776
majority_trend_bars    |r|_avg = 0.780
day_type               |r|_avg = 0.811
htf_alignment          |r|_avg = 0.831
trending_everything    |r|_avg = 0.836
spike_quality          |r|_avg = 0.848
cycle_phase            |r|_avg = 0.870
small_pullback_trend   |r|_avg = 0.876
bpa_trend_bar_density  |r|_avg = 0.879
```

**Top-1 real redundancy candidate.** `small_pullback_trend` ↔
`bpa_trend_bar_density` correlate at r = +0.997. Both magnitude-
family, both per-bar trend-bar density at heart. Strong candidate to
either down-weight as a unit or drop one once Pattern Lab WRs decide.
Several other "top-5 most-correlated" pairs are sample-bank artifacts
(synthetic polarized fixtures + confluent HTF pairing) — see Finding
4 of the redundancy note.

**Why `trending_swings` stands out so far.** It's the only direction-
voter that fires on pullback sessions and not on monotonic — its sign
pattern is the negative of most of the stack. Removing it would
erase the only contributor that uniquely fires where the strict-
threshold contributors silently fail.

## Authority

Autonomous under `~/code/aiedge/scanner/CLAUDE.md` "refactors and
internal carves are fine." No ranking changes, no flags flipped, no
payload changes. The `TrendState` module remains a consumer-less
internal unification target.
