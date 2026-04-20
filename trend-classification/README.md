# Trend classification — current state

**Last updated:** 2026-04-19 · **Latest run:** increment 24 (structure flip-type breakdown — **only 8 % of structure flips are the intended spike → channel evolution**; 30 % are genuine cross-direction reversals but correlate with direction flips at only r = 0.14 — aggregate damping works)

📄 **[Download this run's PDF](pdfs/trend-research-2026-04-19-incr24.pdf)** — phone-readable headline.

> 📚 **Full research archive — [ARCHIVE.md](ARCHIVE.md)** lists every trend-classification increment ever written, with PDFs and notes. Canonical mirror at the aiedge-vault: [github.com/zerosumsystems-ui/aiedge-vault/tree/main/Scanner/methodology](https://github.com/zerosumsystems-ui/aiedge-vault/tree/main/Scanner/methodology).

---

## 👋 What this project is — for everyone

The `aiedge-scanner` watches the US stock market in real time and flags stocks that look like they're about to make a notable move. To do that well, it has to first answer a deceptively simple question:

> **"Is this stock currently trending up, trending down, or just chopping sideways?"**

That sounds easy. It isn't. Over time the codebase had grown **13 different little programs** all trying to answer that same question, all built at different moments, all slightly disagreeing with each other. Like having 13 different speedometers in one car.

This project's job is to:
1. **Unify them** into one canonical "trend reading" the rest of the system can rely on.
2. **Quality-check each one** — figure out which detectors actually work on real market data, and which are silently broken or just noise.

Each numbered increment below (`incr 18`, `incr 19`, etc.) is one research session. Each one ends with a written finding, a few charts, and a PDF. The most recent one is at the top.

**How to read the rest of this page:** every finding has a 🔍 *In plain English* note translating the technical headline. Skip the technical bits if you'd rather — the plain-English notes carry the real story.

---

## TL;DR (for engineers)

The `aiedge-scanner` had **13 parallel "trend-ish" classifiers** doing overlapping work. We've unified them into one canonical `TrendState` that aggregates **12 direction-voting contributors** across **5 families** (directional · magnitude · session-memory · structural · regime). The 13th inventory entry — the regime *amplifier* family — is formally excluded as a stratifier, not a direction voter.

**Status (post-incr-17):** inventory complete, **602 tests / 905 subtests green**, zero look-ahead bias. **`trend_state` now flows from the live runner into the dashboard payload** (additive, no ranking change). HTF daily+weekly closes wired in via the existing `daily_closes_cache`. Front-end panel is the only remaining wiring needed — that needs your nod on the site repo.

## Most recent finding (incr 24) — **only 8 %** of structure flips are benign spike → channel; **30 %** literally cross the bull/bear line, but aggregate damping eats them (r = 0.14 vs direction flips)

### The whole story in a picture

![Structure flip categories](figures/structure_flip_categories.png)

### The whole story in three sentences

Incr 23 left this exact follow-up open: the live classifier's `structure` label flips 16× more often than `direction` (9.53 vs 0.58 per session); are those flips the benign Brooks spike → channel progression, or genuine reversals? **The incr-23 prose said "mostly intended evolution." That was wrong.** Re-reading the 9 425-row trajectory and classifying every one of the 1 905 flips, only **8.3 %** are intended_evolution (bull_spike → bull_channel or bear_spike → bear_channel). **30.2 % are cross_reversal** — the structure label actually crossed the bull/bear boundary. The end-of-day distribution (94 % in channels at bar 78) looked tidy, but the **path** through the session visits trading ranges and flips the bull/bear structure label a median of 3 times. The saving grace: cross_reversal correlates with actual direction flips at only **r = 0.144**, so the aggregator absorbs ~5 of every 6 structure-level reversals without flipping the headline direction.

### What the five categories mean

- **intended_evolution** (8.3 %, 158 flips) — `bull_spike → bull_channel` or `bear_spike → bear_channel`. The Brooks canonical spike → channel progression. **80 % of these happen in bars 10-19** — right when the opening spike ends. This is the well-behaved category.
- **cross_reversal** (30.2 %, 575 flips) — `bull_* → bear_*` or vice versa. Median 3 per session, p95 7 per session. Bimodal timing: 271 in bars 10-19 AND 94 in bars 60-78 (end-of-day chop).
- **consolidation** (30.6 %, 582 flips) — any direction → `trading_range`. Momentum stalls.
- **resumption** (29.5 %, 562 flips) — `trading_range` → any direction. Momentum returns. Roughly balanced with consolidation — sessions enter and leave ranges at similar rates.
- **reverse_evolution** (1.5 %, 28 flips) — `bull_channel → bull_spike`. A re-energising within the same side. Rare.

### When each category fires

![When each category fires](figures/structure_flip_timing.png)

intended_evolution is front-loaded in bars 10-19 — exactly as Brooks' model predicts. cross_reversal / consolidation / resumption are heavy at the open AND heavy at end-of-day, with a quieter middle. The end-of-day chop is the noteworthy pattern: by bar 60, aggregate direction has usually committed with enough strength that structure-level cross-reversals don't propagate — but they're there.

### Does structure cross-reversal track direction flips? Mostly no.

![cross_reversal vs direction flips](figures/structure_flip_vs_direction_flip.png)

Pearson r = **0.144** (n = 200). Most sessions sit on the y = 0 line — lots of structure cross-reversals, zero direction flips. This is the **aggregator working as designed**: twelve contributors vote, and it takes more than a single-family flicker to move the mean past the ±0.05 direction threshold.

### Where each structure goes next

![Transition matrix](figures/structure_transition_matrix.png)

Trading_range is the single most-visited next state from every channel/spike label — it's the session's "rest room." Diagonals blank by construction (same label isn't a flip).

### What this means for live consumption

- **Latch behaviour** — if the front-end ever shows a "trend confirmed" ribbon, it must latch on **direction** (rare flips, known bimodal timing). Don't tie latching to structure — structure literally crosses the bull/bear line 3 times a session on average.
- **Narrative output** — when we tell the trader "session structure is bull_channel", present it as an instantaneous label, maybe with a small "how long has this label held" indicator. Do not summarise structure with a stability or confidence number.
- **No new production recommendation.** The incr-23 rule ("high-confidence live direction iff |strength| ≥ 0.15 AND bar ≥ 20") is reinforced by this pass — gate on direction, not structure.

### Correction to incr 23

Incr 23 prose said "most structure flips are intended Brooks spike → channel evolution, not random flicker." The correct claim is: **the end-of-day distribution is tidy; the path there is not.** 92 % of flips are something other than intended_evolution. Incr 24's note (linked below) records the correction explicitly.

### For the engineers: the numbers

<details>
<summary>Click to expand the technical view</summary>

**Source:** `realtime_stability_stratified_incr23_traj.csv` — 9 425 bar-by-bar rows across 200 real RTH sessions × up to 68 progressive calls, warmup k=10. Pure read-only analysis. No production code touched.

**Category breakdown:**

| category | count | pct |
|---|---:|---:|
| intended_evolution | 158 | **8.3 %** |
| reverse_evolution | 28 | 1.5 % |
| consolidation | 582 | 30.6 % |
| resumption | 562 | 29.5 % |
| **cross_reversal** | **575** | **30.2 %** |
| other (safety net) | 0 | 0.0 % |
| **total** | **1 905** | |

**Timing (flips per 10-bar bucket):**

| category | 10-19 | 20-29 | 30-39 | 40-49 | 50-59 | 60-78 |
|---|---:|---:|---:|---:|---:|---:|
| intended_evolution | **126** | 7 | 12 | 7 | 2 | 4 |
| cross_reversal | 271 | 81 | 76 | 17 | 36 | **94** |
| consolidation | 185 | 103 | 106 | 21 | 46 | 121 |
| resumption | 167 | 101 | 113 | 24 | 47 | 110 |

**Per-session cross_reversal vs direction flips:** Pearson r = **0.144**. Median cross_reversals / session = 3. Median direction flips / session = 0 (from incr 23). Roughly 5 structure-level cross-reversals per 1 direction flip.

**Classifier tool:** `~/code/aiedge/scanner/tools/structure_flip_types_incr24.py`.
**Artifacts:** `structure_flip_breakdown_incr24.csv`, `structure_flip_category_counts_incr24.csv`, `structure_transition_matrix_incr24.csv`, `structure_flip_timing_incr24.csv`, `structure_flip_per_session_incr24.csv`, `structure_flip_types_incr24.json`.

**Mistakes avoided:**
1. Didn't re-run the progressive-k sweep — reused the 9 425-row trajectory CSV.
2. Didn't claim cross_reversal = direction flip. r = 0.144.
3. Didn't propose any production change.
4. Corrected incr 23's prose rather than hiding the contradiction.

</details>

## Previous finding (incr 23) — real-time gate: `|strength| ≥ 0.15 at bar 20` ⇒ **93 %** direction-survival

<details>
<summary>Expand — bimodal flip timing, strength gate, structure trajectory</summary>

![Where flips happen in the session](figures/realtime_flip_timing.png)
![The |strength| gate at bar 20](figures/realtime_strength_predictor.png)

Across 200 sessions: flips cluster bimodally in bars **15-39** (64 % of all 116 flips, peaks at 15-19 and 30-39). Sessions where `|strength| ≥ 0.15` at bar 20 see their direction survive to the final bar **93 %** of the time, vs 47 % for weaker readings. Median lock-in bar drops from 18 → 10 at the 0.15 threshold. **Optional dashboard consumer rule** emerged from this run — display "high-confidence live direction" when `|strength| ≥ 0.15` AND bar count ≥ 20. Zero change to scoring, ranking, or aggregator. See [pdfs/trend-research-2026-04-19-incr23.pdf](pdfs/trend-research-2026-04-19-incr23.pdf).

</details>

## Previous finding (incr 22) — the live classifier almost never flips its mind once it commits

<details>
<summary>Expand — 63 % zero-flip rate, median lock-in bar 11</summary>

![Flip distribution](figures/realtime_stability_flips.png)

Every prior run graded the 12-judge panel on **closed days** — we fed it the whole day and read one verdict. Incr 22 asked a different, more important question: **if a trader listens to the panel live during the day, does it keep changing its mind?** We fed bars one at a time across 300 real trading sessions — up to 68 readings per session, ~20 000 total — and measured how often the direction ("up"/"down") flips. The answer is striking: **63 % of sessions had zero direction flips** from open to close. When the panel commits to a direction, it almost never reverses. The median session "locks in" its final reading by **bar 11** — essentially within the first 55 minutes of trading.

Live readings are usable. See the incr 22 PDF for the full convergence / density / lock-in figure set: [pdfs/trend-research-2026-04-19-incr22.pdf](pdfs/trend-research-2026-04-19-incr22.pdf).

</details>

## Previous finding (incr 21) — one of the 12 judges is a **100 % precision detector**; loosen him to hear him more often

<details>
<summary>Expand — `day_type` TFO sweep, OE × TPM grid, 100 % accuracy across all cells</summary>

![OE × TPM grid](figures/day_type_sweep_heatmap.png)

Another of our 12 judges (`day_type`'s "trend-from-open" read) is the **opposite** of last week's broken `always_in`. He almost never speaks — just **12 %** of trading days at production — **but when he does, he's right 100 % of the time** across 95 fires, zero sign mismatches. Loosening the open-extreme knob from 0.15 → 0.25 gains **+3 pp more speaking days for free**.

**Recommendation (still pending Will's nod):** loosen `OPEN_EXTREME_THRESHOLD: 0.15 → 0.25` in `aiedge/context/daytype.py`. Small, safe, lowest-risk of the pending items.

</details>

## Previous finding (incr 20) — one of the 12 judges is broken; a simple knob change fixes it

<details>
<summary>Expand — `always_in` was below baseline; W=5 → W=10 fixes it</summary>

![Old vs new setting](figures/always_in_layman_before_after.png)

One of our 12 trend judges (`always_in`) was making things **worse**, not better — on 800 real trading days it gave an opinion 36% of the time and was right 58% of the time, which is **worse than just guessing "UP" every day** (that baseline is 66%). Lengthening one of its knobs from "look at the last 5 bars" to "look at the last 10 bars" fixes it — now it gives an opinion on **52%** of days and is right **67%** of the time. Both numbers improve; there's no trade-off.

![Baseline comparison](figures/always_in_layman_baseline_diagram.png)
![How many days does the judge speak](figures/always_in_layman_speaks_counts.png)
![Always-in sweep heatmap](figures/always_in_sweep_heatmap.png)
![Pareto view](figures/always_in_sweep_pareto.png)

**Recommendation:** change `ALWAYS_IN_WINDOW: 5 → 10` in `aiedge/context/trend.py` and mirror `STRONG_TREND_WINDOW` in `aiedge/signals/components.py`. `DIRECTION_MIN_CONSEC` stays at 2.

</details>

## Previous finding (incr 19) — 12-contributor degeneracy audit · 2 silent-fail · sharp directional-accuracy hierarchy

> 🔍 **In plain English.** Imagine the trend reading is decided by a panel of **12 judges**, each looking at the same stock from a different angle. We sat them all down with **800 real trading sessions** (across 387 different US stocks) and graded them.
>
> - **Two judges never spoke at all.** One is broken (its threshold to speak is set so high it almost never trips); the other is technically working but isn't being given the inputs it needs.
> - **The other ten varied wildly in skill.** The best judge was right **97.9%** of the time when she spoke; the worst was right only **57.6%** of the time — barely better than guessing "up" every time.
> - **Right now we count every judge's vote equally.** That means a near-perfect judge gets the same weight as a coin-flipper.
>
> This study didn't change anything in production — it just identified which judges are doing real work and which need to be fixed or re-tuned next.

Same 800 RTH 5-min equity sessions / 387 symbols as incr 18. Asked the natural follow-up: **are other contributors silently failing the same way as `majority_trend_bars`**, and how much directional information does each one actually carry per session?

![Fire rate ranking](figures/contributor_degeneracy_fire_rate.png)

**Headline numbers:**
- **Two silent-fail contributors** out of 12. `htf_alignment` (**0/800** fires — dormant by design when daily/weekly closes aren't passed in; the offline path silently zeroes it) and `majority_trend_bars` (**1/800** fires — confirms incr 18 from a fresh code path).
- **`session_shape` is the workhorse** — fires on **100%** of sessions, **644 unique** outputs, entropy **2.37 bits / 3.0 max**, **96.3%** directional accuracy vs realised open → close.
- **`trending_everything` is the conviction voter** — only fires on **49.6%** of sessions, but **88.7% saturation** at `|score| ≥ 0.9` and **97.9%** directional accuracy. Closest thing to a binary high-confidence signal.
- **`always_in` is surprisingly weak** — 36% fires, **57.6%** directional accuracy. Below the 65.95% always-predict-up base rate from incr 18. The 5-bar window appears to overfit.
- **Sharp tier list when contributors fire:** A (≥90%) `trending_everything`, `session_shape` · B (75-90%) `spike_duration`, `bpa_trend_bar_density`, `day_type`, `trending_swings` · C (60-75%) `small_pullback_trend`, `cycle_phase`, `spike_quality` · D (50-60%) `always_in` · F `majority_trend_bars`, `htf_alignment`.

![Distribution panels](figures/contributor_degeneracy_distribution.png)

**Important caveat — same-session label fidelity, not forward predictive value.** "Directional accuracy" here = sign matches realised open → close on the SAME session. A contributor that fires LATE in a session has the easier task. Pattern Lab WR-driven weighting (still blocked on DB backfill) remains the right path to actually update equal weighting in production.

## Previous finding (incr 18) — `majority_trend_bars` is gated by the **40% majority floor**, not the body-ratio threshold

> 🔍 **In plain English.** Last week we noticed that one of the 12 judges (`majority_trend_bars`) almost never spoke up. Like a smoke detector that refuses to beep unless half the room is on fire.
>
> The question was: *which knob is wrong?* It has two — one for "how strong does each candle have to be" and one for "what fraction of candles need to agree." Most people (me included) would have assumed the first knob was the problem.
>
> We tested both knobs across **800 real trading sessions**. The answer:
>
> - **The candle-strength knob is fine** — most real candles already pass it.
> - **The agreement-fraction knob is set way too high** — it requires 40% of all candles in the session to lean the same way AND be strong, which almost never happens. Reality is closer to 25%.
>
> If we lower that knob from 40% to 25%, the judge would speak up on **78 of 800 sessions** instead of 1, and would be right about **90%** of the time. That fix is recommended but waiting on Will's go-ahead before we touch production code.

Took the incr-17 finding that `majority_trend_bars` was constant 0 on real data and ran the recalibration study on **800 RTH 5-min equity sessions across 387 symbols** (the full `cache/databento/` parquet store). Two-axis sweep: body-ratio threshold {0.30…0.60} × majority floor {0.20…0.50}.

![Floor sweep](figures/majority_trend_bars_floor_sweep.png)

**Headline numbers:**
- At production thresholds (body=0.50, **floor=0.40**): classifier fires in **1 of 800 sessions** (0.12%). On that single fire, the vote was **wrong**.
- Lowering the floor to **0.25**: fire rate **9.75%** (78 sessions), directional accuracy **~90%** vs realised open → close session move.
- Decomposed by side: up-pred **92.2%** (vs 66% base rate), down-pred **87.0%** (vs 34% base rate). Both sides beat their base rate by **+20 to +25 pp**.
- **Body-ratio threshold sweep is not the answer.** Even at body=0.30, fire rate is 1.9%. The floor was the lever all along.

**Recommendation (still needs your nod):** introduce `MAJORITY_TREND_BAR_FLOOR = 0.25` in `aiedge/signals/components.py` and have `_score_majority_trend_bars` use it. Body-ratio threshold stays at 0.50.

## Previous finding (incr 17) — synthetic-bank caveat CONFIRMED on real data; recommendations REVERSED

> 🔍 **In plain English.** A week ago we ran a study on **made-up data** (test fixtures we wrote by hand) and concluded that some of our 12 judges were basically duplicates of each other. Our recommendation was: "drop the redundant ones, save effort."
>
> This time we re-ran the **same study on real market data**. The result flipped: judges that looked like twins on fake data are actually quite independent on real data. The "duplicate" was a side effect of how simple our test fixtures were — real markets are messier and force the judges to disagree more.
>
> **We reversed our previous recommendation.** Lesson: never act on a fake-data finding without re-validating on real data first.

Re-ran the incr-16 redundancy study on **real ES.c.0 5-min bars** from the cache (6 sessions, 593 bar prefixes). Side-by-side comparison vs the synthetic-fixture bank:

![Real vs synthetic uniqueness](figures/contributor_agreement_real.png)

**Headline numbers:**
- `small_pullback_trend ↔ bpa_trend_bar_density`: synthetic **r = +0.997** → real **r = +0.404**. **NOT a near-duplicate.** The incr-16 down-weight recommendation is REVERSED.
- `trending_everything ↔ htf_alignment`: synthetic +0.995 → **real ~0**. Pure polarized-fixture artifact.
- Mean uniqueness across 12 contributors: real **0.148** vs synthetic **0.782** — real shows **~5× more independence**.
- `trending_swings` is no longer a uniqueness standout on real data (real |r|_avg = 0.138, middle of pack).

## Weighting hit-rate study — equal weighting stays

> 🔍 **In plain English.** Right now every judge's vote counts the same. We tested whether **giving smarter judges louder votes** would predict the next 15 / 30 / 60 minutes better than treating everyone equally.
>
> It didn't. Every weighting scheme we tried landed within a rounding error of equal weighting. So we kept it simple: **everyone still votes equally** until we have a better reason to change that.

Pattern Lab DB is empty (0 bytes), so substituted: how often does the equal-weighted TrendState direction match realised close-to-close direction over forward 3 / 6 / 12 bars on cached real sessions?

![Weighting hit-rate comparison](figures/trend_state_weighting_hitrate.png)

- **15 min:** equal **0.567** · drop_zero 0.578 · downweight_pair 0.586 — slight edge over coin flip.
- **30 min:** all variants ≈ 0.53 — coin flip.
- **60 min:** all variants ≈ 0.46 — slight contrarian.

No weighting variant beats equal by more than +0.02. **Equal weighting stays.**

## Why `trending_swings` matters — the blind-spot story

> 🔍 **In plain English.** Most of our 12 judges go quiet on **choppy, two-steps-forward-one-step-back** sessions — the kind of day where price grinds higher but with frequent pullbacks. They only speak up on clean, monotonic moves.
>
> One judge — `trending_swings` — is the opposite. **She's the only one who notices choppy uptrends**, and she stays quiet on the smooth ones the others handle.
>
> Without her, the panel would be **completely silent on roughly a third of trading days**. So even though her vote pattern looks "weird" compared to the others, **removing her would create a giant blind spot**. Keep her at full weight.

![Firing vs blind contributors per fixture](figures/blind_spot_count.png)

`trending_swings` is the only contributor that **fires on pullback sessions and stays silent on monotonic** — exactly opposite to most of the stack. That's why its sign pattern is unique. The strict-threshold contributors (`always_in`, `majority_trend_bars`, `spike_duration`, `day_type`) go blind on the 4 of 12 pullback fixtures; `trending_swings` covers them.

**Removing `trending_swings` would erase the only contributor that uniquely fires where the strict-threshold contributors silently fail. Keep at full weight.**

## The full 12 × 5 control panel

> 🔍 **In plain English.** Think of this as a **scoreboard with 12 columns (one per judge) and 5 rows (one per "type of trading day")** — strong uptrend, choppy uptrend, sideways, choppy downtrend, strong downtrend.
>
> Dark green means a judge is shouting "this is going up." Dark red means "this is going down." White means "I'm not sure / I'm silent." If our 12 judges agree well, you should see mostly green across the top rows and mostly red across the bottom rows. The matrix lets you see, at a glance, **which judges are confident on which kinds of days** — and which ones are noisy outliers.

![Contributor matrix](figures/contributor_matrix.png)

Each row is a canonical market regime. Each column is one classifier's signed score. Dark green = strong long, dark red = strong short. Reads like a control panel for the whole study.

## Equal-weighting drag on bull-to-bear reversal

> 🔍 **In plain English.** When a market that's been going **up all morning suddenly flips and starts going down**, how fast does our 12-judge panel notice?
>
> The answer: **it depends on which judges you ask.**
> - The seven judges who only look at the last few candles flip their vote within a couple of minutes — they catch the reversal cleanly.
> - The five judges who look at "the whole day so far" or "yesterday's close" stay anchored to the morning's uptrend. They're slow to flip.
>
> Because every judge votes equally, the panel's overall reading after the flip is a tepid **-0.13** ("kinda down?") instead of a confident **-0.55** ("clearly down"). Eventually we may want to **temporarily down-weight the slow judges during fast reversals**, but only once we have hard data showing it would have made money. For now, equal weighting stays.

![Recency vs memory vs structural vs regime](figures/contributor_recency.png)

When a bull session flips bear at bar 8, the seven recency-aware contributors rotate negative within a few bars. But session-memory + structural + regime contributors stay anchored to the opening / HTF bias. The all-12 mean settles at **-0.13** post-flip vs the recency-only mean at **-0.55**. Quantifies the case for eventually weighting these families down — once Pattern Lab WRs justify it.

## What's next — still needs your nod

1. **NEW (incr 24)** — no new code recommendation. The pass sharpens existing rules: latch on direction, treat structure as an instantaneous label with no stability semantics. The incr-23 strength-gate rule is reinforced.
2. **STILL PENDING (incr 23)** — optional dashboard consumer rule: display "high-confidence live direction" when `|strength| ≥ 0.15` AND bar count ≥ 20. Zero change to `compute_trend_state`, scoring, or ranking. Relevant only if/when the front-end `TrendState` panel lands.
3. **STILL PENDING (incr 22)** — no new threshold recommendations from the stability run.
3. **STILL PENDING (incr 21):** loosen `OPEN_EXTREME_THRESHOLD: 0.15 → 0.25` in `aiedge/context/daytype.py::_classify_day_type` (currently a magic literal). Gains +3 pp TFO fire rate with zero accuracy cost. Lowest-risk of the pending items.
4. **STILL PENDING (incr 20):** change `ALWAYS_IN_WINDOW: 5 → 10` in `aiedge/context/trend.py` and mirror `STRONG_TREND_WINDOW` in `aiedge/signals/components.py`. Re-baseline `TrendStateAlwaysInContributor` replay tests.
5. **DOCUMENTATION (zero blast radius, incr 19):** add a one-line note to `compute_trend_state` docstring flagging that callers passing `None` for `daily_closes`/`weekly_closes` get a silent zero contribution from `htf_alignment`.
6. **STILL PENDING (incr 18):** introduce `MAJORITY_TREND_BAR_FLOOR = 0.25` in `aiedge/signals/components.py`. Body-ratio stays at 0.50.
7. **Front-end `TrendState` panel** (even stronger case now — incr 22 verified stability + incr 23 provides the strength gate). Payload already ships `trendState` per ticker; site doesn't render it yet.
8. **NEXT STUDY (incr 23 surfaced):** structure-flip type taxonomy — split intended-evolution flips (bull_spike → bull_channel) from genuine reversals (bull_channel → bear_channel).
9. **NEXT STUDY (incr 23 surfaced):** strength-at-bar-30 gate — likely 98 %+ direction-survival at |strength| ≥ 0.15, trade-off is 10 extra minutes.
10. **NEXT STUDY (incr 22 surfaced, still open):** stratify lock-in timing by `day_type`. Likely bimodal — TFO days lock in immediately, chop days may never lock in.
11. **NEXT STUDY (incr 21 surfaced):** full `day_type` sweep covering `spike_and_channel`, `trending_tr`, `trading_range` branches (TFO alone fires at 12 %; full classifier fires at 19.6 %).
12. **Pattern Lab DB backfill.** Without it, the WR-by-setup-type test from incr 16's roadmap stays blocked.
13. **Multi-month sample.** 10-day cache is fine for stability (clean signal) but multi-month would confirm 63 % zero-flip isn't window-specific.

## All figures (40)

- [structure_flip_categories.png](figures/structure_flip_categories.png) — incr 24 flip-category breakdown (8 % intended, 30 % cross-reversal) *(NEW)*
- [structure_flip_timing.png](figures/structure_flip_timing.png) — incr 24 stacked timing by category *(NEW)*
- [structure_transition_matrix.png](figures/structure_transition_matrix.png) — incr 24 row-normalised 5×5 transition heatmap *(NEW)*
- [structure_flip_vs_direction_flip.png](figures/structure_flip_vs_direction_flip.png) — incr 24 cross_reversal vs direction flips scatter *(NEW)*
- [realtime_flip_timing.png](figures/realtime_flip_timing.png) — incr 23 where-in-session flips happen (bimodal)
- [realtime_strength_predictor.png](figures/realtime_strength_predictor.png) — incr 23 |strength| ≥ 0.15 at bar 20 as a real-time gate *(NEW)*
- [realtime_structure_density.png](figures/realtime_structure_density.png) — incr 23 structure trajectory across the session *(NEW)*
- [realtime_dir_vs_struct_flips.png](figures/realtime_dir_vs_struct_flips.png) — incr 23 direction vs structure flip histogram *(NEW)*
- [realtime_stability_convergence.png](figures/realtime_stability_convergence.png) — incr 22 per-bar match-vs-final convergence profile
- [realtime_stability_flips.png](figures/realtime_stability_flips.png) — incr 22 direction-flip histogram per session *(NEW)*
- [realtime_stability_lockin.png](figures/realtime_stability_lockin.png) — incr 22 first-bar lock-in distribution *(NEW)*
- [realtime_stability_density.png](figures/realtime_stability_density.png) — incr 22 up/none/down breakdown at bars 10, 20, 30, 40, 50, 60, 78 *(NEW)*
- [day_type_sweep_heatmap.png](figures/day_type_sweep_heatmap.png) — incr 21 OE × TPM grid: fire / accuracy / median |score|
- [day_type_sweep_pareto.png](figures/day_type_sweep_pareto.png) — incr 21 fire vs accuracy scatter; every cell at 100 % accuracy *(NEW)*
- [day_type_sweep_prod_vs_top.png](figures/day_type_sweep_prod_vs_top.png) — incr 21 production vs top fire-rate cells (fire ≥ 10 %) *(NEW)*
- [always_in_layman_before_after.png](figures/always_in_layman_before_after.png) — incr 20 plain-English before/after on the two questions that matter
- [always_in_layman_baseline_diagram.png](figures/always_in_layman_baseline_diagram.png) — incr 20 is-it-beating-the-dumb-strategy view *(NEW)*
- [always_in_layman_speaks_counts.png](figures/always_in_layman_speaks_counts.png) — incr 20 session-count stacked bar (speaks vs silent) *(NEW)*
- [always_in_sweep_heatmap.png](figures/always_in_sweep_heatmap.png) — incr 20 W × K grid: fire / accuracy / median |score|
- [always_in_sweep_pareto.png](figures/always_in_sweep_pareto.png) — incr 20 fire vs accuracy scatter with always-up baseline overlay *(NEW)*
- [always_in_sweep_prod_vs_top.png](figures/always_in_sweep_prod_vs_top.png) — incr 20 production vs top-accuracy candidates (fire ≥ 15%) *(NEW)*
- [contributor_degeneracy_fire_rate.png](figures/contributor_degeneracy_fire_rate.png) — incr 19 per-contributor fire rate ranking
- [contributor_degeneracy_entropy.png](figures/contributor_degeneracy_entropy.png) — incr 19 output Shannon entropy ranking *(NEW)*
- [contributor_degeneracy_distribution.png](figures/contributor_degeneracy_distribution.png) — incr 19 12-panel signed-score histograms *(NEW)*
- [majority_trend_bars_floor_sweep.png](figures/majority_trend_bars_floor_sweep.png) — incr 18 majority-floor sensitivity + directional accuracy
- [majority_trend_bars_body_ratio_distribution.png](figures/majority_trend_bars_body_ratio_distribution.png) — incr 18 per-bar body-ratio histogram + CDF
- [majority_trend_bars_threshold_sensitivity.png](figures/majority_trend_bars_threshold_sensitivity.png) — incr 18 body-ratio sweep (proves threshold isn't the lever)
- [majority_trend_bars_session_scores.png](figures/majority_trend_bars_session_scores.png) — incr 18 per-threshold session score buckets
- [contributor_agreement_real.png](figures/contributor_agreement_real.png) — incr 17 real-data validation heatmap + side-by-side uniqueness
- [trend_state_weighting_hitrate.png](figures/trend_state_weighting_hitrate.png) — incr 17 weighting variant hit rates
- [contributor_agreement.png](figures/contributor_agreement.png) — incr 16 synthetic redundancy heatmap (now known to be inflated)
- [contributor_matrix.png](figures/contributor_matrix.png) — 12 × 5 control panel
- [blind_spot_count.png](figures/blind_spot_count.png) — firing vs blind per fixture
- [contributor_recency.png](figures/contributor_recency.png) — bull-to-bear reversal, 4-family resolution
- [contributor_family_grid.png](figures/contributor_family_grid.png) — family means across 5 fixtures
- [contributor_differentiation.png](figures/contributor_differentiation.png) — bull vs pullback vs choppy bars
- [trend_state_resolution.png](figures/trend_state_resolution.png) — bar-by-bar evolution on bull-with-pullbacks
- [structural_pair.png](figures/structural_pair.png) — `day_type` vs `session_shape` strict-vs-soft
- [day_type_strictness.png](figures/day_type_strictness.png) — `day_type` vs `bpa_trend_bar_density`
- [htf_confluence.png](figures/htf_confluence.png) — same intraday, three HTF backdrops

## Long-form notes

- [trend-contributor-findings-2026-04-19-incr24-structure-flip-types.md](notes/trend-contributor-findings-2026-04-19-incr24-structure-flip-types.md) — most recent run, structure flip-type breakdown *(NEW)*
- [trend-contributor-findings-2026-04-19-incr23-flip-timing.md](notes/trend-contributor-findings-2026-04-19-incr23-flip-timing.md) — incr 23 flip timing + strength gate + structure trajectory
- [trend-contributor-findings-2026-04-19-incr22-realtime-stability.md](notes/trend-contributor-findings-2026-04-19-incr22-realtime-stability.md) — real-time stability
- [trend-contributor-findings-2026-04-19-incr21-day-type-sweep.md](notes/trend-contributor-findings-2026-04-19-incr21-day-type-sweep.md) — `day_type` TFO-gate sweep
- [trend-contributor-findings-2026-04-19-incr20-always-in-sweep.md](notes/trend-contributor-findings-2026-04-19-incr20-always-in-sweep.md) — `always_in` two-axis sweep
- [trend-contributor-findings-2026-04-19-incr19-degeneracy.md](notes/trend-contributor-findings-2026-04-19-incr19-degeneracy.md) — 12-contributor degeneracy + accuracy hierarchy
- [trend-contributor-findings-2026-04-19-incr18-majority-floor.md](notes/trend-contributor-findings-2026-04-19-incr18-majority-floor.md) — majority-floor recalibration study
- [trend-contributor-findings-2026-04-19-incr17-followups.md](notes/trend-contributor-findings-2026-04-19-incr17-followups.md) — incr 17, all 4 follow-ups closed
- [trend-contributor-findings-2026-04-19-incr16-redundancy.md](notes/trend-contributor-findings-2026-04-19-incr16-redundancy.md) — synthetic redundancy study (largely overturned by incr 17)
- [trend-contributor-findings-2026-04-19-incr15-capstone.md](notes/trend-contributor-findings-2026-04-19-incr15-capstone.md) — read first if cold
- [trend-classification-inventory.md](notes/trend-classification-inventory.md) — original 13-classifier inventory
- [trend-state-canonical-spec.md](notes/trend-state-canonical-spec.md) — the schema

## Where the code lives

- Aggregator: `~/code/aiedge/scanner/aiedge/context/trend.py` (978 LOC)
- Tests (143 classes / 905 subtests): `~/code/aiedge/scanner/tests/context/test_causality.py`
- Figure regenerator: `~/code/aiedge/scanner/tools/visualize_trend_contributors.py`
- Vault canonical: `~/code/aiedge/vault/Scanner/methodology/`

## Run history

- **incr 24** (2026-04-19) — structure flip-type breakdown. Pure read-only re-analysis of the 9 425-row trajectory persisted by incr 23. Classified all 1 905 structure flips: **8.3 % intended_evolution** (bull_spike → bull_channel, bear_spike → bear_channel — 80 % of these in bars 10-19). **30.2 % cross_reversal** (bull_* ↔ bear_*, bimodal timing: open + end-of-day). **30.6 % consolidation** (→ trading_range). **29.5 % resumption** (trading_range →). **1.5 % reverse_evolution** (channel → spike, same side). cross_reversal ↔ direction flips Pearson r = **0.144** — aggregate damping absorbs most structure-level reversals. **Corrects incr 23 prose** ("mostly intended evolution" was wrong — end-of-day distribution is tidy, path is not). Zero production code change. Reinforces the incr 23 front-end rule: gate on direction, not structure.
- **incr 23** (2026-04-19) — stratified real-time stability: flip timing, strength-as-predictor, structure trajectory. 200 RTH sessions, 12 129 per-bar trajectory rows. **64 % of all 116 direction flips fall in bars 15-39** (bimodal peaks at 15-19 and 30-39). **|strength| ≥ 0.15 at bar 20 ⇒ 93 % direction-survival** (vs 47 % baseline), 85 % zero-flip rate (vs 46 %), median lock-in bar 10 (vs 18). `TrendState.structure` is 16× noisier than direction (1 905 vs 116 flips) but most movement is intended spike → channel evolution. Reproduced incr 22 stability: 0.58 mean direction flips / session vs 0.56. **One optional dashboard consumer rule.** No production change.
- **incr 22** (2026-04-19) — real-time stability of `compute_trend_state`. First run to grade the classifier **live** (progressive bar-by-bar calls) rather than on closed sessions. 300 RTH sessions × up to 68 calls each ≈ 20k `compute_trend_state` calls. **63.3 % of directional sessions have zero direction flips**; mean 0.56, max 5. Median lock-in bar = 11, mean = 19.5. 70 % of sessions lock in by bar 20, 93 % by bar 40. Per-bar convergence to final direction: 50 % at bar 10, 75 % at bar 36, 90 % at bar 75. **Conclusion: TrendState is safe to wire into front-end live consumption.** No production change.
- **incr 21** (2026-04-19) — `day_type` TFO-gate sweep (OPEN_EXTREME × TREND_PCT_MIN, 20 cells) on the same 800 RTH 5-min equity sessions / 387 symbols. TFO gate is a **100 % precision detector** — zero sign mismatches in every cell, including the loosest (127 fires at OE=0.30, TPM=0.40). Coverage is the only lever worth tuning; OE is binding (fire rate 9.6 → 14.9 % as OE loosens 0.10 → 0.30). Recommendation: loosen OE 0.15 → 0.25 for +3 pp coverage. **Read-only — no production change.**
- **incr 20** (2026-04-19) — `always_in` two-axis sweep (ALWAYS_IN_WINDOW × DIRECTION_MIN_CONSEC) over 800 RTH 5-min equity sessions. Production (W=5, K=2) fires 36% / 57.6% accuracy — below the 66% always-up baseline. W=10, K=2 strictly dominates: 52% / 66.9%. Binding knob is W, not K. Recommendation: raise window to 10. **Read-only — no production change.**
- **incr 19** (2026-04-19) — 12-contributor degeneracy + directional-accuracy hierarchy on the same 800 RTH 5-min equity sessions / 387 symbols as incr 18. Two silent-fail contributors confirmed: `htf_alignment` (0/800, dormant by design) and `majority_trend_bars` (1/800, confirms incr 18). Sharp tier list when contributors fire: A `trending_everything` (97.9%), `session_shape` (96.3%) → D `always_in` (57.6%) → F silent-fail. New tools, 3 figures, recommended next sweeps for `always_in` and `day_type`. **Read-only — no production change.**
- **incr 18** (2026-04-19) — `majority_trend_bars` floor recalibration study on 800 RTH 5-min equity sessions across 387 symbols. Floor 0.40 → 1/800 fires. Floor 0.25 → 78/800 fires with 90% directional accuracy. Body-ratio threshold sweep proved it's NOT the lever. Recommendation: introduce `MAJORITY_TREND_BAR_FLOOR = 0.25`. **Read-only — no production change.**
- **incr 17** (2026-04-19) — all four "needs your nod" follow-ups closed. Real-data redundancy validation overturns most incr-16 conclusions. `trend_state` wired into live runner + dashboard payload (additive). Weighting study confirms equal weighting stays. New figures + PDF. **602 tests / 905 subtests still green.**
- **incr 16** (2026-04-19) — empirical contributor redundancy study (synthetic). New `contributor_agreement.png` + first PDF. Pure addition, zero production code change. *Largely overturned by incr 17 real-data validation.*
- **incr 15** (2026-04-19) — capstone. `htf_alignment` wired as 12th contributor. Inventory complete.
- **incr 14** (2026-04-19) — `session_shape` wired (11th, structural-pair).
- **incr 13** (2026-04-19) — `day_type` wired (10th, first structural).
- **incr 12** (2026-04-19) — `bpa_trend_bar_density` wired (9th).
- **incr 11** (2026-04-19) — `spike_duration` wired (8th).
- **incr 10** (2026-04-19) — `spike_quality` wired (7th, first session-memory).
- **incr 9** (2026-04-19) — `small_pullback_trend` wired (6th).
- **incr 8** (2026-04-19) — `trending_everything` wired (5th).
- **incr 7** (2026-04-19) — `majority_trend_bars` wired (4th).
- **incr 6** (2026-04-18) — `trending_swings` wired (3rd).
- **incr 5** (2026-04-18) — `cycle_phase` wired (2nd).
- **incr 1–4** (2026-04-18) — `TrendState` schema, replay-equivalence harness, body-ratio renames, `always_in` (1st contributor).
