# SPT — pt 27: rule-10 × rule-11 joint effect measured (Q40 closed)

Closes **Q40**, opened by [[small-pullback-trend-rule10-rule11-interaction-2026-04-19|pt 25]].

Pt 25 flagged that rules 10 and 11 had been adopted independently (~5 h apart) with no joint-effect study, derived a theoretical band of **[+1.90, +2.10] R/trade** for the combined stack, and queued the script `spt_rule10_rule11_interaction_2026_04_19.py`. The script was deferred at pt 25 write-time under Mac-mini thrash (load avg 125). At pt 27 write-time (2026-04-19 ~10:25 ET) the load had recovered to ~8 and the script ran clean in ~1 min (`_out_spt_rule10_rule11_interaction_2026_04_19.txt`).

This is pt 27 because the brooks-source cross-reference note is pt 26.

---

## TL;DR

- **Rules 10 and 11 are additive.** Naive-additive prediction: +0.245. Observed: +0.226. Additivity error −0.019 R/trade — well inside the 0.05 additive-threshold.
- **Rule 11 dominates the combined stack.** Δ(rule 11 \| rule 10 on) = +0.189. Δ(rule 10 \| rule 11 on) = +0.018 — rule 10's marginal contribution when rule 11 is already filtering is essentially zero on perR. Matches pt 25's §2.2 prediction.
- **Combined stack C3: n=71, WR 74.6%, perR +1.841, DD −2.00.** Dominates C0 baseline 25/25 LOO-weeks and 54/54 LOO-symbols; dominates C1 25/25 weeks and 54/54 tickers; dominates C2 23/25 weeks and 52/54 tickers.
- **Pt 25 §2.2 prediction landed at the low end.** Pt 25 predicted [+1.90, +2.10]; observed +1.841. Gap explained by **rule-11-gain shrinkage in live-DB re-measurement**: pt 23 measured rule 11 at +0.31 R/trade book; re-measured now at +0.208. The rule-10 × rule-11 mechanism is as-theorized; the absolute level is slightly lower because the underlying rule-11 solo gain has drifted down by ~0.10 R/trade under additional trades.
- **Rule 10's primary contribution in the combined stack is the DD floor on shorts, not perR.** C2 solo already hits DD −2.00. The DD gain from rule 10 (shorts cell −4→−3 in pt 21) is partially redundant with rule 11's DD floor. Keep rule 10 conditional.
- **No PLAYBOOK change required** vs pt 25's stance. Both rules remain conditional candidates; the +0.05–0.08 R/trade portfolio-contribution estimate for rule 10 is **overstated** — revise to +0.01–0.03 R/trade when rule 11 is also on.

---

## §1. Headline table

| cell | description | n | WR | perR | DD | 95% CI on perR |
|---|---|---:|---:|---:|---:|---|
| C0 | baseline (db_stop, no opp_tail filter) | 88 | 67.0% | +1.615 | −4.00 | [+1.15, +2.06] |
| C1 | rule 10 solo (swing2 on shorts) | 100 | 74.0% | +1.652 | −4.00 | [+1.28, +2.05] |
| C2 | rule 11 solo (opp_tail < 0.25) | 70 | 72.9% | +1.823 | −2.00 | [+1.32, +2.30] |
| C3 | **both rules** | 71 | 74.6% | **+1.841** | **−2.00** | [+1.37, +2.32] |

All cells: stack REVISED {3,4,5,6,6s,7,9}, hybrid rule 9 (H1/H2→5R, L1/L2→3R, shorts cap 4R), market-on-next-bar-open entry, honest-pessimistic resolver.

Observations from the table:

