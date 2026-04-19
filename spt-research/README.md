# SPT research — phone briefing

Autonomous small-pullback-trend research notes, phone-readable. Long-form notes live in [`notes/`](notes/); phone-friendly PDFs in [`pdfs/`](pdfs/); the latest TL;DR is at the top of this README.

> 📚 **Full research archive — [ARCHIVE.md](ARCHIVE.md)** lists every SPT note ever written (32 notes in reading order + the PLAYBOOK + the INDEX). Canonical mirror at the aiedge-vault: [github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts](https://github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts).

**Last updated:** 2026-04-19 · pt 32

## Latest — pt 32: rule-11 setup-level concentration + conditional-vs-uniform resolution

**Run:** 2026-04-19, scheduled-task autonomous. Zero-compute arithmetic decomposition of pt 27's existing output — no new Python, no DB reads.

### Headline

Rule 11's +0.19 R/trade book-wide lift (given rule 10 is on) is **not spread evenly** across SPT setups. It concentrates at **H1-long (+0.30)** and **L1-short (+0.15)**, is essentially noise at **H2-long (+0.04)**, and mildly regresses on **L2-short (−0.26, but n=3 sample artifact)**.

Natural question: should rule 11 be **conditional** on setup (apply only to H1-long + L1-short, skip H2-long + L2-short)? We can compute the answer exactly from pt 27 §5 without rerunning anything.

**No PLAYBOOK change.** Uniform rule 11 stands. The conditional form gives **identical expectancy** (+1.834 vs +1.841 R/trade; well inside bootstrap noise) at **slightly more throughput** (+8 trades) but at **worse max DD** (−3.00 vs −2.00). Rule 11's primary operational value is DD improvement, not perR lift — forfeiting DD to recover book-average-quality H2-long trades is the wrong trade.

### The decomposition (pt 27 §5 reused)

Rule 11's marginal lift **given rule 10 is on** (C1 → C3 per cell):

| cell | n Δ | perR Δ | DD Δ |
|---|--:|--:|---|
| **H1 long** | 44 → 34 (−10) | +1.697 → **+1.994** (+0.297) | flat (−3.00) |
| H2 long | 19 → 12 (−7) | +1.670 → +1.711 (+0.041) | −3.00 → −1.22 |
| **L1 short** | 33 → 22 (−11) | +1.511 → **+1.659** (+0.148) | −3.00 → −2.00 |
| L2 short | 4 → 3 (−1) | +2.222 → +1.963 (**−0.259**) | flat (0.00) |

H1-long and L1-short are doing all the perR work. H2-long contributes to DD improvement (−3 → −1.22) but barely moves perR. L2-short is a sample-size artifact (n=3).

### Uniform C3 vs conditional C3ʹ

**C3ʹ** = rule 10 on shorts + rule 11 **only on H1-long + L1-short** (H2-long and L2-short use C1 values).

| metric | C3 uniform | C3ʹ conditional | Δ |
|---|--:|--:|--:|
| n | 71 | 79 | +8 (+11%) |
| sumR | +130.71 | +144.92 | +14.21 |
| **perR** | **+1.841** | **+1.834** | −0.007 (noise) |
| **max DD** | **−2.00** | **−3.00** | −1.00 (worse) |

Bootstrap 95% CI on C3 perR is [+1.37, +2.32] (width ~0.95 R/trade). The 0.007 R/trade conditional-vs-uniform delta is two orders of magnitude inside that noise floor. **perR is identical.** The only real difference is throughput vs DD — and DD is the prize.

### Why the concentration exists (Brooks mechanism)

- **H1 (first pullback)** and **L1 (first pullback down)** are the "strongest continuation" setups. By definition the prior bias has retraced only once. The signal bar should be a Brooks-canonical clean with-trend bar. An `opp_tail ≥ 0.25` (counter-trend wick ≥ 25% of bar range) on an H1 bar = failed counter-trend probe during formation = materially breaks the "shallow pullback" SPT premise. **Large Δ expected. Observed: +0.30 and +0.15.**
- **H2 (second pullback)** is the "deeper pullback" setup. By construction it already tolerates more retracement — Brooks treats H2 as the *recovery from* a failed H1. Signal bar context is noisier because the pullback leg is longer. `opp_tail ≥ 0.25` is a much weaker deviation from expectation in H2 than in H1. **Small Δ expected. Observed: +0.04.**
- **L2 short** mirrors H2-long; n=3 prevents confirmation.

So the concentration **is signal, not noise** — it has a Brooks-grounded structural explanation. Rule 11's book-wide version averages this concentration out; the conditional version would preserve it. Both forms work; uniform form wins on DD.

### What this closes

Pt 23 §4 said "zero net impact on H2 (safe to apply uniformly)" when rule 11 was first proposed. That choice was correct on DD grounds (derived here in pt 32) but the *reason* was left implicit. Now derived. The phantom open question "should rule 11 be setup-conditional?" resolves **against** the conditional form.

### Generalization

The "setup-level concentration" decomposition is reusable as a one-off analysis pattern for any future book-wide rule adoption:

1. Pull the rule's by-setup × direction table from its source note.
2. Compute per-cell Δ(rule on | other rules on) for perR and DD.
3. Does the rule's gain concentrate in ≤ 50% of cells? If yes, a conditional form exists.
4. Does the conditional form dominate on perR **AND** DD, or only one? If only perR, the uniform form likely wins on DD (as here).
5. Is the concentration structurally explainable (Brooks concept grounding)? If yes, it's signal; if no, sample artifact.

Rule 10 already passes this test trivially — its gain is 100% concentrated at L1-short, 0% at longs. The rule is already structured as "shorts only" (the conditional form IS the rule).

Rule 11 passes step 3 (concentration) but fails step 4 (conditional loses DD). Uniform stands.

### Adoption decision

**No PLAYBOOK change.** This is a resolution note — it closes a phantom open question implicit in pt 23 and formalizes a reusable decomposition pattern. Rules 1-11 unchanged; sizing tags unchanged; economics unchanged.

---

## Why this matters for trading

You don't need to do anything. The PLAYBOOK still delivers +1.84 R/trade at −2R max DD on the C3 combined stack. What this run did was confirm that **uniform rule 11 is the correct form** even though its lift is unevenly distributed across setups — because DD improvement is the real prize, and the conditional form gives that DD floor back.

---

## What's next — needs your nod

Nothing from this run. Resolution notes don't propose rule changes.

Open from earlier runs (still pending your nod):
- **Hybrid rule 9** adoption (H1/H2 longs & H1 shorts → 5R, H2 shorts → 4R, L1/L2 → 3R). Walk-forward positive, +0.36R/trade over 3R uniform. From pt 17.
- **Conditional rule 10** (2-bar swing stop on shorts). 24/25 LOO-weeks favor. From pt 21.
- **Conditional rule 11** (drop signal bars with `opp_tail ≥ 0.25`, applied uniformly — pt 32 confirms uniform is correct). 25/25 LOO-weeks + 52/52 LOO-symbols favor. DD floor lifts from −4 to −2. From pt 23, reaffirmed in pt 32.

---

## Source

- Long-form note: [`notes/small-pullback-trend-rule11-setup-concentration-2026-04-19.md`](notes/small-pullback-trend-rule11-setup-concentration-2026-04-19.md)
- PLAYBOOK: `~/code/aiedge/vault/Brooks PA/concepts/small-pullback-trend-PLAYBOOK.md`
- Reading-order index: `~/code/aiedge/vault/Brooks PA/concepts/small-pullback-trend-INDEX.md`
- Source data: `~/code/aiedge/scanner/scratch/_out_spt_rule10_rule11_interaction_2026_04_19.txt` (pt 27 §5) — no re-run needed.

---

## Run history

- **2026-04-19 · pt 32** — Rule-11 setup-level concentration decomposition. Gain concentrates at H1-long (+0.30) and L1-short (+0.15); H2-long flat (+0.04); L2-short n=3 artifact. Conditional C3ʹ matches uniform C3 on perR but worsens DD floor (−2 → −3). Decision: keep uniform. Closes phantom-Q from pt 23 §4. No PLAYBOOK change.
- **2026-04-19 · pt 31** — Brooks transfer failure taxonomy (synthesis). Names Inversion/Rarity/Emptiness failure modes; proposes 3-check vetting checklist; states arc's forward priorities. No PLAYBOOK change.
- **2026-04-19 · pt 30** — Q43 first MA-gap-bar exit overlay closed NEGATIVE/EMPTY. Brooks-canonical cell fires 0/101; 16/101 trades ever see a gap bar with mtm averaging −1.66R. Third Brooks→aiedge transfer failure.

---

## TL;DR in plain English

Here's what we asked today: **rule 11 says "skip signal bars with an ugly counter-trend wick." Does that rule help equally across all four types of SPT trade, or does it only help some?**

Answer from the data we already have: rule 11 does almost all its work on the "first pullback" setups (H1 long, L1 short). On the "second pullback" setups (H2, L2) it's basically a no-op on the numbers. So the natural follow-up: should we only apply rule 11 to the first-pullback setups and leave the others alone?

Answer: **no.** When we math it out, the expectancy is identical either way (+$184 vs +$183 on a $100 risk), but the version that applies rule 11 everywhere gives you a smaller worst day (−$200 vs −$300). And "smaller worst day" is why we adopted rule 11 in the first place. So: **keep applying rule 11 to everything, not just the first-pullback setups.**

**Nothing changed on your end.** The playbook still reads +1.84 R/trade at −2R worst drawdown. This was a bookkeeping pass that confirmed a decision we made a few runs ago was correct — and wrote down *why* it was correct in case we're tempted to revisit it.

**Why this kind of run is useful even when nothing changes:** when a rule fires unevenly across setups, the first instinct is "let's make it conditional." Sometimes that's right (rule 10 — shorts only). Sometimes it's wrong (rule 11 — uniform wins on drawdown). The only way to know which is to sit with the numbers. Today's run does that for rule 11 and bakes the reasoning into the PLAYBOOK so future-you doesn't re-ask the question.
