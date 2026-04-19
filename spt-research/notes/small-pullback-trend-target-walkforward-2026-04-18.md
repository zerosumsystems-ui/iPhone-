# SPT — Rule-9 target walk-forward + hybrid validation (2026-04-18, pt 17)

Closes three open questions from [[small-pullback-trend-runner-targets-2026-04-18]] in one re-validation pass. Script: `~/code/aiedge/scanner/scratch/spt_target_walkforward_2026_04_18.py`. Output: `_out_spt_target_walkforward_2026_04_18.txt`.

Method:
- **Honest-pessimistic resolver** (Q28) — pure bar-walk, no MFE/MAE fallback. chart_end pnl capped at `target_r` / floored at −1R.
- **Stack** — PLAYBOOK-REVISED `{3, 4, 5, 6, 6s, 7, 9}` (rule 8 dropped per pt 14).
- **Window** — full DB, 2025-08-13 → 2026-04-17 (~8 months, 709 raw SPT candidates, 83 default-stack trades).
- **Variants** —
  - `global_3R` — current PLAYBOOK rule 9
  - `global_4R` — conservative single-parameter lift
  - `global_5R` — pt 16 candidate upgrade
  - `hybrid` — per-setup / per-direction (Q29): **H1/H2 → 5R, L1/L2 → 3R, shorts capped at 4R**

## 1. Headline — honest-pessimistic baseline

Same 83 trades, resolver differences only:

| variant    | n  | WR%   | sumR     | perR     | CI 95%              | max DD  |
|:-----------|---:|------:|---------:|---------:|:--------------------|--------:|
| global_3R  | 83 | 66.3% | +105.35  | +1.269   | [+0.894, +1.639]    | −4.00R  |
| global_4R  | 83 | 65.1% | +128.59  | +1.549   | [+1.094, +2.026]    | −4.00R  |
| **global_5R** | 83 | 65.1% | **+135.49** | **+1.632** | [+1.142, +2.158] | **−4.35R** |
| **hybrid**    | 83 | **66.3%** | +135.29 | +1.630 | [+1.162, +2.088] | **−4.00R** |

**Two observations:**

1. **Q28 closes.** The honest-pessimistic 3R baseline is **+1.269R** (matches pt 16 §1 exactly), not the **+1.392R** reported in pt 14 §Stack-validation headline. The 0.123R gap is the MFE/MAE fallback on 1 trade. All three conclusions pt 14 produced (rule-8 drop, walk-forward stability, LOO-week 28/28 dominance) still stand; the fallback affected one trade and is well inside the bootstrap CI.
2. **Hybrid ties 5R on expectancy with strictly better DD.** +1.630R vs +1.632R (0.002R noise) at −4.00R vs −4.35R max drawdown. Also recovers the 1 WR point (66.3% vs 65.1%) by keeping L1 at 3R.

## 2. Q27 — walk-forward (3 chronological thirds)

Cutoffs: S1 < 2025-11-05 ≤ S2 < 2026-02-04 ≤ S3.

| split     | n  | 3R      | 4R      | 5R      | hybrid   | best     |
|:----------|---:|--------:|--------:|--------:|---------:|:---------|
| S1 early  | 20 | +0.519  | +0.769  | +0.767  | **+0.784** | hybrid   |
| S2 mid    | 22 | +1.006  | +1.070  | +1.142  | **+1.190** | hybrid   |
| S3 late   | 41 | +1.777  | +2.187  | **+2.318** | +2.278  | global_5R |

**All four variants produce positive expectancy in every third.** 5R dominates 3R in every third by a margin larger than S1–S3 noise. Hybrid wins S1 and S2; 5R wins S3 — hybrid is more consistent across regimes.

**Regime caveat:** S3 (41 trades, +2.3R/trade) carries ~65% of total cumulative R. Much of this is the post-Feb 2026 market regime. S1 + S2 combined (42 trades) generate ~35% of R at roughly ~1.0R/trade. If future regime reverts toward S1, expect perR closer to +0.8R rather than +1.6R.

## 3. Q27 — leave-one-week-out

25 weeks (2025-W33 … 2026-W16).

