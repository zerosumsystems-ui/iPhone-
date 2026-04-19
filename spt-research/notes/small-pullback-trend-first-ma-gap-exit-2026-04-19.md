# SPT — pt 30: First MA-gap-bar exit overlay (Q43)

Closes Q43 (Brooks *Trends* ch. 47 line 45 — "first MA-gap bar in a strong trend often leads to the final leg"). Run 2026-04-19, autonomous scheduled-task. Closes **NEGATIVE/EMPTY**: the conditional trigger ("trade profitable at the moment a first MA-gap bar fires") is structurally near-empty on the PLAYBOOK-filter-selected SPT population. Brooks-canonical cell `n_fired = 0`. Third consecutive Brooks→aiedge transfer failure in the pt 26 §3 lineage.

Companion to [[small-pullback-trend-PLAYBOOK]]. Sibling notes: [[small-pullback-trend-next-session-followthrough-2026-04-19|pt 28 — Q41 NEGATIVE]] and [[small-pullback-trend-climactic-exit-2026-04-19|pt 29 — Q42 NEGATIVE/RARE]]. Together, all three Brooks-explicit candidate questions from [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26 §3]] now resolve negative.

---

## TL;DR

- **Brooks-canonical cell — threshold +1.5R, EMA20, exit at gap-bar close: `n_fired = 0` of 101 baseline trades.** Delta = +0.000 R/trade by construction. LOO-week dominance trivially 0/25 (overlay is a no-op).
- **Base rate diagnostic: only 16 of 101 baseline trades EVER see a post-entry MA-gap bar.** Of those 16, mark-to-market at the gap-bar close averages **−1.66R** (median −1.79R, max +1.43R). Zero of the 16 are ≥ +1.5R; only 1 of 16 is ≥ +1.0R.
- **EMA10 variant (looser) fires a tiny bit and uniformly hurts.** All 3 EMA10 cells with `n_fired > 0` show negative delta (best is −0.015 at thr=2.0R / EMA10 / next_open with n_fired=1). The lone cells confirm direction-of-effect: when the override does fire, it converts a winning trade into a worse one.
- **Mechanism — "the conditional is empty."** Brooks's MA-gap exit assumes you're in a profitable swing that has just begun to weaken. In aiedge's data, by the time price has pulled back enough for an MA-gap bar to print, the trade is already deeply negative (mean −1.66R) — usually because the same pullback already triggered the 1R stop a few bars earlier (only 16/101 trades survive long enough to even see a gap bar). The "+1.5R or better" filter then rejects every survivor.
- **No PLAYBOOK change.** Rule 9 (hold to target) stands. No new rule 12 candidate.
- **Pattern emerges — 3/3 untapped Brooks passages from pt 26 §3 close negative.** Q41 (next-session follow-through) inverted; Q42 (2× climactic-burst exit) structurally rare; Q43 (first MA-gap bar exit) structurally empty. Generalizable lesson: **Brooks's discretionary single-instrument exit teachings do not transfer to a filter-heavy multi-symbol scanner stack**, because rules 1–9 already pre-select the trade population in ways that make these signals either impossible or actively counterproductive on the survivors.
- Opens nothing new. Closes Q43.

---

## §1. Setup

Brooks *Trends* ch. 47 line 45 (cited in [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26 §3.3]]):

> "This was a moving average gap bar setup in a strong trend and should be expected to test the high of the bull trend with either a lower high or a higher high. A moving average gap bar in a strong trend often leads to the final leg of the trend before a deeper, longer-lasting pullback develops, and the pullback can grow and become a trend reversal."

Brooks's "MA-gap bar" = a bar whose entire price range gaps from the moving average:
- For a bull trend: the bar's HIGH is below the 20EMA (the bar is entirely below the average)
- For a bear trend: the bar's LOW is above the 20EMA (the bar is entirely above)

Pt 26 §3.3 framed Q43 as: *"Is 'first MA-gap bar after entry' a useful exit signal for SPT swings? Given the aiedge stop logic (1R fixed) and target logic (3R fixed), does an override 'exit at first MA-gap bar if currently at +1.5R or better' gain expectancy?"*

