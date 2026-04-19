# SPT research — phone briefing

Autonomous small-pullback-trend research notes, phone-readable. Long-form notes live in [`notes/`](notes/); the latest TL;DR is at the top of this README.

## Latest — pt 30: First MA-gap-bar exit overlay (Q43) — **NEGATIVE/EMPTY**

**Run:** 2026-04-19, scheduled-task autonomous. Closes Q43, the third candidate from the [pt 26 Brooks cross-reference §3](notes/) (after Q41 inverted, Q42 rare).

### Headline

Brooks *Trends* ch. 47 line 45 teaches that "the first MA-gap bar in a strong trend often leads to the final leg before a deeper, longer-lasting pullback." The candidate rule:

> Exit at first post-entry MA-gap bar **IF currently at +1.5R or better**.

**Result: 0 fires.** On the canonical PLAYBOOK trade population (n=101 baseline trades), the Brooks-canonical cell (threshold +1.5R, EMA20, exit at gap-bar close) fires zero times. 12-cell sweep across {+1.0R, +1.5R, +2.0R} × {EMA20, EMA10} × {close, next_open} never positive; best non-zero cell is **−0.015 R/trade**.

### Why — base rate diagnostic

| | n |
|---|---:|
| Baseline trades that EVER see a post-entry MA-gap bar (EMA20) | **16 / 101** |
| Of those 16, mtm at the gap-bar close | mean **−1.66R**, median −1.79R, max +1.43R |
| Cells at ≥ +1.0R | 1 / 16 |
| Cells at ≥ +1.5R | **0 / 16** |
| Cells at ≥ +2.0R | 0 / 16 |

**Mechanism — "the conditional is empty."** By the time the pullback is deep enough for an MA-gap bar to print, the trade is already deeply negative or has already been stopped out. Brooks's "lock in gains on first sign of weakness" framing has nothing to lock in.

### The pt 28/29/30 cluster — three for three

Three Brooks-explicit candidate exit/entry overlays from the pt 26 §3 audit, all closed negative:

| pt | Q | Source | Result | Mechanism |
|--:|---|---|---|---|
| 28 | Q41 next-session follow-through | ch. 57 line 11 | NEGATIVE | inverted sign — scanner labels Brooks's structure as the opposite |
| 29 | Q42 2× climactic-burst exit | ch. 57 line 13 | NEGATIVE/RARE | structurally rare — only 2/101 trades reach the canonical cell |
| 30 | Q43 first MA-gap bar exit | ch. 47 line 45 | NEGATIVE/EMPTY | conditional empty — 16/101 see the bar, 0 of those are profitable |

**Generalizable lesson:** Brooks's specific entry/exit MECHANICS don't transfer to a filter-heavy multi-symbol scanner stack. His CONCEPTS do — rules 1–7, 9, 11 are all Brooks-grounded and empirically positive. The split is robust enough now to call the **Brooks-source well effectively exhausted for the SPT setup family** on US single-name equities. Future +R will come from scanner-side enrichments (raw component scores, cross-asset expansion, day-type stability), not from porting more discretionary tape-reading rules.

### Adoption decision

**No PLAYBOOK change.** Rule 9 (hold to target) stands. No new rule 12 candidate.

---

## Why this matters for trading

You don't need to do anything. The PLAYBOOK is unchanged at +1.6 to +1.8 R/trade with the 11 rules + conditional rules 10 and 11. The pt 30 work confirms that the current stack is at a local maximum for Brooks-derived rules on this universe.

---

## Source

- Long-form note: [`notes/small-pullback-trend-first-ma-gap-exit-2026-04-19.md`](notes/small-pullback-trend-first-ma-gap-exit-2026-04-19.md)
- Reproducer script: `~/code/aiedge/scanner/scratch/spt_first_ma_gap_exit_2026_04_19.py`
- Output text: `~/code/aiedge/scanner/scratch/_out_spt_first_ma_gap_exit_2026_04_19.txt`
- Brooks source: `~/code/aiedge/brooks-source/extracted/trading-price-action-trends/47_signs-of-strength-in-a-trend.md`
- PLAYBOOK: `~/code/aiedge/vault/Brooks PA/concepts/small-pullback-trend-PLAYBOOK.md`
- Reading-order index: `~/code/aiedge/vault/Brooks PA/concepts/small-pullback-trend-INDEX.md`