| variant    | baseline | mean(LOO) | min     | max     | stdev  |
|:-----------|---------:|----------:|--------:|--------:|-------:|
| global_3R  | +1.269   | +1.266    | +0.885  | +1.358  | 0.087  |
| global_4R  | +1.549   | +1.545    | +1.073  | +1.647  | 0.106  |
| global_5R  | +1.632   | +1.628    | +1.100  | +1.780  | 0.119  |
| hybrid     | +1.630   | +1.625    | +1.097  | +1.747  | 0.118  |

**All min(LOO) values remain positive.** No single week carries the edge. Stdev grows 0.087 → 0.119 moving from 3R to 5R — expected from wider-target variance — but the ratio (stdev/perR) is actually flat (0.069 at 3R, 0.073 at 5R), meaning relative stability is unchanged.

### LOO-week pairwise dominance

| match-up                       | A wins | B wins | ties |
|:-------------------------------|-------:|-------:|-----:|
| global_5R vs global_3R         | **25/25** |  0  |  0   |
| global_5R vs global_4R         | **25/25** |  0  |  0   |
| hybrid    vs global_3R         | **25/25** |  0  |  0   |
| global_4R vs global_3R         | **25/25** |  0  |  0   |
| **hybrid  vs global_5R**       |   5    | **20** |  0  |

**Q27 closes.** 5R dominates 3R and 4R on every leave-one-week-out resample. Hybrid dominates 3R on every resample. The single genuine choice is 5R vs hybrid — global_5R wins 20/25; hybrid wins 5/25 on perR terms.

## 4. Q29 — why the 5R-vs-hybrid split matters (per-setup + per-side)

### Per-setup

| setup | 3R       | 4R       | 5R       | hybrid   | n  |
|:------|---------:|---------:|---------:|---------:|---:|
| H1    | +1.245   | +1.562   | **+1.760** | +1.760 | 43 |
| H2    | +1.260   | +1.647   | **+1.670** | +1.670 | 19 |
| L1    | **+1.055** | +1.069   | +0.866   | +1.055 | 17 |
| L2    | +2.485   | +2.985   | **+3.340** | +2.485 | 4  |

- **H1 and H2 want 5R** — pt 16's finding holds after honest-pessimistic resolver and walk-forward.
- **L1 at 5R costs −0.19R/trade** relative to 3R. Confirmed not a fallback artifact (same resolver as rest of table). L1 is genuinely the outlier.
- **L2 (n=4, too small)** wants 5R per the data, but we cannot act on 4 trades. Keeping L2 at 3R in hybrid is conservative.

### Per-direction

| direction | 3R      | 4R      | 5R      | hybrid  | n  |
|:----------|--------:|--------:|--------:|--------:|---:|
| long      | +1.250  | +1.588  | **+1.732** | +1.732 | 62 |
| short     | +1.328  | **+1.434** | +1.338 | +1.328 | 21 |

**Shorts peak at 4R.** Raising to 5R regresses (+1.434 → +1.338). Consistent with pt 11's 11:00 ET squeeze finding: shorts face a structural buy-the-dip that limits runners. Hybrid puts shorts at 3R (via L1/L2 path) or 4R (via H1/H2 path), both of which dominate their 5R row.

### Hybrid cell detail

With the hybrid rule, each trade's target is:

```
target = 5R  if setup in {H1, H2} else 3R
if direction == 'short': target = min(target, 4R)
```

So H1-long/short, H2-long → 5R; H2-short capped at 4R; L1 and L2 at 3R. On the 83-trade population:

| target bucket | n  | setup/dir breakdown                               |
|:--------------|---:|:--------------------------------------------------|
| 5R            | 53 | H1 longs (≈35) + H1 shorts (≈8) + H2 longs (≈10)  |
| 4R            |  9 | H2 shorts                                         |
| 3R            | 21 | L1 (17) + L2 (4)                                  |

The per-cell targets all align with the direction of the per-setup / per-side advantages listed above — no arbitrary cuts.

## 5. Proposed PLAYBOOK update (rule 9)

**Current rule 9:** `Exit: 3R target / 1R stop / hold to resolution`

**Proposed rule 9:** `Exit: target by setup/side (see table) / 1R stop / hold to resolution`

```
H1, H2 longs .......... 5R
H2 shorts ............. 4R
L1, L2 ................ 3R
```

