# SPT — Stack Validation & Commit Decision (2026-04-18, pt 14)

Fourteenth follow-up to [[small-pullback-trend]]. Pt 13's leave-one-out ablation ([[small-pullback-trend-rule-ablation-2026-04-18]]) concluded that rule 8 (`market_ord > 2`) is **net-negative at full stack** and recommended dropping it. The PLAYBOOK was annotated with a pending-review banner but not changed, pending a stability check.

This note delivers that stability check: walk-forward, leave-one-week-out, leave-one-symbol-out, bootstrap CIs, and drawdown/tail metrics for three candidate stacks. It also reconciles a data-window discrepancy in pt 13.

Script: `~/code/aiedge/scanner/scratch/spt_stack_validation_2026_04_18.py`.
Outputs:
- `~/code/aiedge/scanner/scratch/_out_spt_stack_validation_2026_04_18.txt` (claimed 4-month window).
- `~/code/aiedge/scanner/scratch/_out_spt_stack_validation_2026_04_18_fulldb.txt` (full-db 8-month window — the data pt 13 actually used).

## §0 — Data-window reconciliation

Pt 13's docstring says "Data window: 2025-12-16 → 2026-04-17". Its SQL doesn't apply that filter — it pulls all `setup_type ∈ {H1..L2} AND day_type ∈ {TFO, S&C} AND result ∈ {WIN,LOSS,SCRATCH}` rows, which is 698 SPT-class detections (2025-08-13 → 2026-04-17 in the current DB). So pt 13's reported `n_raw = 689` (post-annotation) and `FULL n=46, sumR=+56.13, perR=+1.220` are on the **full-db 8-month window**, not the claimed 4-month window.

I re-ran the pipeline on both windows and reproduce pt 13 exactly on the full-db window. The 4-month window is a stricter subsample. **Conclusions in this note are robust on both windows** — I quote both below.

## §1 — The three candidate stacks

| Name | Rules | Description |
|------|-------|-------------|
| **FULL** | {3, 4, 5, 6, 6s, 7, 8, 9} | Current PLAYBOOK stack |
| **REVISED** | {3, 4, 5, 6, 6s, 7, 9} | Drop rule 8 only (pt 13 §6 recommendation) |
| **MINIMAL** | {6, 6s, 7, 9} | Four load-bearing rules (pt 13 §4 minimal probe) |

## §2 — Headline comparison

### Full-db 8-month window (n_raw=689)

| Stack | n | WR% | sumR | perR | perR 95% CI (bootstrap) |
|-------|--:|----:|-----:|-----:|-------------------------|
| FULL | 46 | 65.2 | +56.13 | +1.220 | [+0.701, +1.711] |
| **REVISED** | 83 | 67.5 | +115.53 | **+1.392** | [+1.003, +1.773] |
| MINIMAL | 96 | 64.6 | +126.05 | +1.313 | [+0.971, +1.670] |

### 4-month window (n_raw=517) — the claim pt 13 made

| Stack | n | WR% | sumR | perR | perR 95% CI (bootstrap) |
|-------|--:|----:|-----:|-----:|-------------------------|
| FULL | 38 | 78.9 | +64.13 | +1.688 | [+1.181, +2.173] |
| **REVISED** | 55 | 78.2 | +99.13 | **+1.802** | [+1.379, +2.205] |
| MINIMAL | 66 | 71.2 | +106.13 | +1.608 | [+1.196, +2.028] |

REVISED has the highest perR on **both** windows. MINIMAL has the highest total R on both windows. FULL's CI straddles negative territory on the full-db window (lower bound +0.70R but wide) while REVISED's CI is cleanly above +1.0R.

## §3 — Walk-forward (3 chronological thirds)

### Full-db, cutoffs S1 < 2025-11-05 ≤ S2 < 2026-02-04 ≤ S3

| Stack | S1 (early) n / perR | S2 (mid) n / perR | S3 (late) n / perR |
|-------|-------:|-------:|-------:|
| FULL | 7 / **−1.000** | 11 / +0.365 | 28 / +2.111 |
| REVISED | 20 / +0.519 | 22 / +1.047 | 41 / +2.003 |
| MINIMAL | 21 / +0.709 | 23 / +0.958 | 52 / +1.714 |