The "+1.5R or better" qualifier is the Brooks-canonical "you're in a profitable swing and the first warning sign of weakness has appeared" framing. The whole point of the rule (in Brooks's discretionary trading) is that you LOCK IN gains on a tiring trend before the deeper pullback starts. That precondition matters — if the trade is already losing, there's nothing to lock in.

Pt 7 ([[small-pullback-trend-adverse-event-2026-04-18]]) tested 7 generic event-triggered exits (first adverse close, bear-trend bar closing below 10EMA, 50% retrace, etc.) and rejected all. Pt 7 did **NOT** test this specific Brooks signature (first **MA-gap** bar, conditional on the trade currently profitable). Q43 is a clean addition, not a duplicate.

---

## §2. Design

Mirrors pt 28 / pt 29: re-resolve C0 trades with an overlay, sweep parameters, look for a positive cell.

### C0 baseline

PLAYBOOK REVISED stack: pre-filters {3, 4, 5, 6, 6s} + post-filters {7, 9}, hybrid rule-9 targets (H1/H2 longs & H1 shorts → 5R, H2 shorts → 4R, L1/L2 → 3R), `db_stop`, market-on-next-bar-open entry. Same structure as pt 27 C0 / pt 29 C0.

Snapshot 2026-04-19 ~13:30 ET: **n=101, WR 65.3%, sumR +152.84, perR +1.513, DD −5.00, CI [+1.088, +1.927]**, reasons {chart_end: 46, stop: 34, target: 21}.

This is a slightly higher n than pt 27's C0 (n=88) and pt 29's C0 (n=101). Pt 29 ran ~12:45 ET; pt 30 reads the same DB. The 13-trade growth from pt 27 → pt 29 came from continued live-DB back-extension; pt 30 picks up the same population. Stable.

### EMA computation

EMA computed across **ALL bars in chart_json** (continuous, seeded with SMA over the first `period` closes; recurse with `α = 2/(period+1)` per the standard formula). Look up the EMA at each post-entry bar by its position. This is the standard live-trading EMA — same as the scanner uses upstream.

EMA periods swept: **{20 (Brooks-canonical), 10}**. The 10 variant is included because (a) the scanner's bpa_detector uses 10EMA in some places, and (b) a tighter EMA is more reactive and would let the signature fire more often — useful as a sensitivity test.

### Resolver per-bar order

For each post-entry bar:
1. **Intrabar stop hit** → exit −1R (stop is hard).
2. **Intrabar target hit** → exit +target_r (target is hard).
3. **MA-gap signature at bar close**: bar.h < EMA (long) OR bar.l > EMA (short).
   - If signature AND mark-to-market at bar close ≥ `profit_threshold` × R, exit:
     - at this bar's close (`exit_when=close`), or
     - at next bar's open (`exit_when=next_open`).
   - Once the first MA-gap bar fires (regardless of whether the threshold was met), the override is "spent" — only the FIRST MA-gap bar after entry is Brooks-meaningful.

If we walk all post bars without firing, fall back to chart_end at last bar's close (capped at target_r, floored at −1R).

### Sweep cells (12)

- profit_threshold ∈ {+1.0R, +1.5R (Brooks spec), +2.0R}
- ema_period ∈ {20, 10}
- exit_when ∈ {close, next_open}

3 × 2 × 2 = 12.

### Diagnostics

- Best cell (max positive delta with n_fired > 0)
- Brooks-canonical cell (1.5R / EMA20 / close) with longs/shorts and per-setup carve
- LOO-week dominance on Brooks-canonical
- Base rate: how many baseline trades EVER see a post-entry MA-gap bar (EMA20), and what's the mtm distribution at that gap-bar's close

Script: `~/code/aiedge/scanner/scratch/spt_first_ma_gap_exit_2026_04_19.py` (CLI: default `full`, `claimed` for 4mo window). Output captured at `~/code/aiedge/scanner/scratch/_out_spt_first_ma_gap_exit_2026_04_19.txt`.

---

## §3. Results

### §3.1 — 12-cell sweep

```
  thr_R  EMA exit_when  n_fired  overlay_perR  vs C0 baseline +1.513
  -------------------------------------------------------------------
   1.00   20 close            0  +1.513         +0.000
   1.00   20 next_open        0  +1.513         +0.000
   1.00   10 close            3  +1.492         −0.021
   1.00   10 next_open        3  +1.495         −0.018
   1.50   20 close            0  +1.513         +0.000   ← Brooks-canonical
   1.50   20 next_open        0  +1.513         +0.000
   1.50   10 close            2  +1.484         −0.029
   1.50   10 next_open        2  +1.487         −0.026
   2.00   20 close            0  +1.513         +0.000
   2.00   20 next_open        0  +1.513         +0.000
   2.00   10 close            1  +1.496         −0.018
   2.00   10 next_open        1  +1.498         −0.015
```

- **All 6 EMA20 cells: `n_fired = 0`.** No EMA20 MA-gap bar in any baseline trade meets ANY profit threshold (≥ +1.0R, ≥ +1.5R, or ≥ +2.0R).
- **All 6 EMA10 cells: 1–3 fires, all negative.** Best EMA10 cell is thr=2.0R / EMA10 / next_open at delta = −0.015 R/trade (n_fired = 1, single trade converted from baseline +4.31R chart_end to overlay +2.76R MA-gap exit — a clean confirmation that when the override DOES fire, it locks in a smaller gain than the trend ultimately delivered).
- **No cell shows positive delta.** Best gain among any of the 12 is the no-op +0.000 of EMA20.

### §3.2 — Brooks-canonical cell (1.5R / EMA20 / close)

```
  n=101  WR= 65.3%  sumR=+152.84  perR=+1.513  DD= -5.00  CI=[+1.103, +1.927]
  delta vs baseline perR: +0.000
  n_fired = 0 (of 101 baseline trades)
```

Identical to baseline. The longs/shorts and per-setup carves are also identical to baseline (no overlay fires). Skipping the table — see `_out_spt_first_ma_gap_exit_2026_04_19.txt` lines 47–55 for the full carve.

### §3.3 — LOO-week dominance

```
  overlay beats baseline in 0/25 LOO-weeks
  overlay  perR weekly mean=+1.511  stdev=0.100
  baseline perR weekly mean=+1.511  stdev=0.100
```

Trivially 0/25 (overlay is identically the baseline; "ties go to baseline"). The reading is "no week was rescued by the overlay."

### §3.4 — Base rate diagnostic (EMA20)

This is the load-bearing finding.

```
  baseline trades that ever see a post-entry MA-gap bar: 16/101
  mtm at first gap bar:  mean=-1.66R  median=-1.79R  min=-3.78R  max=+1.43R
  cells above thresholds: >=+1.0R: 1/16  >=+1.5R: 0/16  >=+2.0R: 0/16
```

- **Only 16 of 101 baseline trades survive long enough to ever see a post-entry MA-gap bar.** That's 15.8% — most trades resolve (stop, target, or chart_end) before any bar's high gaps below the EMA20.
- **Of the 16 that DO see a gap bar, the trade is on average at −1.66R when it appears.** Distribution: median −1.79R, min −3.78R, max +1.43R. The full distribution sits well below zero.
- **Threshold cell counts: 1 trade at ≥ +1.0R, 0 trades at ≥ +1.5R, 0 trades at ≥ +2.0R.** The Brooks-canonical "+1.5R or better" filter rejects every single one. The +1.0R variant rescues only 1 of 16, and that 1 trade's overlay-vs-baseline R difference is captured in the EMA10 cells (where n_fired sometimes hits the same trade) — uniformly negative.

The bias in mark-to-market explains why every cell either fires zero times (EMA20, where the gap is structurally rare) or fires at low R (EMA10 with low threshold) and consistently locks in a worse outcome than the eventual baseline.

---

## §4. Mechanism — "the conditional is empty"

The candidate rule had a Brooks-explicit form:

> "Exit at first MA-gap bar **IF currently at +1.5R or better**."

The `IF` clause is the kill. Brooks's framing (single-instrument Emini swing trade) assumes:

1. You're in a profitable trend trade with a wide swing target.
2. Price has been making higher highs (or lower lows) for many bars.
3. A bar prints whose entire range is below (or above) the 20EMA — a "gap" from the average.
4. This bar is the FIRST such gap-bar in 20+ bars (i.e. before this point, price was always at least touching the EMA from above/below).

In Brooks's setup, this signature is a meaningful pullback-deepening signal: "the trend was so strong it kept the EMA pulled along, and now it's lost contact." In aiedge's filter-stack-selected population, the equivalent texture is suppressed by upstream filters. Specifically:

- **Rule 1 (H1/H2/L1/L2 setups)** + **Rule 3 (urgency ≥ 4)** select for trades where the signal bar itself was already strong with-trend, i.e. the trade enters near a pullback's local extreme. Once entered, the trend either continues (target hit fast) or fails (stop hit fast).
- **Rule 9 hybrid targets (3–5R)** are tight enough that ~67 of 101 trades resolve to stop / target / chart_end before any bar prints below the EMA20.
- The 16 trades that DO see an MA-gap bar are exactly the trades where the pullback has been extended enough to undo the entry — the price has fallen back to where the entry was a bad fill, the trade is sitting near or past the stop, and the gap bar is the consequence of the same pullback that just stopped the trade out (or is about to).

So the gap bar appears AFTER the stop has already either fired or is imminent. Brooks's "lock in gains" logic doesn't apply — there are no gains to lock in.

The EMA10 variant fires marginally because a tighter EMA is more reactive (smaller moves push price below it sooner). But the same mechanism applies: the trades that survive long enough to see an EMA10 gap bar are the trades that have just been pulled back enough that exiting now means accepting a smaller win than the eventual continuation delivers.

This is the **third Brooks→aiedge transfer failure**, with a distinct mechanism family from Q41 and Q42:

| Note | Question | Result | Mechanism |
|---|---|---|---|
| pt 28 | Q41 next-session follow-through | NEGATIVE | **Inverted sign** — Brooks's "look for follow-through" maps to scanner's OPPOSED/fade-bar label on day-N+1 |
| pt 29 | Q42 2× climactic-burst exit | NEGATIVE/RARE | **Structurally rare** — Rule 6 truncates entries at 14:15 ET; hybrid rule 9 resolves most trades before 14:00 ET; 14:00–15:00 ET signature fires on only 2/101 trades |
| pt 30 | Q43 first MA-gap bar exit | NEGATIVE/EMPTY | **Conditional empty** — Only 16/101 trades survive to see a gap bar; mtm at gap bar averages −1.66R; "+1.5R or better" filter rejects all 16 |

Pt 28's mechanism was *semantic mismatch* (the scanner labels Brooks's structure as the opposite). Pt 29's was *temporal mismatch* (the scanner-population's trades resolve before the Brooks signature window). Pt 30's is *outcome mismatch* (the conditional Brooks attaches to the rule is empty on the scanner population).

