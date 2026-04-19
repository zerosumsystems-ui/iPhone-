# SPT — Low-n day behavior (Q22 resolution)

Pt 12 of the [[small-pullback-trend-INDEX|SPT research series]]. Closes Q22 ("first-3-gate behavior on very-low-n days, < 4 triggers"). Data window unchanged: 2025-12-16 → 2026-04-17. Script: `~/code/aiedge/scanner/scratch/spt_low_n_days_2026_04_18.py`.

## The question

Pt 11's daily-confirmation gate (rule 9: "after the day's first 3 policy-stack trades, continue only if ≥ 2 won") is structurally silent on days with < 4 total triggers. The gate passes those through ungated because there's nothing to confirm. Q22 asked: is that silent-passthrough a hidden loser, or are low-n days genuinely harmless?

## Population

All trades that pass rules 1–7 + the short-side 11:30 ET carveout, data through 2026-04-17:

- n = 171, WR = 57.9%, sumR = +187.87, +1.099R/trade.
- 61 distinct trading days. After applying the rule-9 daily-gate (first-3 ≥ 2 wins): n = 147.

## Day-count distribution

| Day-n bucket | # days | trades | WR% | sumR | perR |
|:------------:|:------:|:------:|----:|------:|------:|
| n=1 (singletons) | 27 | 27 | 37.0 | +10.62 | +0.394 |
| n=2 | 15 | 30 | 60.0 | +42.00 | +1.400 |
| n=3 | 1 | 3 | 0.0 | −1.44 | −0.481 |
| n=4–5 | 10 | 41 | 68.3 | +53.62 | +1.308 |
| n=6–10 | 5 | 33 | 63.6 | +40.85 | +1.238 |
| n=11–20 | 2 | 37 | 59.5 | +42.23 | +1.141 |

**72% of trading days are low-n** (< 4 triggers), contributing 35% of trade volume and only 29% of total R.

## §1 — Low-n vs high-n, aggregate

| Bucket | days | n | WR% | sumR | perR |
|--------|:---:|:--:|----:|------:|-------:|
| low-n (< 4 triggers) | 43 | 60 | 46.7 | +51.18 | +0.853 |
| high-n (≥ 4 triggers) | 17 | 111 | 64.0 | +136.69 | +1.231 |

A +0.38R/trade gap and 17 pp WR gap. Low-n days are materially weaker.

## §2 — The damage is inside the singleton bucket, not the whole low-n bucket

Decomposing low-n by exact trigger count:

- **n=1 (singletons)**: 37.0% WR, +0.394R/trade — the problem cell.
- **n=2**: 60.0% WR, +1.400R/trade — as good as high-n days.
- **n=3**: one day only (2026-01-27), not enough to say.

The "low-n is weak" story is really a **singleton story**. n=2 days behave like the healthy population.

### Singleton breakdown

| Axis | n | WR% | sumR | perR |
|------|:--:|----:|------:|-------:|
| longs | 12 | 41.7 | +6.63 | +0.552 |
| shorts | 15 | 33.3 | +4.00 | +0.266 |
| trend_from_open | 11 | 45.5 | +8.06 | +0.732 |
| spike_and_channel | 16 | 31.2 | +2.57 | **+0.161** |

Weakest cell: spike_and_channel singletons. Shorts in spike-and-channel with no follow-up fire on the day rarely build continuation.

## §3 — But rule 7 already solves this

The PLAYBOOK's rule 7 is **"skip the 1st ticker-day detection."** A singleton day has exactly one detection — which is, by definition, the 1st. Rule 7 already rejects every singleton day; no new rule is needed.

Verification: apply full stack (rules 1–9) with and without an explicit `day_n ≥ 2` filter:

| Stack | n | WR% | sumR | perR |
|-------|:--:|----:|------:|-------:|
| Full stack, current | 37 | 75.7 | +60.91 | +1.646 |
| Full stack + `day_n ≥ 2` | 37 | 75.7 | +60.91 | +1.646 |

Identical. `day_n ≥ 2` is a no-op once skip-1st applies.

## §4 — Where the residual low-n exposure hides: intermediate stacks

The `day_n ≥ 2` filter only differs from current behavior at **intermediate stack depths** (rules 1–6 or 1–8), where the user trades more but with thinner filtering:

| Stack depth | current n | current perR | + `day_n ≥ 2` n | + `day_n ≥ 2` perR |
|-------------|:---------:|:-------------:|:----------------:|:-------------------:|
| 1–6 (entry filters only) | 147 | +1.154 | 120 | +1.325 |
| 1–9 (full PLAYBOOK) | 37 | +1.646 | 37 | +1.646 |

The stack-1–6 trader gives up 27 trades / +10.62R of expected value but gains +0.17R/trade. A 1.5% total-R loss for a 15% per-trade quality improvement. This is a volume-vs-quality tradeoff the PLAYBOOK's stack-depth choice already encodes.

## §5 — Trigger-count monotonicity check

Is per-R monotonic in daily trigger count? Partly — the ordering is U-shaped, not clean:

| day-n | n | WR% | perR |
|:------|:--:|----:|-------:|
| 1 | 27 | 37.0 | +0.394 |
| 2 | 32 | 56.2 | +1.250 |
| 3 | 3 | 0.0 | −0.481 |
| 4–5 | 38 | 68.4 | +1.331 |
| 6–10 | 27 | 59.3 | +1.068 |
| 11–20 | 20 | **75.0** | **+2.049** |

The best single bucket is **day-n 11–20** at +2.049R/trade — breadth confirmation is a real signal. But the daily-gate (rule 9) already captures this by surviving the first-3 confirmation pass. An explicit breadth-count filter would just be a weaker proxy for the same thing.

## §6 — Conclusion: Q22 is closed, no policy change

1. The low-n (< 4 triggers) passthrough is a non-issue at full PLAYBOOK stack depth. Rule 7 (skip-1st) mechanically excludes the weak cell (singletons).
2. The n=2 bucket is healthy (+1.25R/trade) and correctly survives both gates.
3. Adding an explicit `day_n ≥ 2` filter to the PLAYBOOK is redundant — the existing stack does the work.
4. For traders running a shallower stack (1–6 or 1–8), adding `day_n ≥ 2` as a nice-to-have is worth +0.17R/trade at a 19% n cost. Not compelling enough to promote to the canonical rule set.

**Policy unchanged.** Q22 was the last open monitoring item from the 12-note SPT research arc.

## §7 — What this closes and leaves open

- Closes Q22 (low-n day gate behavior) — resolved mechanically by rule 7.
- Remaining open items reduce to: Q1 (scanner schema), Q3 (next-day follow-through — needs more data), Q5 (cross-asset — blocked on multi-asset expansion), Q12 (time-to-target glidepath — documented, no rule), Q18 (2026-04-14 partial — closed in practice by Q21/rule 9).

## Related

- [[small-pullback-trend-PLAYBOOK]] — no rule changes; this note just confirms rule 7 does the expected filtering on singleton days.
- [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] §1 — where rule 7 (skip-1st) was derived.
- [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] — introduces rule 9 (daily-confirmation gate) and first raised Q22.
- [[small-pullback-trend-INDEX]] — full research arc index.