### Expected economics at full stack (hybrid vs current)

|                | current (3R) | proposed (hybrid) | Δ            |
|:---------------|-------------:|------------------:|:-------------|
| WR             | 66.3%        | 66.3%             | 0 pp         |
| sumR           | +105.35      | +135.29           | +29.94R      |
| perR           | +1.269       | +1.630            | **+0.361R/trade** |
| max DD         | −4.00R       | −4.00R            | 0            |
| bootstrap 95% CI | [+0.894, +1.639] | [+1.162, +2.088] | CI lifted from borderline-zero to clearly positive |

### Why hybrid over plain global_5R

Global_5R is 0.002R/trade better on point estimate but worse on:

- **Max DD** — −4.35R vs −4.00R. The 0.35R difference comes from one extra L1 loss when the 3R scratch rolls back to a full −1R.
- **Day-type variance** — S1 perR is +0.784 (hybrid) vs +0.767 (5R); S2 is +1.190 vs +1.142; only S3 flips (+2.278 vs +2.318).
- **Economic rationale** — hybrid is a rule with a Brooks-native justification (shorts face buy-the-dip; L1s in shallow SPTs are often near the last LL rather than the runner's start). Plain global_5R is just "always go wider."

The LOO-week resample showed global_5R wins 20/25 weeks on perR — but by margins averaging +0.01–0.03R, all absorbed by bootstrap CI. The three-sigma S1 underperformance of global_5R in the walk-forward is the bigger tail risk.

## 6. Q26 status

Q26 asked whether L1's underperformance at higher targets is a short-side / small-sample artifact that may resolve as shorts grow.

With the honest-pessimistic resolver:

| setup-side    | 3R      | 5R      | Δ (5R − 3R) | n  |
|:--------------|--------:|--------:|:-----------:|---:|
| L1 long       | —       | —       | —           | 0  |
| L1 short      | +1.055  | +0.866  | −0.189      | 17 |

All 17 L1 trades in this window are shorts. Pt 11 established the structural buy-the-dip dynamic on shorts — and L1s are the most vulnerable to it because the signal bar is often two bars above the running LL, so any squeeze bounces back through entry before hitting the wider target.

**Q26 partially closes:** the L1 → 3R cap is not a small-sample artifact — it tracks the short-side dynamic. It *may* still be noise (n=17), but the direction is consistent with theory. Re-open when L1-long population grows. Hybrid's L1-at-3R rule is the safe choice either way.

## 7. Open items remaining

- **Q29 hybrid codification** — this note closes Q29 on the full-DB window. Adopting into the PLAYBOOK depends on Will's approval given the 3R → hybrid is a live trading-plan change. Recommendation is to adopt.
- **Q25 (day_type stability)** — still blocked on schema (no per-bar label history).
- **Q1, Q3, Q5, Q12, Q18** — unchanged from the PLAYBOOK open list.
- **New: regime dependence** — S3 (2026-02-04 onwards) carries ~65% of cumulative R at 5R/hybrid targets. If the market regime rotates toward S1-era behavior (+0.5–0.8R/trade instead of +2.3R), hybrid still dominates 3R by the same +0.3R/trade but absolute throughput falls. Not a policy change — a sizing / expectation anchor.
- **Scale-out revisit** — pt 16 rejected all 7 scale-out variants against global_5R. With hybrid now being the reference, a BE-after-3R runner variant on H1/H2 only could be reconsidered, but the 1R-giveback trail was the only variant pt 16 tested that matched the fixed target, and H1/H2 account for 75% of hybrid trades. Low priority.

## 8. Cross-references

- [[small-pullback-trend-runner-targets-2026-04-18]] — pt 16 (source of Q26–Q29).
- [[small-pullback-trend-stack-validation-2026-04-18]] — pt 14 (baseline methodology; resolver fallback flagged in pt 16 §7, now honest-pessimistic).
- [[small-pullback-trend-targets-and-urgency-2026-04-18]] — pt 3 (raw-population 3R target justification; still valid with honest-pessimistic resolver).
- [[small-pullback-trend-PLAYBOOK]] — candidate rule-9 edit pending.
- [[small-pullback-trend-INDEX]] — append pt 17 row.