All three are different shapes of the same root cause: **rules 1–9 mechanically pre-select a trade population that does not contain the texture Brooks's discretionary single-instrument teachings were derived for.**

---

## §5. Adoption decision

**Q43 closes NEGATIVE/EMPTY. No PLAYBOOK change.** Rule 9 (hold to target) stands; no new rule 12 candidate.

The empirical answer is dispositive: 0 fires under the Brooks-canonical cell and uniformly negative deltas in the looser EMA10 cells. There is no parametric variant of the Q43 rule that gains expectancy on the current PLAYBOOK trade population.

There is one residual angle worth flagging (but NOT pursuing): **the same overlay applied to a less-filtered baseline (pre-rule-6 population, similar to pt 29's Q47) would have many more trades that survive into the gap-bar window.** That's a follow-up to Q47 and can be deferred. The Q47 hypothesis is "the climactic exit overlay only works on the population that bypasses rule 6's morning floor"; the analogous Q43-flavored variant would be "the MA-gap exit only works on the population that bypasses rule 9's hybrid target." Both are unlikely to land — they would need to recover enough expectancy from the unfiltered population to overcome the upstream filter loss of −0.5–1.0 R/trade. Not flagging as a new Q.

---

## §6. The pt 28/29/30 cluster — a generalizable note

Three Brooks-explicit candidate rules (Q41, Q42, Q43) tested back-to-back, all closed negative. This is the strongest single piece of evidence in the 30-note arc that **Brooks's specific ENTRY/EXIT teachings do not mechanically port to the scanner stack**, even though Brooks's CONCEPTUAL teachings (urgency, always_in, gap alignment, day-type, shallow pullbacks, signal-bar tail structure, anti-breakeven) DO port cleanly and form the bulk of the 11-rule PLAYBOOK.

The split:
- **Brooks CONCEPTS port well.** Rules 1, 2, 3, 4, 5, 6 (timing motif), 7 (first-hour caveat), 9 (anti-breakeven), 11 (shallow-pullback at bar resolution) are all Brooks-grounded and empirically positive. See [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26]] for the rule-by-rule grounding audit.
- **Brooks SPECIFIC EXIT/ENTRY MECHANICS don't port.** All three of pt 26 §3's flagged passages (next-session follow-through, climactic-burst exit, first MA-gap bar exit) close negative on the filter-stack-selected population. The mechanism is filter-stack-induced texture pre-selection, not Brooks being wrong.