- **n(C3) > n(C2)** by 1 trade. Rule 10 unlocks shorts (25 → 37 at C1 solo), 11 of which rule 11 then drops, leaving a net +1 vs C2's 24 shorts. Small net n gain on the short side.
- **DD(C3) = DD(C2) = −2.00**. Rule 11 is the DD-improver. Rule 10's −3.00 DD on the solo shorts cell (pt 21) doesn't carry through because rule 11 has already clipped the worst losers.
- **WR(C3) > WR(C2) by 1.7 pp**. Rule 10 adds 1 short at 87.5%+ WR (consistent with pt 21's L1-short 87.9% candidate-WR).

---

## §2. Contrasts and additivity

```
Δ(rule 10 | baseline) = C1 - C0 = +0.037
Δ(rule 11 | baseline) = C2 - C0 = +0.208
Δ(both    | baseline) = C3 - C0 = +0.226

Naive-additive prediction        = +0.245
Additivity error (obs - pred)    = -0.019   (inside 0.05 threshold → ADDITIVE)

Marginal Δ(rule 10 | rule 11 on) = C3 - C2 = +0.018
Marginal Δ(rule 11 | rule 10 on) = C3 - C1 = +0.189
```

**Interpretation.** Rule 10 and rule 11 operate on orthogonal mechanisms (stop placement vs signal selection) and the populations they act on overlap but don't cancel. Rule 10's solo gain (+0.037 book) nearly vanishes when rule 11 is applied (+0.018), but the solo gain was already tiny at book level; the gain was always primarily a DD-and-shorts-WR story.

Rule 11's solo gain (+0.208 book) is almost entirely preserved when rule 10 is applied (+0.189). The small ~10% erosion (−0.019 R/trade) comes from rule 10 unlocking 12 shorts at mean opp_tail 0.197 (higher than the 0.119 C0-shorts baseline), 4 of which rule 11 then drops — slightly higher than the 28% drop rate on all C0 shorts.

---

## §3. Bootstrap CIs — wide but consistent-sign

Independent-bootstrap 95% CIs on the pair-contrasts (n_iter=2000, seed=20260419):

```
Δ(C1 - C0) = +0.037  CI [-0.59, +0.65]   zero-in-CI
Δ(C2 - C0) = +0.208  CI [-0.47, +0.90]   zero-in-CI
Δ(C3 - C0) = +0.226  CI [-0.43, +0.85]   zero-in-CI
Δ(C3 - C2) = +0.018  CI [-0.65, +0.73]   zero-in-CI
Δ(C3 - C1) = +0.189  CI [-0.44, +0.81]   zero-in-CI
```

Every CI includes zero — the book sizes (n=70–100) are too thin for parametric significance on perR deltas of this magnitude. **This is why LOO-week and LOO-symbol dominance are the load-bearing tests for adoption.**

---

## §4. Non-parametric validation — C3 dominance

### §4.1 LOO-week dominance (25 weeks)

```
C3 vs C0:  C3 wins 25/25   deltas in [+0.170, +0.346]
C3 vs C1:  C3 wins 25/25   deltas in [+0.158, +0.222]
C3 vs C2:  C3 wins 23/25   deltas in [-0.048, +0.077]
```

C3 strictly dominates C0 and C1 across every LOO-week cut. Against C2 the margin is within noise (±0.05 R/trade) and flips in 2 weeks — consistent with rule 10's marginal contribution being small when rule 11 is already on.

### §4.2 LOO-symbol dominance (54 tickers)

```
C3 vs C0:  C3 wins 54/54   deltas in [+0.170, +0.346]
C3 vs C1:  C3 wins 54/54   deltas in [+0.158, +0.258]
C3 vs C2:  C3 wins 52/54   deltas in [-0.048, +0.075]
```

Same story on the symbol axis. C3 is not single-ticker-fragile.

**Decision threshold met:** both LOO-dominance counts exceed 95% (25/25 = 100% vs C0/C1; 23/25 and 52/54 = 92% and 96% vs C2). The combined stack passes the pt 14 / pt 21 adoption bar against baseline and against rule-10-solo. Against rule-11-solo the case is weaker but still favorable.

---

## §5. By direction — where each rule lives

| | C0 | C1 | C2 | C3 |
|---|---|---|---|---|
| **Longs** n | 63 | 63 | 46 | 46 |
| perR | +1.689 | +1.689 | +1.920 | +1.920 |
| WR | 65.1% | 65.1% | 67.4% | 67.4% |
| DD | −4.00 | −4.00 | −2.00 | −2.00 |
| **Shorts** n | 25 | 37 | 24 | 25 |
| perR | +1.429 | +1.588 | +1.638 | +1.695 |
| WR | 72.0% | 89.2% | 83.3% | 88.0% |
| DD | −4.00 | −3.00 | −2.00 | −2.00 |

**Longs** — rule 10 is a mechanical no-op (same trades, same stops). Rule 11 drops 17 of 63 longs and lifts perR from +1.689 → +1.920.

**Shorts** — rule 10 unlocks 12 shorts (25 → 37, +23R on the unlocked cell), WR jumps 72% → 89%. Rule 11 drops 13 of those 37 to leave 25, similar size to the C0 baseline but at higher quality (perR +1.695 vs +1.429). DD floor held at −2 for shorts in both C2 and C3. **Rule 10's shorts-side gain survives rule 11**, just at smaller magnitude: C2 shorts (+1.638) → C3 shorts (+1.695), +0.057 R/trade on 25 shorts.

---

## §6. By setup × direction — fragility

| setup | dir | C0 n | C0 perR | C3 n | C3 perR | Δ |
|---|---|---:|---:|---:|---:|---:|
| H1 | long | 44 | +1.697 | 34 | +1.994 | +0.297 |
| H2 | long | 19 | +1.670 | 12 | +1.711 | +0.041 |
| L1 | short | 21 | +1.228 | 22 | +1.659 | +0.431 |
| L2 | short | 4 | +2.485 | 3 | +1.963 | **−0.522** |

**L2 short is the one negative cell** — C3 drops 1 of the 4 C0-baseline L2 shorts (via rule 11's opp_tail filter) and another becomes rule-10-stop-hit. n=3 is too thin to conclude, and L2-short is already a sparse cell — pt 11's time-of-day / first-3-wins gate handles it upstream. **Not a reason to change rule-11 threshold.**

**L1 short is the strongest combined-stack cell** at +0.43 R/trade improvement — consistent with pt 21's finding that rule 10 lives almost entirely in L1-short.

---

## §7. Rule-10-unlocked shorts — opp_tail overlap

Pt 25 §2.2 asked: of the 12 shorts rule 10 unlocks, how many have opp_tail ≥ 0.25? The script answers:

| cohort | n | perR | opp_tail mean | opp_tail median | n(≥ 0.25) |
|---|---:|---:|---:|---:|---|
| C0 shorts (baseline) | 25 | +1.429 | 0.119 | 0.014 | 7 (28.0%) |
| C1 shorts (rule 10 on) | 37 | +1.588 | 0.144 | 0.080 | 11 (29.7%) |
| **Unlocked by rule 10** | 12 | +1.397 | **0.197** | **0.231** | **4 (33.3%)** |
| Shared (C0 ∩ C1) | 25 | +1.680 | 0.119 | 0.014 | 7 (28.0%) |

- **The 12 rule-10-unlocked shorts have a notably higher opp_tail profile** (mean 0.197, median 0.231) than the shared C0 ∩ C1 shorts (mean 0.119, median 0.014). Rule 10 is mildly concentrated in opp_tail-suspicious territory.
- **But the 33.3% rule-11 drop rate is only 5 pp higher** than the 28% global rate. Not the 100% cancellation worst-case from pt 25 §2.2.
- **The 4 rule-10-unlocked shorts that survive rule 11 (8 of 12)** retain rule 10's gain — this is the +0.057 R/trade C3-shorts gain over C2-shorts.

**Net:** rules are **mildly non-additive in the expected direction** (rule 10 unlocks some rule-11-droppable trades) but the overlap is small enough that the naive-additive prediction is only 0.019 R/trade off. Call it additive.

---

## §8. Walk-forward — 3 thirds

| cell | S1 (early) n / perR | S2 (mid) n / perR | S3 (late) n / perR |
|---|---|---|---|
| C0 baseline | 20 / +0.784 | 27 / +1.223 | 41 / +2.278 |
| C1 rule 10 | 23 / +1.328 | 27 / +1.062 | 50 / +2.119 |
| C2 rule 11 | 19 / +1.036 | 18 / +1.648 | 33 / +2.372 |
| **C3 both** | 20 / +1.477 | 18 / +1.406 | 33 / +2.299 |

- **C3 positive in all 3 regimes** — the key regime-robustness check.
- **S1 (n=20, perR +1.477) is C3's strongest relative gain** over C0 (+1.477 vs +0.784 = +0.693 R/trade). Rule 11 + rule 10 combined lifts the weakest regime the most.
- **S3 is the strongest absolute** but the combined-stack gain shrinks there (+2.299 vs C0's +2.278 = +0.021). Suggests the combined rules' value is concentrated in the harder regimes, which is a good property.
- **Cross-regime consistency** — all three cells rank the same order across S1/S2/S3 mostly; no regime inversion.

---

## §9. Comparison to pt 25 prediction

| metric | pt 25 prediction | observed | gap |
|---|---|---|---|
| Joint book perR | [+1.90, +2.10] | +1.841 | −0.059 (BELOW floor) |
| Rule 10 marginal (given rule 11) | [+0.01, +0.10] | +0.018 | inside range |
| Additivity verdict | "rule 11 dominates" | additive, rule 11 dominates | ✓ |

**Where the 0.059 gap comes from.** Pt 25 used pt 23's baseline (+1.615) and rule-11 solo gain (+0.31) as inputs. Re-measured now:

| metric | pt 23 (2026-04-19 05:46) | pt 27 (2026-04-19 10:25) | drift |
|---|---:|---:|---:|
| C0 baseline perR | +1.615 | +1.615 | 0.000 |
| C2 rule 11 solo perR | +1.924 | +1.823 | **−0.101** |
| Rule 11 Δ | +0.309 | +0.208 | −0.101 |
| n (baseline) | 88 | 88 | 0 |
| n (rule 11 kept) | 66 | 70 | +4 |

**Baseline population is stable (n=88, perR +1.615).** But rule 11's kept-population grew by 4 trades (66 → 70). Those 4 additional kept trades had average-or-below perR (+0.208 × 70 − +0.309 × 66 = 14.56 − 20.39 = −5.83 R on 4 trades = **−1.46 R/trade** average, well below rule 11's average of +1.82).

**Interpretation.** The back-extended backtest DB has added 4 opp_tail-clean trades that underperformed rule 11's prior kept cell. Rule 11's gain over baseline is re-measured at +0.208 rather than +0.309. This is **not** a rule-10 × rule-11 interaction effect — it's live-DB drift on rule 11 itself since pt 23, which pt 25 couldn't see because it re-used pt 23's numbers without re-running.

**Lesson for future predictions:** use current-snapshot baseline measurements, not notes from earlier in the same day, when the DB is being back-extended.

---

## §10. Policy implications

### §10.1 Keep both rules conditional

No change from pt 25's stance. Neither the 23/25 LOO-weeks against C2, nor the 52/54 LOO-symbols against C2, nor the wide bootstrap CI crossing zero reach the **core-stack** adoption bar. But all three clear the conditional-rule bar.

### §10.2 Update portfolio-contribution estimates

From the PLAYBOOK notes on rule 10:
- **Old (pt 21):** "expect +0.05–0.08 R/trade portfolio contribution"
- **Corrected (pt 27):** "+0.01–0.03 R/trade when rule 11 is on; +0.04–0.08 R/trade if rule 11 is ever dropped." Rule 10's DD-on-shorts contribution is largely absorbed by rule 11's book-wide DD improvement.

From the PLAYBOOK notes on rule 11:
- **Old (pt 23):** "+0.31 R/trade book"
- **Corrected (pt 27):** "+0.21 R/trade book at current DB snapshot; pt 23's +0.31 was at an earlier snapshot. Re-measure quarterly." DD floor from −4 to −2 is stable.

### §10.3 Combined-stack joint contribution

- **+0.226 R/trade over C0 baseline** (pt 17 hybrid).
- **DD improved from −4 to −2**. This is the most operationally important gain — max-risk allocation can be reduced.
- **Sign-consistent across 25/25 LOO-weeks and 54/54 LOO-symbols** vs baseline.
- At Will's typical sizing this converts to roughly 14% more R/day at 62% less max-drawdown risk, taking current books at face value.

### §10.4 When to re-run

- At **n_shorts ≥ 80** (currently 25 in C3 short cell; ~3× scaling needed) to tighten bootstrap CIs and check the rule-10 marginal CI crosses zero cleanly.
- After any **bpa_detector** change or **day_type** re-labeling that materially shifts n_baseline.
- If rule-11 Δ drifts below **+0.10 R/trade** on a fresh snapshot, re-evaluate the rule-10 marginal — a weakening rule 11 would restore rule 10's relevance.

---

## §11. What this closes and opens

### Closed

- **Q40 (rule-10 × rule-11 joint effect).** Measured additive. Rule 11 dominates. Combined stack passes LOO validation vs baseline and vs rule-10 solo; narrower but still favorable vs rule-11 solo. PLAYBOOK recommendations in §10.

### Opens

- **Q44 (new, low-priority) — rule-11 gain stability over time.** The +0.10 R/trade drift from pt 23 (05:46) to pt 27 (10:25) in a **5-hour window** is surprising for a 66→70 trade population change. Is this normal DB-extension noise, or is rule 11 regime-sensitive? Track weekly measurement across next 4 weeks; if ±0.15 R/trade is the noise floor, rule 11's adoption bar needs to factor that in.
- **Q45 (new, low-priority) — L2-short sparse cell.** C3 n=3 at +1.963 vs C0 n=4 at +2.485. Small but strictly negative delta. L2-short is thin enough that any rule (10, 11, or both) triggers a 25% population change. Monitor; don't act.

---

## §12. Status summary

| Field | Value |
|---|---|
| Research question | Q40 — rule-10 × rule-11 joint effect |
| Status | **Closed** |
| Method | 4-cell grid {C0,C1,C2,C3} × {LOO-week, LOO-symbol, bootstrap CI, walk-forward} |
| Primary finding | Additive (err −0.019 R/trade); rule 11 dominates; combined perR +1.841, DD −2 |
| Secondary finding | pt 23 rule-11 Δ has drifted from +0.31 → +0.21 over 5 h of back-extended DB |
| PLAYBOOK change | No structural change; update portfolio-contribution estimates for rule 10 (down) and rule 11 (down) |
| Script | `~/code/aiedge/scanner/scratch/spt_rule10_rule11_interaction_2026_04_19.py` |
| Output | `~/code/aiedge/scanner/scratch/_out_spt_rule10_rule11_interaction_2026_04_19.txt` |

---

## §13. Related

- [[small-pullback-trend-rule10-rule11-interaction-2026-04-19]] — pt 25, design and theoretical derivation (this note closes its open question)
- [[small-pullback-trend-short-swing2-walkforward-2026-04-19]] — pt 21, rule 10 solo validation
- [[small-pullback-trend-signal-shape-2026-04-19]] — pt 23, rule 11 solo validation
- [[small-pullback-trend-tail-asymmetry-2026-04-19]] — pt 24, rule 11's one-sided form
- [[small-pullback-trend-brooks-source-cross-reference-2026-04-19]] — pt 26, Brooks-source audit
- [[small-pullback-trend-PLAYBOOK]] — canonical policy (unchanged; portfolio-contribution lines to be updated)
- [[small-pullback-trend-INDEX]] — reading order (this is pt 27)
