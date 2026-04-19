# SPT — Day-type label stability through the session (2026-04-18, pt 15)

**TL;DR** — The PLAYBOOK's rule 2 (`day_type ∈ {TFO, S&C}`) is a live classification, not an EOD label. On breadth-firing days, only **~41% of "SPT-labelled" detections during the 10:30–11:30 ET entry band match the day's last label** — i.e., at the point you'd take the trade, the label that justified rule 2 is wrong about half the time on the days we care about. Label stability climbs meaningfully after 11:30 ET (60% → 73% → 83% by 14:30). This is not a rule change, but a diagnostic: rule 2 is NOT a clean EOD-regime filter in live contexts, and rule 8 (first-3-gate) plausibly does the heavy lifting of catching the mislabels.

## Why ask this?

Every prior SPT note treats `day_type` as a known input at signal time. In reality, `classify_session_shape` ([aiedge/context/shape.py:274](../../../code/aiedge/scanner/aiedge/context/shape.py#L274)) runs on the live intraday `df` — so the label at 10:30 ET is a snapshot built from ~13 bars, not an EOD verdict. If labels are unstable, rule 2's empirical WR uplift may be overstated (survivorship from filtering on the eventual-TFO days) or understated (noise drowning the true signal).

The 15-note research arc had never measured label stability directly. This note closes that gap.

## Data

Pattern Lab SQLite, `detections` table. Each row stores the `day_type` value as observed when the detection fired.

Two runs examined:

- **`audit_unfiltered_full`** — 2026-04-14 only, 2,637 detections across 599 ticker-days. No filtering — every scan tick is persisted. Captures raw label volatility.
- **`bt-20260417-011938-6e6ab6`** — Jan 22 → Apr 15, 392 detections across 226 ticker-days. Urgency-style filtering upstream. Captures the post-filter picture closer to live-trading population.

"EOD proxy" = label attached to the last detection of that ticker-day. Imperfect (it's the last qualifying setup, not the 15:55 close classifier run) but the only per-ticker snapshot we have without a bar-by-bar rebuild. Bias direction: if anything, pessimistic — last detections skew toward late-session action and may themselves be flips into trading_range.

Script: `scanner/scratch/spt_daytype_stability_2026_04_18.py`. Re-runnable; raw output at `scanner/scratch/_out_spt_daytype_stability_2026_04_18.txt`.

## Q1 — How often does the label flip?

| Run | ticker-days | single-label | ≥ 1 flip | ≥ 2 flips |
|---|---:|---:|---:|---:|
| audit_unfiltered_full (2026-04-14) | 599 | 317 (52.9%) | 282 (47.1%) | 31 (5.2%) |
| bt-20260417-011938-6e6ab6 (3 mo) | 226 | 202 (89.4%) | 24 (10.6%) | 0 |

**Read:** on unfiltered intraday snapshots during a messy breadth day, nearly half of ticker-days carry ≥ 2 distinct day_type labels. On a filtered multi-month population, label volatility looks much tamer — but that's because the filter already skips the ambiguous moments. The gap between 47% and 11% is not about which population is "right"; it's about when you're observing. Before rule 8 fires (i.e., at the instant each detection qualifies), the raw rate is the relevant one.

## Q2 — First → last confusion matrix (2026-04-14, unfiltered)

|  first \ last       | TFO   | S&C   | TR     | TTR    | UND    |
|---------------------|------:|------:|-------:|-------:|-------:|
| trend_from_open     | 32/86 | 3/86  | 47/86  | 2/86   | 2/86   |
| spike_and_channel   | 3/82  | 32/82 | 46/82  | 1/82   | 0/82   |
| trading_range       | 19/167| 11/167| 131/167| 5/167  | 1/167  |
| trending_tr         | 6/58  | 3/58  | 45/58  | 2/58   | 2/58   |
| undetermined        | 7/69  | 1/69  | 26/69  | 7/69   | 28/69  |

**Sticky-SPT rate** (first ∈ SPT → last ∈ SPT):
- trend_from_open: 35/86 = **40.7%**
- spike_and_channel: 35/82 = **42.7%**

**Flip-to-SPT rate** (first ∉ SPT → last ∈ SPT):
- trading_range: 30/167 = 18.0%
- trending_tr: 9/58 = 15.5%
- undetermined: 8/69 = 11.6%

**Implication:** On a breadth day, calling a day "SPT" early is only ~41% right relative to the day's last label. Conversely, ~15% of days that LOOK like trading_range early will end tagged TFO/S&C. Rule 2 is neither a precise include nor a clean exclude in-flight — it's a noisy contemporaneous vote.

For the 3-month filtered run the TFO→TFO (3/6) and S&C→S&C (2/7) cells are too sparse to draw rates from, but the pattern is visibly less chaotic.

## Q3 — Label match probability by session bar

For each detection prior to the ticker-day's last, we score 1 if its label equals the last label and 0 otherwise, then bucket by session_bar_number (1 bar = 5 min; bar 13 ≈ 10:30 ET, bar 58 ≈ 14:15 ET).

All labels (2026-04-14):

| bar range | n    | match% | approx clock |
|-----------|-----:|-------:|:------------:|
| 13-24     | 296  | **37.5%** | 10:30-11:30 |
| 25-36     | 237  | 60.3%  | 11:30-12:30 |
| 37-48     | 214  | 56.5%  | 12:30-13:30 |
| 49-60     | 160  | 73.1%  | 13:30-14:30 |
| 61-72     | 90   | 83.3%  | 14:30-15:30 |

SPT-labelled only:

| bar range | n    | match% |
|-----------|-----:|-------:|
| 13-24     | 96   | 26.0%  |
| 25-36     | 96   | 43.8%  |
| 37-48     | 100  | 38.0%  |
| 49-60     | 51   | 47.1%  |
| 61-72     | 24   | 70.8%  |

**Read:** label stability rises monotonically through the session (ex the 12:30 lunch dip), hitting trust-worthy territory (~70%+) only after bar 49 (13:30 ET). SPT labels specifically are the most unstable band — an SPT label at 10:30 matches EOD only 26% of the time on this breadth day.

## Q: Should this drive a new rule?

**Not yet.** Two reasons:

1. **Rule 8 (first-3-gate) already catches the failure mode.** The PLAYBOOK's first-3-gate was derived from this exact 2026-04-14 day. It gates continuation on observed outcomes, not on label stability — but since unstable-label days are also chop days where the first 3 trades fail, the net effect overlaps. Pt 14's [stack-validation](small-pullback-trend-stack-validation-2026-04-18.md) confirmed rule 8 is load-bearing.

2. **A "label stable for N bars" rule would reduce the rule-6 entry window to 13:00+ ET.** The stability-reaches-60% point falls in the 11:30–12:30 band, and SPT-specific stability doesn't cross 50% until 14:30+. Waiting that long collides with rule 6 (14:15 ET cutoff) and the lunch-sweet-spot intuition from [entry-side-filters](small-pullback-trend-entry-side-filters-2026-04-18.md). Tightening here would likely eliminate more good trades than bad ones.

The right posture is diagnostic, not corrective: **rule 2 is a contemporaneous vote that is right ~40% on the hardest days. The stack works despite this, because downstream filters (rules 3, 4, 5 on urgency/gap/always_in) upweight the high-signal moments and rule 8 catches the remaining mislabels.**

## Hypothesis for future work

If scanner schema ever stores `day_type_history` (label + bar_index over the session), a more surgical test is possible:

- **H:** "Require day_type label to have been TFO or S&C for ≥ 2 consecutive detections before entering" should improve per-trade expectancy on the worst-WR quartile of days (2026-04-14-class breadth days) without hurting the best days.
- Test would need per-bar label snapshots, not just per-detection. Blocked on the same schema gap that blocks Q1 (raw component scores).

Filed as informal Q-25 on the SPT question map.

## Related

- [small-pullback-trend-PLAYBOOK.md](small-pullback-trend-PLAYBOOK.md) — rule 2 lives in row 2 of the stack.
- [small-pullback-trend-short-side-and-daily-gate-2026-04-18.md](small-pullback-trend-short-side-and-daily-gate-2026-04-18.md) — first-3-gate derivation; 2026-04-14 is the reference day.
- [small-pullback-trend-stack-validation-2026-04-18.md](small-pullback-trend-stack-validation-2026-04-18.md) — walk-forward evidence the stack is robust.
- Scanner source: [aiedge/context/shape.py:274](../../../code/aiedge/scanner/aiedge/context/shape.py) — `classify_session_shape`.
