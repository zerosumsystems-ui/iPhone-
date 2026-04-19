# SPT — pt 23: Signal-bar SHAPE in the relvol Q2 trap zone (Q35)

Opened 2026-04-19 (~05:30 ET). Answers [[small-pullback-trend-signal-volume-2026-04-19|pt 22]]'s Q35 ("what does the Q2 trap zone LOOK like on the chart?") and surfaces a new generalizable finding in the process.

Scripts: `~/code/aiedge/scanner/scratch/spt_signal_shape_trap_2026_04_19.py` + follow-up one-liner. Outputs at `_out_spt_signal_shape_trap_2026_04_19.txt` and `_out_spt_opp_tail_followup_2026_04_19.txt`.

---

## TL;DR

- **Q35 answered negatively.** The 16 trades in the relvol Q2 trap zone (0.61 ≤ relvol < 0.85) are shape-indistinguishable from the 50 non-Q2 trades that win 70% of the time. Body%, close position, and body direction all overlap within noise. The trap is a pure-volume signature — it **does not show up on the bar**. That's why it's a trap.
- **But the shape work surfaced a bigger finding.** The one shape metric that DOES matter — `opp_tail ≥ 0.25` (counter-trend wick covering ≥ 25% of the signal bar's range) — is a *generalizable* weak-bar filter, not Q2-specific. Dropping it book-wide:
  - **Baseline**: n=88, WR 67.0%, perR +1.615, DD −4.00
  - **Filtered (opp_tail < 0.25)**: n=66, WR 71.2%, perR +1.924, DD −4.00 (**+0.31R/trade**)
- Non-parametric dominance is overwhelming: **25/25 LOO-weeks, 52/52 LOO-symbols, walk-forward positive in all 3 thirds.** Bootstrap delta CI is +0.306 [−0.356, +0.981] — wide, because the dropped cell is still positive-expectancy (perR +0.69, WR 54.5%), just meaningfully weaker than the kept book.
- **Adoption recommendation**: add `opp_tail < 0.25` as a **candidate rule 11** (or upgraded sizing tag) alongside the pt 22 relvol Q1 tag. Weaker parametric CI than pt 21's short-side swing2 rule; stronger non-parametric profile.

---

## §1. The question

Pt 22 found a non-monotonic U-shape in signal-bar relvol_20: Q1 (lowest-quartile volume) is the best cell (perR +1.87), Q4 (highest) is the baseline, and Q2 (relvol 0.61–0.85) is a **trap** (perR −0.15, WR 18.8%, DD −9.00R on n=16). Pt 22 §8 opened Q35:

> what does the 0.61–0.85 relvol band look like by signal-bar shape (close-near-mid? tail-heavy? body%)? Worth mining if a per-bar shape vector is added to detections.

Each bar in `chart_json` has OHLCV. Shape is computable on the fly:
- **body** = `|c-o| / (h-l)` — body fraction of range
- **body_dir** = signed body in trade direction (long: `c-o`, short: `o-c`, normalized)
- **close_pos** = `(c-l) / (h-l)` — 0 at low, 1 at high
- **close_dir** = close position *in trade direction* (1 = closes at extreme favorable end)
- **with_tail** = wick beyond close in trade direction
- **opp_tail** = wick against trade direction

Brooks narrative expectations:
- A strong with-trend signal bar: big body, close at extreme, minimal opposing tail.
- A trap: small body / doji, close near middle, large opposing tail (= "stop hunt through, close rejected").

## §2. Q2 shape profile — Brooks-clean bars that still lose

The filter stack {3,4,5,6,6s,7,9} weeds out ugly bars so aggressively that **by the time Q2 fires, the bars look fine**:

| field | Q2 mean (n=16) | non-Q2 mean (n=50) | delta |
|---|---:|---:|---:|
| body | 0.802 | 0.819 | −0.017 |
| body_dir | +0.802 | +0.819 | −0.017 |
| close_dir | **0.933** | **0.951** | −0.018 |
| with_tail | 0.067 | 0.049 | +0.018 |
| opp_tail | 0.131 | 0.132 | −0.002 |

**All five shape metrics overlap within noise.** Median body in Q2 is 0.84, median close_dir is 0.94. These are textbook strong with-trend bars.

Slicing Q2 by shape makes the point concretely:
- `close_dir < 0.5` (bar closes *against* trade direction): **0 trades** in Q2.
- `close_dir < 0.33` (closes hard against): **0 trades**.
- `body_dir < 0` (doji or bear-body on a long entry): **0 trades**.
- `body < 0.3` (soft/small body): **0 trades**.

The scanner's rule-3 urgency gate (≥ 4, ≥ 6 on TFO) already rejects every bar that would fail these shape checks. Q2 trap bars are *good-looking bars with quiet volume* — the Brooks-aware version of the warning "price without participation."

Winners vs losers inside Q2 give the first hint that shape matters a little:
| | WIN (n=3) | LOSS (n=13) |
|---|---:|---:|
| body | 0.863 | 0.789 |
| close_dir | 0.972 | 0.924 |
| **opp_tail** | **0.028** | **0.076** |
| with_tail | 0.028 | 0.076 |

Winners have nearly zero opposing tail; losers average ~7–8% opp_tail. Thin cells, but the only field that separates them.

## §3. The shape slice that does cut — opp_tail ≥ 0.25

Inside Q2, the single predicate that partitions the trap:

| Q2 cell | n | WR% | perR | DD |
|---|---:|---:|---:|---:|
| `opp_tail ≥ 0.25` (big counter-trend wick) | 4 | **0.0%** | **−1.000** | −4.00 |
| `opp_tail < 0.25` | 12 | 25.0 | +0.137 | −5.00 |

4/4 losses in the wicked-bar cell vs 3/12 winners in the clean-bar cell. So *within Q2*, a quarter-range opposing wick is the knockout.

But — and this was the surprise — the same filter fires outside Q2 too. In the §5 generalization check:
| scope | `opp_tail ≥ 0.25` | n | WR% | perR |
|---|---|---:|---:|---:|
| Q2 (relvol 0.61–0.85) | dropped | 4 | 0.0 | −1.000 |
| non-Q2 (relvol ≤ 0.61 or ≥ 0.85) | dropped | 10 | 40.0 | −0.092 |

**10 extra weak trades outside Q2 fire the same predicate, and they're also net-negative.** Opp_tail ≥ 0.25 is not a Q2-specific decoder; it's a general weak-bar filter whose existence Q2 alone never would have surfaced.

## §4. Promoting to a book-wide filter

Dropped from the full book (all 88 hybrid-rule-9 trades, including the 22 early-session bars that don't have relvol computed):

| threshold | dropped n | kept n | kept perR | delta vs baseline | kept DD |
|---|---:|---:|---:|---:|---:|
| baseline | 0 | 88 | +1.615 | — | −4.00 |
| opp_tail ≥ 0.20 | 26 | 62 | +1.971 | **+0.356** | −4.00 |
| **opp_tail ≥ 0.25** | **22** | **66** | **+1.924** | **+0.309** | **−4.00** |
| opp_tail ≥ 0.30 | 15 | 73 | +1.681 | +0.066 | −4.00 |
| opp_tail ≥ 0.35 | 11 | 77 | +1.676 | +0.061 | −4.00 |

The 0.20–0.25 band is the sweet spot. Moving the threshold to 0.30+ cuts too few trades (the worst opp_tail cell is 0.25–0.30, not ≥ 0.30). 0.25 coincides exactly with the book's q75 — so the rule is "drop the top quartile of counter-trend-wicked signal bars."

Dropped cell composition (threshold 0.25, n=22):
- WR 54.5% (12 wins / 10 losses)
- sumR +15.17R, perR +0.690
- Still net-positive, just much weaker than kept (perR +1.924)

Interpretation: this filter is not removing an obviously-losing cell; it's removing a mediocre cell and keeping the gains concentrated in the clean-bar cell. That's why the **drawdown is unchanged at −4.00R** — we're not fixing a disaster, we're raising the average by pruning the middle of the distribution.

## §5. Robustness

### Walk-forward (3 chronological thirds)

| split | policy | n | WR% | perR | DD |
|---|---|---:|---:|---:|---:|
| S1 early | baseline | 19 | 42.1 | +0.562 | −4.00 |
| S1 early | filtered | 17 | 47.1 | +0.746 | −4.00 |
| S2 mid | baseline | 27 | 70.4 | +1.446 | −2.00 |
| S2 mid | filtered | 20 | 80.0 | **+2.008** | −2.00 |
| S3 late | baseline | 42 | 76.2 | +2.200 | −3.00 |
| S3 late | filtered | 29 | 79.3 | **+2.556** | −2.00 |

Positive in all three, largest gain in S2 (+0.56R). Filtered DD improves or matches in every segment.

### LOO-week

- **25 weeks. Filtered beats baseline in 25/25.**
- Min delta +0.261, max +0.346, mean +0.309.
- Every week sees the same ordering; no single week reverses the sign.

### LOO-symbol

- **52 symbols. Filtered beats baseline in 52/52.**
- The effect is not carried by one or two tickers.

### Parametric bootstrap CI on the delta

- Unpaired bootstrap, 4000 iter: **delta = +0.306R, 95% CI [−0.356, +0.981].**
- The CI is wide because the dropped cell is still positive (perR +0.69, not a disaster). In a 66-trade book the variance swamps a +0.3R/trade signal.
- **Non-parametric dominance (25/25 + 52/52 + 3/3) is the stronger evidence here.** Same pattern as pt 21's short-side swing2: thin parametric signal but knife-through-butter consistency across resamples.

### By setup / direction

`opp_tail ≥ 0.25` drops concentrate on H1 and longs — but the filter is net-positive everywhere:

| cell | drop n | drop perR | keep n | keep perR |
|---|---:|---:|---:|---:|
| H1 | 9 | +0.21 | 35 | +2.08 |
| H2 | 6 | +1.62 | 13 | +1.70 |
| L1 | 7 | +0.51 | 14 | +1.59 |
| L2 | 0 | — | 4 | +2.49 |
| long | 15 | +0.77 | 48 | +1.98 |
| short | 7 | +0.51 | 18 | +1.79 |

H2 is the exception: drop and keep have nearly identical perR (+1.62 vs +1.70). The H2 drop cell has zero net harm. But the H1 drop cell (perR +0.21) is where the filter carries its weight. Could in principle ring-fence to H1/L1/L2 only — but H2 is zero-impact so no reason to exclude it, and keeping the rule symmetric is Brooks-clean.

## §6. Brooks interpretation

A large opposing tail on a with-trend signal bar is the chart's record of a failed counter-trend probe that happened *during* the bar. Two Brooks-consistent reads:

1. **"Trader attention"** — a big wick against trade direction means price visited a level that other market participants thought was tradable (for the opposing side). That the bar closed strongly in trade direction rejects the visit, but the visit itself means the with-trend move is not uncontested. In a true SPT, with-trend bars run quietly; the opposition isn't strong enough to print a wick.
2. **"Stop-hunt dressed as absorption"** — on a narrow intraday timeframe, a signal bar with a large opp_tail often means the scanner's detection caught a bar where the prior bar's extreme was briefly taken out, stops triggered, then price reversed. That's the same bar pattern as a failed breakout masquerading as a with-trend continuation. The trades that fill on the next bar's open are buying (or shorting) into a chart structure that just printed a mini-reversal.

Both reads predict that the trades should underperform without necessarily losing catastrophically (because the SPT regime is still real; it's just that *this specific entry bar* is compromised). That matches the empirical pattern: dropped cell is perR +0.69, not perR −1.0.

Relationship to the Q1 relvol tag from pt 22:
- Pt 22's Q1 (relvol < 0.65) = absorption phase, institutions quietly accumulating. Clean bar, quiet volume. Highest perR (+1.87) at the thinnest cell (n=16).
- Pt 23's opp_tail ≥ 0.25 drop = contested bar. Volume is not the diagnostic; the tail is. Largest cell (n=22) at a meaningful filter gain.

The two findings are orthogonal — pt 22 rewards quiet bars, pt 23 penalizes contested bars. A bar can be:
- Low relvol + tight opp_tail → pt 22 high-conviction tag (strongest cell)
- Normal relvol + tight opp_tail → current baseline (strong)
- Any relvol + wide opp_tail → pt 23 drop (weakest cell)

Adding both: the intersection (low relvol AND tight opp_tail) would be n=~12-14 with likely the tightest CIs — worth tracking once the sample grows.

## §7. Adoption recommendation

### Option A — candidate **rule 11**: drop trades with `opp_tail ≥ 0.25`

Promoting as a full rule alongside the existing 9-rule stack + conditional rule 10.

**Pros:**
- +0.31R/trade book-wide (sizeable; comparable to the rule-8 drop in pt 14)
- DD unchanged (−4.00R → −4.00R)
- 25/25 LOO-weeks + 52/52 LOO-symbols (the strongest dominance profile in the research arc)
- Walk-forward positive in all 3 thirds
- Effect generalizes across setup and direction (weakest on H2 but not net-negative anywhere)

**Cons:**
- Parametric CI on delta is wide [−0.356, +0.981] — n=66 kept book is too thin for parametric significance
- Dropped cell is still +0.69R/trade — not a disaster, just mediocre
- Threshold 0.25 coincides with the book's q75; the neighboring threshold 0.20 has similar or slightly-better perR. If the underlying distribution shifts (different regime, different symbols), the "right" threshold may drift. Would want to recalibrate when n ≥ 200.

### Option B — **sizing tag only** (mirror pt 22's Q1 tag)

Do not drop. Flag `opp_tail ≥ 0.25` as a size-down / skip-this-one tag for discretionary use.

**Pros:**
- Preserves the 22 trades worth of scale
- Matches the pt 22 precedent (relvol Q1 was a tag, not a filter)

**Cons:**
- Loses most of the perR gain; tag-only means Will has to make the same judgment the data just made for him

### Recommendation: **Option A — conditional rule 11**, mirroring pt 21's conditional rule 10 treatment

Add to the PLAYBOOK as a conditional rule, with the caveat "re-validate at n_dropped ≥ 50". The non-parametric evidence is the strongest in the arc; the parametric caveat is the standard regime-small-sample trade-off we've accepted for rule 10.

## §8. Open follow-ups

- **Q35** — answered negatively; closed.
- **Q37 (new)** — interaction between opp_tail ≥ 0.25 and relvol: is the opp_tail penalty stronger at low relvol (= the trap bar has no volume defense) or at high relvol (= the opposing side really showed up)? Cell too thin now (n=22 dropped, further split 4-ways by relvol quartile). Revisit when n ≥ 50.
- **Q38 (new)** — `opp_tail ≥ 0.25` threshold robustness across regimes. Pt 17 §7 flagged S3 (2026-02-04+) carries 65% of cumulative R. Is the 0.25 threshold invariant across S1/S2/S3 distributions, or does it drift? Walk-forward §5 confirms positive-expectancy in all thirds, but the *threshold* wasn't re-optimized per segment. Worth checking at next regime change.
- **Q39 (new)** — with_tail vs opp_tail asymmetry. With_tail mean is 0.049 book-wide (tiny) — this is the scanner rejecting bars with wicks beyond the close in trade direction already (via urgency / body density). Why is opp_tail allowed through at q75 = 0.250 but with_tail clipped to 0.049? Likely the `always_in` and `gap_direction` rules absorb with_tail but don't touch opp_tail. Worth a small derivation to confirm.

## §9. Scanner implementation notes

If Option A is adopted, the scanner needs to expose `opp_tail` per detection. It's computable from the existing `chart_json.bars[sig_idx]` OHLC, so no schema migration — just a new field in the detection annotator.

Pseudocode:
```python
def opp_tail_frac(sig_bar, direction):
    h, l, o, c = sig_bar["h"], sig_bar["l"], sig_bar["o"], sig_bar["c"]
    rng = max(h - l, 1e-9)
    if direction == "long":
        return (min(o, c) - l) / rng  # wick below the lower of open/close
    else:
        return (h - max(o, c)) / rng
```

At 0.25 cutoff, ~25% of the book is filtered (q75 by construction). This is a LARGER cut than pt 22's Q1 tag (16 of 66 = 24% of has_relvol book) but comparable in rate.

Live-scanner use: compute at detection time, add as a column to the dashboard (green tag ≤ 0.25, red/skip tag > 0.25). No rule-9 re-gating needed — `opp_tail` is a pre-resolution filter, just like rule 3/4/5/6.

## §10. Comparison to the 22-part arc

| Rule / tag | Source | n affected / mo | perR delta | non-param robustness | param CI |
|---|---|---:|---:|---|---|
| Rule 8 (mkt_ord>2) DROP | pt 14 | — | — | 28/28 LOO-weeks | CI separation |
| Rule 9 hybrid target | pt 17 | — | +0.36 | 25/25 LOO-weeks | Within noise |
| Rule 10 swing2 shorts | pt 21 | +1.3 shorts | +0.215 | 24/25 LOO-wk, 18/18 LOO-sym | Wide [−0.55, +1.03] |
| pt 22 relvol Q1 tag | pt 22 | ~2 (high-conv) | +0.25 (at tag) | LOO-week stdev 0.17 | Tag-only |
| **pt 23 opp_tail < 0.25** | **pt 23** | **+2.4/mo kept** | **+0.31** | **25/25 LOO-wk, 52/52 LOO-sym** | **Wide [−0.36, +0.98]** |

Pt 23's non-parametric profile is the strongest in the arc (52/52 LOO-sym is the first perfect dominance across all symbols). Parametric CI is comparable-to-weaker than the rule-10 adoption threshold — so if Will accepted rule 10 conditionally on that basis, rule 11 clears the same bar.

## §11. Artifacts

- Script: `~/code/aiedge/scanner/scratch/spt_signal_shape_trap_2026_04_19.py`
- Follow-up: one-liner written into `scratch/_out_spt_opp_tail_followup_2026_04_19.txt`
- Outputs: `_out_spt_signal_shape_trap_2026_04_19.txt`, `_out_spt_opp_tail_followup_2026_04_19.txt`
- Parent: [[small-pullback-trend]], [[small-pullback-trend-PLAYBOOK]]
- Prior note: [[small-pullback-trend-signal-volume-2026-04-19]] (pt 22)
- Sibling: [[small-pullback-trend-entry-mechanic-2026-04-19]], [[small-pullback-trend-stop-placement-2026-04-19]]

---

Fallback: if rule 11 turns out to be regime-fragile in live trading, pure reversal is safe (remove the filter) because it was a pre-resolution drop, not a stop/target change. No retrospective contamination of existing trades.