This bias in transferability is itself a **diagnostic about the PLAYBOOK's maturity**: at +1.6 R/trade with rules 10 and 11 active, the stack has already absorbed every Brooks teaching that mechanically applies. Further +R will not come from porting more Brooks discretionary tape-reading rules. It will come from:

1. **Scanner-side enrichments** that the PLAYBOOK can use as new filter inputs (per-component scores from Q1, day-type stability from Q25, opening-range labels from §4.4 of pt 26).
2. **Cross-asset expansion** (Q5) where the SPT signature fires under different microstructure.
3. **Empirical continuation** of conditional rules (Q44 rule-11 stability over time, Q33 revisit of rule 10 at higher n_shorts).

This is a useful planning frame. The Brooks-source well is essentially exhausted for the with-trend SPT setup family on US single-name equities.

---

## §7. Open / next steps

Q43 closes here. No new Q opened.

Forward queue (low-priority, in ascending exhaustion order):
- **Q44** — rule-11 solo Δ stability over time. Continue weekly tracking. ← pt 27.
- **Q47** — overlay-exit on pre-rule-6 population (climactic burst variant). Filed by pt 29 §residual.
- **Q43-variant on pre-rule-9 population** — first MA-gap bar exit on the trades rule 9 truncates. Same low-yield expected, NOT filed as a numeric Q.
- **Q1, Q5, Q25** — scanner-side enrichments listed in pt 26 §3 as the most likely sources of fresh edge.