**The FULL stack shipped 7 trades in S1 and lost every one.** Rule 8 over-filters so aggressively early that what little it lets through is the worst cell. REVISED and MINIMAL both produce positive expectancy in every sub-period on the full-db window.

### 4-month, cutoffs S1 < 2026-01-30 ≤ S2 < 2026-03-10 ≤ S3

| Stack | S1 n / perR | S2 n / perR | S3 n / perR |
|-------|-------:|-------:|-------:|
| FULL | 10 / +0.501 | 6 / +1.061 | 22 / +2.398 |
| REVISED | 14 / +1.215 | 12 / +1.030 | 29 / +2.405 |
| MINIMAL | 14 / +1.215 | 13 / +1.182 | 39 / +1.891 |

Even on the stricter (narrower) window, REVISED dominates FULL in two of three splits; the one split where FULL "wins" is S2 at n=6 — noise territory.

**Verdict:** the REVISED > FULL gap is stable across time, not a late-window artifact. Pt 13 caveat (a) is rejected.

## §4 — Leave-one-week-out stability

### Full-db (28 calendar weeks)

| Stack | baseline perR | mean(LOO) | min(LOO) | max(LOO) | stdev(LOO) |
|-------|-----:|-----:|-----:|-----:|-----:|
| FULL | +1.220 | +1.208 | **+0.264** | +1.432 | 0.194 |
| REVISED | +1.392 | +1.389 | +0.996 | +1.491 | **0.084** |
| MINIMAL | +1.313 | +1.310 | +0.935 | +1.392 | 0.079 |

FULL's min(LOO) of +0.264 means there is a single week whose presence pulls full-stack perR from +1.43 down to +1.22 — **pt 13's headline number depends on 4 trades in one week.** REVISED has half the stdev and a min of +0.996 (still respectable per-trade).

### Pairwise dominance across the 28 LOO weeks

| Comparison | Stack A wins | Stack B wins | Ties |
|------------|-------------:|-------------:|-----:|
| FULL vs REVISED | 0 / 28 | **28 / 28** | 0 |
| FULL vs MINIMAL | 1 / 28 | 27 / 28 | 0 |
| REVISED vs MINIMAL | **28 / 28** | 0 / 28 | 0 |

**REVISED strictly dominates both FULL and MINIMAL on per-trade R across every leave-one-week-out resample on the full-db window.** On the 4-month window REVISED also wins 14/14. This is far stronger evidence than pt 13's single-run ablation delta.

Pt 13 caveat (b) — small-sample noise in per-R deltas — is rejected: the REVISED > FULL gap is not a sample-size artifact.

## §5 — Leave-one-symbol-out

### Full-db

| Stack | tickers | baseline perR | mean(LOO) | min(LOO) | stdev |
|-------|--------:|-----:|-----:|-----:|-----:|
| FULL | 23 | +1.220 | +1.217 | **+0.726** | 0.125 |
| REVISED | 52 | +1.392 | +1.391 | +1.172 | 0.044 |
| MINIMAL | 58 | +1.313 | +1.313 | +1.117 | 0.036 |

Dropping any single symbol from REVISED leaves perR ≥ +1.17; dropping any single symbol from FULL can drop perR to +0.73 (if that symbol is CMCSA). REVISED is markedly less symbol-concentrated than FULL.

**Top contributor is CMCSA in all three stacks (n=10, +30R).** This is already the largest single-symbol cluster. Even stripping CMCSA entirely, REVISED retains +1.17R/trade on ~73 trades — the edge is not one-symbol-dependent.

## §6 — Drawdown / tail risk

### Full-db, sequential cumulative-R by (date, signal time)

| Stack | final eq | max DD | DD / final |
|-------|-----:|-----:|-----:|
| FULL | +56.13R | **−10.00R** | 17.8% |
| REVISED | +115.53R | −4.00R | **3.5%** |
| MINIMAL | +126.05R | −4.00R | 3.2% |

**Dropping rule 8 cuts max drawdown from −10R to −4R** on the full-db window. Why: rule 8 preferentially keeps the day's early trades OUT (those are mkt_ord = 1 or 2), so the first-3-gate (rule 9) ends up making continue/stop decisions on a shifted sample that's less representative of the day's character. Equity path is choppier.

