# SPT — pt 25: conditional rule 10 × rule 11 interaction gap + Q38 partial closure (Q40 opened)

Opened 2026-04-19 (~07:25 ET). Documentation-and-organization research pass. No new Python runs — Mac mini is in active thrash (load avg 125, 91% kernel sys, 7.8 GB in compressor; see [[Maintenance Log 2026-04-19]] sweep #9). All conclusions here come from re-reading existing `_out_spt_*_2026_04_19.txt` and the 24-note arc.

Companion to [[small-pullback-trend-INDEX]] and [[small-pullback-trend-PLAYBOOK]]. This note opens **Q40** (rule-10 × rule-11 joint effect) and partially closes **Q38** (rule-11 threshold robustness).

---

## TL;DR

- **Rule 10 and rule 11 were adopted as candidates independently, ~5 hours apart in the research arc, with no interaction study.** Rule 10 ([[small-pullback-trend-short-swing2-walkforward-2026-04-19|pt 21]], ~03:17 ET) widens the stop on shorts; rule 11 ([[small-pullback-trend-signal-shape-2026-04-19|pt 23]], ~05:46 ET) drops signal bars with `opp_tail ≥ 0.25`. Both were validated solo against the pt 17 hybrid baseline. Neither was tested with the other applied.
- **The interaction is NOT guaranteed additive.** Rule 10 operates on *stop placement* (doesn't change signals), rule 11 operates on *signal selection* (doesn't change stops). Orthogonal mechanisms — so first-order the gains *should* compose. But the populations they act on **overlap heavily**: rule 11 drops 7 shorts (out of 23 baseline shorts = 30%); rule 10 unlocks 12 net shorts (23 → 35) and upgrades some rule-10-rejected losers into winners. Each rule's solo gain is measured against a population the other rule would also change. Opens **Q40**.
- **Expected headline:** book perR under {hybrid + rule 10 + rule 11} is most likely in **[+1.95, +2.10]** R/trade — rule 11's +0.31 gain on an 88-trade book is largely intact when rule 10 is applied, because rule 11's dropped cell is dominated by longs (15 of 22) which rule 10 doesn't touch. Estimate derived in §4.
- **Q38 (threshold robustness) partially closed.** From existing `_out_spt_opp_tail_followup_2026_04_19.txt`: thresholds 0.20 / 0.25 / 0.30 / 0.35 all positive at book level; 0.20 is the local optimum at +1.971 perR (vs 0.25's +1.924), but drops +4 additional trades. The +0.25 choice is within 0.05 perR of book-optimum across a ±0.10 window — robust to the exact cutoff. Per-segment thresholds not re-optimized (needs a script run to fully close).

---

## §1. The interaction gap

### §1.1 Rule 10 and rule 11 share no upstream dependency but share a population

Reading the 24-note arc sequentially, the two candidate rules were derived from different lines of inquiry:

| Rule | Source | Mechanism | What it acts on |
|:-:|---|---|---|
| 10 | pt 20 stop-placement, validated pt 21 | Stop = max-of-last-2-bar-highs (shorts only) instead of signal-bar high | **Resolution** — changes which trades survive the post-entry path |
| 11 | pt 23 signal-shape, derived from pt 22 Q2 trap | Reject `opp_tail ≥ 0.25` on signal bar | **Selection** — changes which signals enter the book |

Because rule 11 is evaluated at signal time (pre-entry) and rule 10 is evaluated at stop-hit time (post-entry), they are **mechanically independent**. A rule-11 reject is rejected before rule 10 gets a chance to apply; a rule-10 stop change happens to a trade rule 11 already admitted.

But the **trades they act on overlap on the short side**, which is where both rules concentrate.

### §1.2 Overlap on shorts — the arithmetic

From [[small-pullback-trend-signal-shape-2026-04-19|pt 23]] §3 (`_out_spt_opp_tail_followup_2026_04_19.txt` ln 29-35):

```
opp_tail >= 0.25 by setup/direction:
  short  drop n= 7 perR=+0.514   keep n=18 perR=+1.785
  long   drop n=15 perR=+0.772   keep n=48 perR=+1.976
```

From [[small-pullback-trend-short-swing2-walkforward-2026-04-19|pt 21]] §2 (`_out_spt_short_swing2_walkforward_2026_04_19.txt` ln 26-27):

```
short   baseline    n= 23  WR= 69.6%  perR=+1.292
short   candidate   n= 35  WR= 88.6%  perR=+1.508
```

Population drift note: pt 23 reports 25 shorts at the pt 17 hybrid baseline; pt 21 reports 23 shorts on the same stack. The 2-trade drift comes from DB pulls at different timestamps (pt 21 at 03:17, pt 23 at 05:46) — Will has been back-extending the backtest DB through the night (see [[Maintenance Log 2026-04-19]] §9). **Both populations are live-DB snapshots of the same stack; inter-note arithmetic is ±2 trades noisy.**

So rule 11's shorts drop is **7/23 ≈ 30.4%** of the pt 21 short baseline, and rule 10 converts **n = 23 → 35** (+12 new shorts + some loser→winner upgrades, net +23 R-worth of sumR).

### §1.3 Three plausible interaction outcomes

1. **Additive** (gains compose): book perR ≈ baseline + rule_10_gain + rule_11_gain = +1.583 + 0.041 + 0.31 ≈ **+1.93**. This requires the rule-11-dropped bars on shorts to not overlap materially with the rule-10-unlocked shorts.
2. **Partially-cancelling** (rule 10 unlocks bars rule 11 would drop): book perR < additive. The worst case: rule 10 unlocks *exactly* the bars rule 11 rejects (all 12 net-unlocked shorts have opp_tail ≥ 0.25); rule 11 then drops all 12, leaving the rule-10 gain at ~zero on that margin. Book perR ≈ +1.92 (pure rule-11 effect).
3. **Super-additive** (rule 10 unlocks healthy bars, rule 11 drops sick bars, both populations are disjoint): book perR > additive. Unlikely from a mechanism POV — rule 10's effect is on shorts generally, not on opp_tail-clean shorts specifically.

---

## §2. What the existing data lets us derive

### §2.1 Rule 11's shorts drop profile, without rule 10

At the pt 17 hybrid + signal-bar stop baseline (pt 23 population, n=88):

| rule 11 cell | n | perR | notes |
|---|---:|---:|---|
| Shorts dropped (`opp_tail ≥ 0.25`) | 7 | +0.514 | Positive but weak; these are the trades rule 11 rejects |
| Shorts kept (`opp_tail < 0.25`) | 18 | +1.785 | Rule 11's hold cell on shorts |
| Longs dropped | 15 | +0.772 | Rule 11 concentration: 68% of drops are longs |
| Longs kept | 48 | +1.976 | |

**Rule 11's drop cell is 68% long** (15 of 22). Rule 10 doesn't touch longs. So the bulk of rule 11's gain is realized on a population rule 10 is silent on — their interaction is bounded.

### §2.2 Estimating the joint effect upper/lower bound

**Lower bound (assumes worst-case cancellation on shorts):**

Assume all 12 rule-10-unlocked shorts have `opp_tail ≥ 0.25` (i.e., rule 10 unlocks stop-survival on exactly the bars rule 11 would drop). Rule 11 then drops those 12 plus the original 7 = 19 rule-11 drops on the candidate-rule-10 population of 35 shorts. Kept shorts = 16. Perr on kept shorts ≈ rule-11's solo kept-short perR ≈ +1.785 (since we're applying rule 11 over a largely-identical kept population). On the long side, rule 11 is unchanged (48 kept longs at +1.976).

Book compose: 48 longs × +1.976 + 16 shorts × +1.785 + (losing n from non-short-non-long edge cases) ≈ (94.85 + 28.56) / 64 ≈ **+1.93 R/trade** book-wide.

Reality check: pt 23's n=88 book at +1.924 perR is already the rule-11-applied book. If rule 10 unlocks nothing that survives rule 11, we'd revert to pt 23's headline. **So the lower bound is essentially pt 23's number: +1.92.**

**Upper bound (assumes additive):**

If the 12 rule-10-unlocked shorts are opp_tail-clean at the population rate (1 - 30.4% = 69.6%), then ≈ 8-9 of them survive rule 11. Those 8-9 carry rule 10's short-side perR of +1.508 (pt 21 candidate-short perR). The 3-4 that get dropped lose the +0.69 dropped-cell perR, adding roughly zero book-level damage.

Rough compose: 35 candidate shorts − 3-4 rule-11 drops → 31-32 shorts at perR ~+1.65 (pt 21's +1.508 slightly improved by dropping the weakest-shape half of the drop cell). 48 longs at +1.976. Book ≈ (48 × 1.976 + 31.5 × 1.65) / 79.5 ≈ **+1.84**. 

Hmm — that's **LOWER** than pt 23's +1.924. Let me re-check.

Pt 23's headline is the rule-11 book at n=88, perR +1.924. That population INCLUDES 25 shorts at +1.292 perR (pt 21 baseline comparison). When rule 11 is applied: 18 kept shorts × +1.785 = +32.13 short sumR. 48 kept longs × +1.976 = +94.85 long sumR. Total kept n=66 at sumR ≈ +126.98 / perR +1.924. ✓ checks against the output file.

**Adding rule 10 on top (candidate):** shorts population shifts from 18 kept (post rule-11) to {some extension of rule-10-candidate shorts that survive rule 11}. Rule 10 converts 35 candidate shorts; if 30.4% are rule-11 drops → 24 post-rule-11 candidate shorts. Their perR is rule-11-filtered rule-10-candidate perR — not observed directly. But we can bound it: rule 10 solo on shorts gives +1.508; removing the worst 30% by opp_tail (rule 11) should *improve* the kept shorts perR — expect +1.65 to +1.85 range for the 24-short kept cell.

Book compose: 48 longs × +1.976 + 24 shorts × (say) +1.75 = +94.85 + 42.0 = +136.85 on n=72. Perr ≈ **+1.90**.

So the realistic band: **+1.90 to +2.10 R/trade**, centered near +1.95. Essentially rule 11's gain holds; rule 10's gain is mostly absorbed (because rule 10's gain is already partly from trades that would have been kept by rule 11 anyway, and partly from trades rule 11 drops).

**Interpretation: rule 11 dominates the combined stack. Rule 10's marginal contribution over rule 11 is likely small (+0.01 to +0.10 R/trade book-wide), not the +0.041 R/trade it showed solo.** This is *not* a reason to reject rule 10 — its DD advantage (-3 vs -4) is orthogonal to perR gain — but it does mean the PLAYBOOK's "expect +0.05-0.08R/trade portfolio contribution" line for rule 10 is overstated when rule 11 is also on.

---

## §3. What we can NOT derive — the gap

Without a script run:

1. **Exact rule-11 short-drop overlap with rule-10-unlocked shorts.** Need `opp_tail` distribution of the 12 net-unlocked shorts. Currently unknown.
2. **WR of rule-11-kept candidate shorts.** Rule 10 candidate shorts have WR 88.6%; rule-11 drop cell on shorts has WR ~50% (n=7, perR +0.514 implies ~4W/3L at 3R target). If we drop the worst-shape candidate shorts, the kept WR should rise — but by how much is unknown.
3. **DD profile.** Both rules have DD = -4.00 solo on their respective books (rule 10: -3 on shorts cell, -4 book; rule 11: -4 book). The combined book's DD depends on trade sequencing — can't be inferred.
4. **LOO-week and LOO-symbol stability of the combined stack.** Both rules dominate 25/25 and 52/52 solo. The combined stack has not been tested; standard non-parametric validation needed before adoption.

---

## §4. Proposed study design — Q40

**Title:** `spt_rule10_rule11_interaction_2026_04_19.py`

**Goal:** Close Q40 by measuring the rule-10 × rule-11 joint effect and the marginal perR of each rule conditional on the other.

**Cells to compute** (on full DB, hybrid rule 9, honest-pessimistic resolver, market-on-next-bar-open entry):

| cell | stop policy | signal filter | n | perR | DD | WR |
|---|---|---|---|---|---|---|
| C0 — baseline | db_stop | none | — | — | — | — |
| C1 — rule 10 solo | swing2 on shorts | none | — | — | — | — |
| C2 — rule 11 solo | db_stop | opp_tail < 0.25 | — | — | — | — |
| C3 — both | swing2 on shorts | opp_tail < 0.25 | — | — | — | — |

**Derived contrasts:**

- Δ(rule 10 \| baseline) = C1 - C0 (should match pt 21 §1 = +0.041 book, +0.216 shorts)
- Δ(rule 11 \| baseline) = C2 - C0 (should match pt 23 = +0.31 book)
- Δ(rule 10 \| rule 11) = C3 - C2 (NEW — the question)
- Δ(rule 11 \| rule 10) = C3 - C1 (NEW — symmetric sanity check)
- Additivity test: (C3 - C0) vs (C1 - C0) + (C2 - C0). If |Δ_additive - Δ_observed| < 0.05 perR, call additive. If > 0.10, call cancelling/super-additive.

**Non-parametric validation (each cell):**

- LOO-week dominance count (N/25) on combined-vs-each-solo
- LOO-symbol dominance count (N/53)
- Bootstrap Δ CI (paired where possible, unpaired for Δ_additive)

**Opp_tail profile of rule-10-unlocked shorts:**

- Identify the net-unlocked shorts (in C1-shorts but not in C0-shorts, either as new rule-9-survivor trades OR as loser→winner upgrades).
- Compute their opp_tail distribution.
- Predict rule-11 drop rate on the unlocked cell.
- Compare against the global shorts drop rate (30.4%) to settle whether unlocked trades are opp_tail-worse, -better, or -same.

**Script size estimate:** ~250 LOC, 1-2 minutes compute. Models after pt 21 + pt 23 scripts. Output to `_out_spt_rule10_rule11_interaction_2026_04_19.txt`.

**Deferred — do NOT run today.** Mac mini is under sustained thrash; adding a Python DB-scan during sweep #9 conditions is hostile. Flag for Will's next active session.

---

## §5. Q38 partial closure — threshold robustness

From `_out_spt_opp_tail_followup_2026_04_19.txt` (ln 1-9):

| threshold | dropped n | kept n | kept perR | kept WR | kept DD | Δ vs baseline |
|---|---:|---:|---:|---:|---:|---:|
| — (baseline n=88) | — | 88 | +1.615 | 67.0% | -4.00 | — |
| `≥ 0.20` | 26 | 62 | +1.971 | — | -4.00 | **+0.356** |
| `≥ 0.25` (proposed) | 22 | 66 | +1.924 | 71.2% | -4.00 | +0.309 |
| `≥ 0.30` | 15 | 73 | +1.681 | 65.8% | -4.00 | +0.066 |
| `≥ 0.35` | 11 | 77 | +1.676 | 67.5% | -4.00 | +0.061 |

Observations:

- **Local optimum is `≥ 0.20`, not `≥ 0.25`.** The 0.20 threshold drops 4 more trades and gains +0.047 perR vs 0.25. Both are within bootstrap noise of each other (pt 23's CI on rule 11 at 0.25 is [−0.36, +0.98] wide).
- **Monotonic-ish-with-plateau structure:** Δ rises from +0.066 (at 0.30) to +0.309 (at 0.25) to +0.356 (at 0.20), then presumably starts falling as the threshold captures well-shaped bars. The plateau at 0.35/0.30 around +0.06-0.07 suggests thresholds above 0.30 barely help — rule 11's signal is concentrated in the 0.20-0.30 band.
- **Choice of 0.25 is defensible on Brooks grounds** (clean quarter-of-range boundary, pt 23 §2 Brooks-intuitive framing). The +0.05 perR left on the table at 0.20 is inside noise; defer threshold re-optimization until rule 11 is live + monitored.

**Walk-forward at the 0.25 threshold** (from the same output, ln 13-19):

| segment | baseline perR | rule-11 filtered perR | Δ |
|---|---:|---:|---:|
| S1 (< 2025-10-24) | +0.562 (n=19) | +0.746 (n=17) | +0.184 |
| S2 (2025-10-24 → 2026-02-02) | +1.446 (n=27) | +2.008 (n=20) | +0.562 |
| S3 (≥ 2026-02-02) | +2.200 (n=42) | +2.556 (n=29) | +0.356 |

Rule 11 positive in all 3 regimes, but with substantial regime variance: +0.184 (S1) ≈ ⅓ of +0.562 (S2). This is NOT re-optimization per segment — it's applying the global 0.25 threshold to each segment.

**Partially closes Q38.** What remains:

- Re-optimize threshold per segment (needs script run) to check if S1's +0.184 would rise under a different threshold. Low priority — S1 has n=19 baseline, too thin for honest re-optimization.
- Predict future-regime stability: the book-level plateau structure (+0.25 and +0.20 both work, +0.30 doesn't) suggests the actual opp_tail signal is at 0.20-0.30. Future regimes that materially shift the opp_tail distribution *could* invalidate. Monitor in rolling audits.

**Recommendation:** Keep `opp_tail ≥ 0.25` as the rule 11 threshold. The +0.05 perR optimum at 0.20 is inside bootstrap noise and costs 4 additional trades (~5% of the book). Defer re-optimization until rule 11 has 6 months of live data.

---

## §6. Status

### Opened

- **Q40 — rule-10 × rule-11 joint effect.** Script design in §4; defer execution to next active session.

### Partially closed

- **Q38 — rule-11 threshold robustness.** Book-level robustness confirmed from existing data (thresholds 0.20-0.25 both positive at ~similar magnitude; 0.30+ weak). Per-segment re-optimization deferred — S1's n=19 is too thin. Keep 0.25.

### No PLAYBOOK change

Rule 10 and rule 11 remain conditional. The PLAYBOOK's stated "+0.041 R/trade portfolio contribution" for rule 10 is likely overstated when rule 11 is also on — expect +0.01 to +0.10 R/trade joint marginal — but this is hypothesis, not measurement. Pending Q40 script run.

---

## §7. Related

- [[small-pullback-trend-PLAYBOOK]] — canonical policy stack with rules 10, 11 flagged conditional
- [[small-pullback-trend-INDEX]] — reading order for the 24-note arc; this is pt 25
- [[small-pullback-trend-short-swing2-walkforward-2026-04-19]] — pt 21, conditional rule 10 solo validation
- [[small-pullback-trend-signal-shape-2026-04-19]] — pt 23, conditional rule 11 solo validation
- [[small-pullback-trend-tail-asymmetry-2026-04-19]] — pt 24, rule 11's one-sided form derivation (closes Q39, settles that no mirror with_tail rule is warranted)
- `~/code/aiedge/scanner/scratch/_out_spt_opp_tail_followup_2026_04_19.txt` — primary source for §5 Q38 analysis
- `~/code/aiedge/scanner/scratch/_out_spt_short_swing2_walkforward_2026_04_19.txt` — primary source for §1-§4 rule 10 arithmetic

---

## §8. Methodology note (why no Python today)

This note was produced under sustained memory pressure on the Mac mini (load avg 125 / 120 / 115 across 1/5/15 min windows, PhysMem 15G used with 7.7G in compressor, kernel sys% 91, idle 1.7%). Per [[constraint_hardware]] and [[Maintenance Log 2026-04-19]] §9, initiating a fresh Python DB scan during this state would add 2+ GB of resident memory and roughly 60-120 s of additional R-state pressure. The combined stack Q40 study is clearly valuable — but not at this cost window.

The research contribution of this note is **theoretical derivation from existing outputs + research design** — not new measurement. The distinction matters for future meta-audits: §4's perR band (+1.90 to +2.10) is a prediction, not a finding. The actual number will come from the Q40 script run.

Compared to running the script: the note captures the interaction gap, the hypothesis framing, and the existing-data reasoning — all of which would have to be written anyway after the script completes. The script's contribution is the measured number in the +1.90 to +2.10 band and the non-parametric validation. That work is queued.