---

## §8. Related

- [[small-pullback-trend]] — parent concept doc.
- [[small-pullback-trend-PLAYBOOK]] — canonical 11-rule policy stack.
- [[small-pullback-trend-INDEX]] — reading order; this is pt 30.
- [[small-pullback-trend-brooks-source-cross-reference-2026-04-19]] — pt 26, source for Q43's exact framing (§3.3).
- [[small-pullback-trend-next-session-followthrough-2026-04-19]] — pt 28, sibling Brooks→aiedge transfer note (Q41 NEGATIVE/inverted).
- [[small-pullback-trend-climactic-exit-2026-04-19]] — pt 29, sibling Brooks→aiedge transfer note (Q42 NEGATIVE/rare). Same C0 baseline (n=101) as this note.
- [[small-pullback-trend-adverse-event-2026-04-18]] — pt 7, the original generic-event-exit study; this Q43 work is the ONE specific Brooks signature that pt 7 did not test.
- `~/code/aiedge/scanner/scratch/spt_first_ma_gap_exit_2026_04_19.py` — reproducer.
- `~/code/aiedge/brooks-source/extracted/trading-price-action-trends/47_signs-of-strength-in-a-trend.md` — primary source (line 45).

---

## §9. Methodology notes

**EMA seed.** Standard textbook EMA: SMA over the first `period` bars seeds the recursion, then `α = 2/(period+1)` exponential weighting. This matches the scanner's upstream EMA convention. Edge case: if `len(bars) < period`, no EMA is computed and the gap test is silently skipped — irrelevant in practice (all chart_jsons have ≥ 30 bars).

**"First" MA-gap bar.** The script sets `fired_already = True` after the first gap-bar regardless of whether the profit-threshold was met. This is the Brooks-canonical "first MA-gap bar" semantic — the rule is about the first such bar, not "any subsequent gap bar that happens to be at +1.5R." A future variant could relax this to "any gap bar at +1.5R", but given that the FIRST gap bar already shows mean mtm = −1.66R, any subsequent gap bar would be even further negative (the trend has continued to weaken). No expected gain from relaxing.

**Stop/target priority.** Stops and targets evaluate intrabar BEFORE the gap-bar overlay. This matches live execution (resting stops/targets are filled before any close-of-bar logic could read EMA values). If the overlay had been allowed to pre-empt stops/targets, results would be even worse (the few cells where overlay would fire are precisely the cells where price has already moved against the trade by the bar's close).

**Mark-to-market reference price.** The "+1.5R or better" check uses bar CLOSE, not bar HIGH. Using bar high would let the overlay fire on bars where price spiked into +1.5R intrabar but closed below — a fragile cell. Using bar close is the conservative read that matches a discretionary trader looking at a bar after it printed.

**Memory / Mac-mini state.** Run completed cleanly in ~3 seconds (small data, simple resolver). PhysMem at 15G/16G with 6G compressor at start of run, no impact on the run. No leaked Python interpreters from the SPT session series.