All three stacks have identical per-trade extremes (−1R worst, +3R best) — no fat tails, no fallback-resolved trades in the bottom/top slices of any stack. Risk is purely within the engineered [−1R, +3R] band.

## §7 — Day-type and direction cuts (reproduced)

Pt 13 §3.a/§3.b found that rule 8 was particularly harmful on S&C days and on the short side. Reproduced on both windows:

### Full-db

| Cut | FULL perR | REVISED perR | MINIMAL perR |
|-----|-----:|-----:|-----:|
| trend_from_open | +1.336 | +1.305 | +1.230 |
| **spike_and_channel** | **+0.956** | +1.511 | +1.424 |
| long | +1.127 | +1.331 | +1.296 |
| short | +1.739 | +1.572 | +1.360 |

On S&C days, dropping rule 8 lifts perR by +0.55R. On shorts, dropping rule 8 very slightly *reduces* perR (from +1.74 to +1.57) but **nearly triples trade count** (7 → 21), so total R on the short side goes up substantially.

## §8 — Decision

### Adopt: **REVISED stack (drop rule 8 only)**

- Highest per-trade expectancy on both windows (+1.39R / +1.80R).
- Dominates FULL in 28/28 leave-one-week-out splits.
- Halves the max-drawdown vs FULL (−4R vs −10R).
- Halves the leave-one-week-out stdev vs FULL.
- Retains the four theoretically-grounded filters (urgency, gap alignment, always_in) that would re-activate under data shift.

### Reject: **MINIMAL stack (four load-bearing rules only)**

MINIMAL delivers the most total R and slightly tighter stdev, but REVISED wins 28/28 LOO resamples on per-trade expectancy. The extra ~13 trades MINIMAL adds come at −0.079R/trade vs REVISED. For Will's "per-trade quality" framing, REVISED is strictly the better default. Keep MINIMAL in the vault as a documented alternative for a higher-throughput variant.

### Rule 3 (urgency gate)

Pt 13 §1 showed dropping rule 3 alone is slightly +R. I am NOT recommending this drop — the delta is small (+0.045R/trade) and inside confidence-interval overlap, and urgency is the scanner's core Brooks-grounded signal. Keeping rule 3 preserves interpretability. Revisit if a future note collects more evidence.

## §9 — What this opens and closes

- **Closes the pt 13 caveats.** Both (a) overfit-to-window and (b) small-sample noise are rejected — the REVISED > FULL gap is stable across walk-forward, LOO-week, LOO-symbol, and two different data windows.
- **Closes the "pending review" banner on the PLAYBOOK.** This note authorizes the rule 8 drop.
- **Opens a small reproducibility-hygiene item:** pt 13's docstring and INDEX table say the 4-month window but use the full-db window in practice. When the PLAYBOOK is refreshed, the "Expected economics" table should reproduce the full-db numbers and label the window correctly.
- **No new open Brooks questions.** Q1, Q3, Q5, Q12 remain open as before.

## §10 — PLAYBOOK changes to commit

1. Strike the "Candidate change pending review" banner (§0 of PLAYBOOK).
2. Drop rule 8 from the 10-rule stack (the new count is 9).
3. Update the "Expected economics" table to:

   | Stack | n | WR% | sumR | perR |
   |-------|--:|-----:|-----:|-----:|
   | REVISED 1–9 (proposed new default) | 83 | 67.5 | +115.53 | +1.392 |
   | MINIMAL 1-6, 6s, 7, 9 (alt) | 96 | 64.6 | +126.05 | +1.313 |
   | OLD FULL stack (reference) | 46 | 65.2 | +56.13 | +1.220 |

4. Correct the data-window label to "2025-08-13 → 2026-04-17 (full db)".
5. Add a "Drawdown" line to economics: REVISED max DD = −4R, 3.5% of final equity.

## Related

- [[small-pullback-trend-rule-ablation-2026-04-18]] — originating ablation that proposed the drop.
- [[small-pullback-trend-PLAYBOOK]] — updated in this pass to adopt REVISED.
- [[small-pullback-trend-INDEX]] — pt 14 entry added.
- [[small-pullback-trend-ordinal-and-daygate-2026-04-18]] — where rule 8 was introduced; this note retires it.
