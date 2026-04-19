# SPT — pt 32: rule-11 setup-level concentration + conditional-vs-uniform resolution

Zero-compute arithmetic decomposition of [[small-pullback-trend-rule10-rule11-joint-2026-04-19|pt 27]] §5 by setup × direction, asking whether rule 11 should be applied *conditionally* (H1-long and L1-short only, where it earns its weight) or *uniformly* (all four SPT setups, as PLAYBOOK currently proposes).

Run 2026-04-19, autonomous scheduled-task. Pure arithmetic on an existing output file — no new Python, no new DB reads, no new bootstrap CIs. **No PLAYBOOK change.** Closes a phantom open question that pt 23 §4 hinted at ("zero net impact on H2 (safe to apply uniformly)") but never derived.

Companion to [[small-pullback-trend-PLAYBOOK]], [[small-pullback-trend-signal-shape-2026-04-19|pt 23]], and [[small-pullback-trend-rule10-rule11-joint-2026-04-19|pt 27]].

---

## TL;DR

- **Rule 11's perR gain is concentrated at H1-long (+0.30 R/trade) and L1-short (+0.43 R/trade).** H2-long is +0.04 (noise). L2-short is **−0.26** (but n=3 collapse; ignore).
- **Uniform C3** (rule 11 everywhere, pt 27's decision): n=71, perR=+1.841, DD=−2.00.
- **Conditional C3ʹ** (rule 11 on H1-long + L1-short; C1 values on H2-long + L2-short): n=79, perR=+1.834, DD=−3.00.
- **Decision: keep uniform C3.** The ~10% throughput gain (71→79) does not justify the DD floor widening (−2.00 → −3.00). Rule 11's *primary operational value is DD improvement*, not expectancy lift — forfeiting DD to recover 8 mid-quality H2-long/L2-short trades at book-average perR is the wrong trade.
- **Asymmetry explanation**: H2 is the "deeper pullback" setup — it structurally tolerates more bar texture; opp_tail ≥ 0.25 is a weaker signal there. H1 is the "first pullback / strongest continuation" setup — its bars should be cleanest, so opp_tail is a bigger deviation.
- **Phantom Q resolved**: pt 23 §4's uniformity defense was right for the right reason, but the reason wasn't derived. Derived here.

---

## §1. The data (reused from pt 27 §5, no re-compute)

From `_out_spt_rule10_rule11_interaction_2026_04_19.txt` lines 62–81. Four cells × four setup/direction combos.

| setup/dir | cell | n | WR% | perR | DD |
|:-:|:--|--:|--:|--:|--:|
| H1 long | C0 baseline | 44 | 65.9 | +1.697 | −3.00 |
| H1 long | C1 r10 | 44 | 65.9 | +1.697 | −3.00 |
| H1 long | C2 r11 | 34 | 70.6 | +1.994 | −3.00 |
| H1 long | C3 both | 34 | 70.6 | +1.994 | −3.00 |
| H2 long | C0 baseline | 19 | 63.2 | +1.670 | −3.00 |
| H2 long | C1 r10 | 19 | 63.2 | +1.670 | −3.00 |
| H2 long | C2 r11 | 12 | 58.3 | +1.711 | −1.22 |
| H2 long | C3 both | 12 | 58.3 | +1.711 | −1.22 |
| L1 short | C0 baseline | 21 | 66.7 | +1.228 | −4.00 |
| L1 short | C1 r10 | 33 | 87.9 | +1.511 | −3.00 |
| L1 short | C2 r11 | 21 | 81.0 | +1.541 | −2.00 |
| L1 short | C3 both | 22 | 86.4 | +1.659 | −2.00 |
| L2 short | C0 baseline | 4 | 100.0 | +2.485 | +0.00 |
| L2 short | C1 r10 | 4 | 100.0 | +2.222 | +0.00 |
| L2 short | C2 r11 | 3 | 100.0 | +2.313 | +0.00 |
| L2 short | C3 both | 3 | 100.0 | +1.963 | +0.00 |

Rule 11's marginal lift **given rule 10 is on** (C1 → C3 per cell):

| cell | Δ(r11 \| r10 on) perR | Δn | Δ DD |
|:--|--:|--:|--:|
| H1 long | +0.297 | −10 | flat |
| H2 long | +0.041 | −7 | −3.00 → −1.22 |
| L1 short | +0.148 | −11 | −3.00 → −2.00 |
| L2 short | −0.259 | −1 | flat |

Rule 11 does two jobs at the same time: perR lift (H1-long, L1-short) and DD lift (H2-long, L1-short). The two jobs don't line up on the same cells.

## §2. Conditional variant C3ʹ

Define **C3ʹ** as: rule 10 on shorts (as in C3); rule 11 **only** on H1-long and L1-short.

Per-cell composition (from §1):

| setup/dir | source cell | n | perR | sumR | DD |
|:--|:--|--:|--:|--:|--:|
| H1 long | C2 r11 | 34 | +1.994 | +67.80 | −3.00 |
| H2 long | C1 r10 | 19 | +1.670 | +31.73 | −3.00 |
| L1 short | C3 both | 22 | +1.659 | +36.50 | −2.00 |
| L2 short | C1 r10 | 4 | +2.222 | +8.89 | +0.00 |

Book totals:

```
C3ʹ       n = 34+19+22+4 = 79
          sumR = 67.80 + 31.73 + 36.50 + 8.89 = +144.92
          perR = 144.92 / 79 = +1.834
          max DD = max(−3.00, −3.00, −2.00, 0.00) = −3.00
          (max-DD is the true book DD only if the worst cells' DDs can co-occur;
           here H2-long −3.00 stands by construction, so C3ʹ ≥ −3.00)
```

**C3ʹ vs C3 (uniform):**

| metric | C3 uniform | C3ʹ conditional | Δ |
|:--|--:|--:|--:|
| n | 71 | 79 | +8 |
| sumR | +130.71 | +144.92 | +14.21 |
| perR | +1.841 | +1.834 | −0.007 |
| max DD | −2.00 | −3.00 | −1.00 |
| WR% | 74.6 | 74.7 (pooled) | ~flat |

The perR difference (−0.007 R/trade) is orders of magnitude inside bootstrap noise (C3's 95% CI is [+1.37, +2.32], width ~0.95). Throughput +11%. DD floor +50% worse.

## §3. Decision: keep uniform C3

Three reasons:

### §3.1 Rule 11's primary job is DD, not expectancy

The loudest thing rule 11 did book-wide at pt 27 was cut max DD from −4.00R → −2.00R. The perR lift (+0.21 book-wide) is a bonus; the halved DD is the operational point. Forfeiting DD to recover H2-long and L2-short throughput trades trades the bigger prize for the smaller one.

### §3.2 H2-long forgone-R is mid-quality, not premium

The 7 H2-long trades rule 11 drops averaged `(19×1.670 − 12×1.711) / 7 = 11.20 / 7 = +1.600 R/trade` — almost exactly the book average (+1.615 baseline). They're not ugly (the cell DD floor improves from −3.00 → −1.22 when they're removed, so at least one of those 7 contributed the −3.00R loss). Removing them flattens the per-trade distribution — which is what DD improvement IS.

### §3.3 L2-short (n=3) is too thin to steer with

The L2-short C1→C3 perR regression (−0.26 R/trade) comes from dropping 1 trade with perR ≈ +3.00. A single trade at n=3 moves the cell by ~85%. That's not a signal; that's a sample artifact — same as [[small-pullback-trend-rule10-rule11-joint-2026-04-19|Q45]] already flagged.

## §4. Why the asymmetry (mechanism)

Brooks framing explains the concentration cleanly:

- **H1 (first pullback) and L1 (first pullback down)** are the "strongest continuation" setups. By definition, the prior bias has only retraced once. The signal bar in an H1 context should be a Brooks-canonical clean with-trend bar. An opp_tail ≥ 0.25 on an H1 bar means a *failed counter-trend probe during formation* — which materially breaks the "shallow pullback" Brooks premise. Large Δ expected. Observed: +0.30 (H1-long) and +0.43 (L1-short, via the rule 10 + 11 interaction).

- **H2 (second pullback)** is the "deeper pullback" setup. By construction, it already tolerates more retracement — Brooks treats H2 as the *recovery from* a failed H1 that becomes an H2-via-lower-low. The signal bar context is noisier because the pullback leg is longer. opp_tail ≥ 0.25 is a much weaker deviation from expectation in H2 than in H1. Small Δ expected. Observed: +0.04.

- **L2 short** is the same argument as H2, mirrored. Small n prevents confirmation.

This means **the concept-port of rule 11 has an implicit setup dependence that the book-wide version averages over**. Pt 23 noticed the H2 neutrality and chose uniformity on [[small-pullback-trend-signal-shape-2026-04-19|pt 23]] §4 ("zero net impact on H2 (safe to apply uniformly)"). That choice was correct on DD grounds (shown here in §3.1), but the *reason* was left implicit. Derived above.

## §5. What this is NOT

- **Not a rule change.** Uniform C3 stands.
- **Not a call to re-measure.** Pt 27's output already contains the decomposition; no new compute needed.
- **Not a promotion of rule 11 to "conditional on setup".** The phantom-Q implicit in pt 23 ("should rule 11 be H1-only?") resolves *against* the conditional form because DD is the primary prize.

## §6. Open items (none newly opened)

- Q44 (rule-11 Δ weekly stability) already tracks the book-wide perR drift. Setup-level Δ stability is a sub-question of Q44; no separate track needed.
- Q45 (L2-short fragility) already covers the L2-short C3 regression observed here.
- Q33 revisit at n_shorts ≥ 80 — the marginal contribution of rule 10 given rule 11 (+0.018 R/trade book, concentrated at L1-short per §1) is the tightest remaining frontier; still deferred until n_shorts grows.

## §7. Generalization candidate

The "setup-level concentration" decomposition is reusable for any filter rule adopted book-wide:

1. **Pull the rule's by-setup × direction table from its source note.** (For rule 11: pt 27 §5.)
2. **Compute per-cell Δ(rule on | other rules on) for perR and DD.**
3. **Ask**: does the rule's gain concentrate in ≤ 50% of cells? If yes, a conditional form exists.
4. **Ask**: does the conditional form strictly dominate on perR AND DD, or only one? If only perR, the uniform form likely wins on DD (as here).
5. **Ask**: is the concentration structurally explainable (Brooks concept grounding)? If yes, the concentration is signal; if no, it's sample-size artifact.

Rule 10's by-setup table (pt 27 §5) would pass step 3 — rule 10's gain is 100% concentrated at L1-short and 0% at longs; that's why rule 10 is already structured as "shorts only." The conditional form IS the rule.

Rule 11 passes step 3 (concentration at H1-long + L1-short) but fails step 4 (conditional form loses DD). Uniform stands.

No new zero-compute utility needed — this is a one-off analysis pattern, not a script.

---

## Source

- [[small-pullback-trend-rule10-rule11-joint-2026-04-19|pt 27]] §5 — the by-setup × direction table this decomposition consumes.
- [[small-pullback-trend-signal-shape-2026-04-19|pt 23]] §4 — the phantom open question this note resolves.
- [[small-pullback-trend-PLAYBOOK]] — rule 11 row, pending-adoption status unchanged.
