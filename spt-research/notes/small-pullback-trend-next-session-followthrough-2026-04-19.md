# SPT — pt 28: Next-session follow-through does NOT replicate (Q41)

Opened 2026-04-19 (~11:05 ET). Closes Q41 (candidate opened in [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26 §3.1]]). Scheduled-task run, autonomous.

Companion to [[small-pullback-trend-PLAYBOOK]]. **Question answered:** does a prior-day SPT label bias day-N+1's morning H1/H2/L1/L2 detections in the SAME direction, as Brooks *Trends* ch. 57 line 11 claims? **Answer: no — the effect points the opposite way at thin n, and the ALIGN bucket is ≤ baseline. Do NOT relax rule 6 for post-SPT mornings.**

Script: `~/code/aiedge/scanner/scratch/spt_next_session_followthrough_2026_04_19.py`. Output: `_out_spt_next_session_followthrough_2026_04_19.txt`.

---

## TL;DR

Brooks *Trends* ch. 57 line 11:

> "This type of trend can be so strong that there can be follow-through in the first hour or two of the next day, so traders should be looking to enter with trend on pullbacks after the open of the next day."

**Teaching does not replicate in aiedge data over the full DB (~8 months).** Three observations:

1. **9:30-10:30 ET window is structurally empty.** Only 9 morning-hour H1/H2/L1/L2 detections across the full DB; **zero** of them occurred after a polarity-labeled prior day. The scanner rarely fires its continuation setups in the first hour — there is no population to rehabilitate.
2. **9:30-11:00 ET window: ALIGN underperforms OPPOSED by ~1.85 R/trade.** ALIGN n=6 perR −0.012 vs OPPOSED n=9 perR +1.841 (raw) — opposite sign from Brooks' prediction. NO_SPT baseline is n=44 perR +1.352.
3. **9:30-11:30 ET window (adding the afternoon pre-band): ALIGN still sub-baseline.** ALIGN n=9 perR +0.548 vs NO_SPT n=104 perR +0.947 vs OPPOSED n=13 perR +1.275. ALIGN is the worst bucket at this window too.

**Cell sizes are thin (n=6-13 for ALIGN/OPPOSED)**, so the counter-signal is NOT strong enough to invert the rule 6 direction — but it is strong enough to say the Brooks-predicted positive signal is absent.

**No PLAYBOOK change.** Rule 6's `signal time ∈ [10:30 ET, 14:15 ET)` floor stands. Candidate Q41 resolved negative.

---

## §1. Design

### 1.1 Polarity definition on day-N

For each `(ticker, date)` cell:
- `n_long_spt` = # of `H1`/`H2` detections with `day_type ∈ {trend_from_open, spike_and_channel}` and `direction=long`
- `n_short_spt` = mirror for `L1`/`L2` and `direction=short`
- `polarity = 'long'` iff `n_long_spt ≥ 2` and strictly `> n_short_spt`; `'short'` mirror; `None` otherwise.

The `≥ 2` threshold matches rule 7's skip-first-detection mechanic (singleton SPT fires are unreliable).

### 1.2 Buckets on day-N+1

For each day-N+1 H1/H2/L1/L2 detection with signal time inside a given window:
- `ALIGN`   — day-N had polarity AND polarity matches detection direction
- `OPPOSED` — day-N had polarity AND polarity is the opposite direction
- `NO_SPT`  — day-N had no polarity (control group)

### 1.3 Windows swept

Brooks says "first hour or two". Tested three windows:
- `[09:30, 10:30)` — literal "first hour"
- `[09:30, 11:00)` — one-and-a-half-hour extension
- `[09:30, 11:30)` — outer "two hours" bound

### 1.4 Resolver

Market-on-next-bar-open entry, DB-stamped `stop_price` (signal-bar stop), 3R uniform target, honest-pessimistic resolver (bar-walk, no MFE/MAE fallback). Intentionally baseline — Q41 is a rule 6 **gating** question, not a rule 9 target choice.

### 1.5 Counterfactual comparisons

- `ALIGN vs NO_SPT` — does prior-day polarity add edge on same-direction morning signals?
- `OPPOSED vs NO_SPT` — does prior-day polarity predict reversal-signal edge (unintended counter-Brooks reading)?
- `ALIGN + rules 3/4/5 vs NO_SPT + rules 3/4/5` — does the effect survive urgency/gap/always_in filtering?

---

## §2. Population counts

Full-DB universe: 1,530 H1/H2/L1/L2 detections with chart_json. 849 unique `(ticker, date)` cells. **158 cells carry a polarity label** (101 long, 57 short). That's 18.6% of ticker-days — slightly below the 20% that Brooks cites for TFO frequency (ch. 57 line 9), consistent with the "SPT is a trend-from-open variant" framing.

### 2.1 Morning-window base rates

| Window | Total dets | ALIGN | OPPOSED | NO_SPT | % with prior-SPT |
|--------|-----------:|------:|--------:|-------:|-----------------:|
| 09:30-10:30 | 9 | 0 | 0 | 9 | **0.0%** |
| 09:30-11:00 | ~59 | 6 | 9 | 44 | 25.4% |
| 09:30-11:30 | ~126 | 9 | 13 | 104 | 17.5% |

The 9:30-10:30 base-rate of 0% is the first surprise. A ticker that just had an SPT day rarely fires an H1/H2/L1/L2 detection in the first hour of the next session at all — probably because these detections require enough prior-bar structure to assemble (pullback + signal bar), and the first hour is the pattern's **formation** phase, not its continuation phase.

Totals for 9:30-11:00 and 9:30-11:30 are approximate (reading from the cell sums in the output).

---

## §3. Results

### 3.1 9:30-10:30 ET window — no population

| Bucket | n | WR% | sumR | perR | DD | 95% CI |
|--------|--:|----:|-----:|-----:|---:|--------|
| ALIGN | 0 | — | — | — | — | — |
| OPPOSED | 0 | — | — | — | — | — |
| NO_SPT | 8 | 75.0 | +11.57 | +1.447 | −2.00 | [+0.32, +2.45] |

No polarity-labeled detections. Rule 6 cannot be relaxed for the first hour because there is **nothing there** — the answer isn't "yes/no/maybe" but "the cell is structurally empty."

### 3.2 9:30-11:00 ET window — ALIGN below baseline, OPPOSED above it

Raw (no upstream filters):

| Bucket | n | WR% | sumR | perR | DD | 95% CI |
|--------|--:|----:|-----:|-----:|---:|--------|
| ALIGN | 6 | 33.3 | −0.07 | **−0.012** | −4.00 | [−1.00, +1.32] |
| OPPOSED | 9 | 77.8 | +16.57 | **+1.841** | −1.09 | [+0.72, +2.81] |
| NO_SPT | 44 | 63.6 | +59.47 | **+1.352** | −16.00 | [+0.80, +1.91] |

ALIGN vs NO_SPT Δ = −1.364 R/trade. If the Brooks-predicted effect existed, ALIGN should be ≥ NO_SPT by ~0.3-0.5 R/trade. Observed sign is **opposite** and magnitude is larger than the adoption bar.

Rules 3/4/5 applied (urgency, gap_dir, always_in):

| Bucket | n | WR% | sumR | perR | DD | 95% CI |
|--------|--:|----:|-----:|-----:|---:|--------|
| ALIGN | 2 | 0.0 | −2.00 | −1.000 | −2.00 | [−1.00, −1.00] |
| OPPOSED | 5 | 100.0 | +14.66 | +2.933 | +0.00 | [+2.80, +3.00] |
| NO_SPT | 11 | 81.8 | +22.89 | +2.081 | −2.00 | [+1.16, +3.00] |

Upstream filters **amplify** the counter-direction: filtered ALIGN is 0% WR (n=2 is noise, but the sign agrees with the raw data). Filtered OPPOSED is n=5 and 100% WR — a tantalizing but too-thin reversal-morning cell.

### 3.3 9:30-11:30 ET window — same pattern, wider cells

Raw:

| Bucket | n | WR% | sumR | perR | DD | 95% CI |
|--------|--:|----:|-----:|-----:|---:|--------|
| ALIGN | 9 | 44.4 | +4.93 | **+0.548** | −5.00 | [−0.56, +1.88] |
| OPPOSED | 13 | 61.5 | +16.57 | **+1.275** | −4.09 | [+0.30, +2.32] |
| NO_SPT | 104 | 54.8 | +98.46 | **+0.947** | −40.95 | [+0.59, +1.30] |

At the outer "two hours" boundary, ALIGN is closer to baseline (Δ = −0.40 R/trade) but **still below** NO_SPT. OPPOSED remains above by +0.33 R/trade. Baseline perR at this window (+0.947) is below the entry-band perR (+1.269 full-DB 3R baseline from PLAYBOOK — pt 17 §1) — confirming rule 6's morning-cutoff is correctly shaped at the afternoon side.

### 3.4 LOO-week dominance

Skipped at primary window (9:30-10:30) — cell too empty. Not computed for the 11:00/11:30 windows because n ≤ 13 in ALIGN/OPPOSED; LOO-week dominance on a 6-13 trade cell is noise.

---

## §4. Interpretation

### 4.1 Why Brooks' teaching may not replicate

Three competing explanations, all consistent with the data:

**(a) Instrument difference.** Brooks is Emini-focused. The aiedge universe is single-name US equities, which react overnight to stock-specific news (earnings, analyst, sector rotation, overnight gaps) that the index doesn't see. An equity that trended hard on day-N will frequently gap-and-fade on day-N+1 when the news catalyst exhausts in after-hours. The Brooks "strong trend carries into next session" generalization is an index-level statistical regularity that doesn't survive transfer to single-name equities.

**(b) Scanner filter structure.** The scanner's H1/H2/L1/L2 setup criteria require a **pullback** to form (signal-bar structure presupposes a prior down-move in a bull context, and vice versa). A day-N+1 that opens with follow-through and trends cleanly in the prior direction **does not form the pullback needed to stamp H1/H2**. The pattern that Brooks describes — "enter with trend on pullbacks" — literally only materializes when the day-N+1 has a fade first (which IS the opposite-direction signal). So what Brooks calls "follow-through with pullback entry" the scanner labels as **OPPOSED on the fade bar**. This is the most likely reason OPPOSED outperforms ALIGN: the scanner's detection criterion is pattern-completion, not direction-continuation.

**(c) Overnight risk premium.** Post-SPT tickers carry concentrated overnight risk. The next-day open frequently gaps AGAINST the prior trend as overnight positioning unwinds. Morning detections that align with the prior polarity fire IN TO gap-fills (low edge), while opposed detections fire on the gap-continuation (high edge).

All three are consistent. Explanation (b) is the most scanner-specific and the most actionable: it says the Brooks teaching and the aiedge scanner are measuring different things. Brooks' "SPT followed by pullback entry next day" = continuation **with a fade** on the next day's open; the scanner's **OPPOSED** bucket IS that pattern at the fade bar — just labeled against the prior-day polarity.

### 4.2 The OPPOSED signal — real or thin-n artifact?

OPPOSED n=9 at 9:30-11:00 ET, perR +1.841, WR 77.8%. With rules 3/4/5: n=5, perR +2.933, WR 100%. Tempting to promote as "rule 6 reversal carve-out for post-SPT mornings," but:

- n=5-9 is dangerously thin. 95% CI on the raw cell is [+0.72, +2.81] — wide.
- The 100% WR on the filtered cell is likely a fluke — n=5 bootstraps bimodally.
- The filter cross-section is already tiny (rules 3/4/5 inside morning window after polarity OPPOSED is a 4-condition intersection).

**Monitor, don't act.** Add to "sizing tag candidates" rather than PLAYBOOK filter. Revisit at n_opposed ≥ 25 (probably 6-12 months of further data given current rate of ~9 per 8 months). Opens **Q46** (OPPOSED post-SPT morning reversal cell) as a low-priority monitoring item.

### 4.3 What this means for rules 6 and 9

- **Rule 6 stands unchanged.** Brooks' next-session follow-through does not replicate; the 10:30 ET morning floor is Brooks-silent but empirically sound. The counter-Brooks finding (OPPOSED outperforms ALIGN) does NOT invert it because the ALIGN cell is sub-baseline and OPPOSED is thin.
- **Rule 9 unchanged.** This was a gating question, not a target question.
- **No new filter rule proposed.** Q46 (OPPOSED cell monitoring) stays as a data-accumulation observation, not a rule candidate.

### 4.4 Brooks-text reconciliation

Section §3.1 of [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26]] framed this as a Brooks-grounded untested candidate. The conclusion now is:

> Brooks' next-session-follow-through teaching from ch. 57 line 11 is correct at the Emini index level (per Brooks' own chart annotations) but does not transfer to single-name US equity continuation setups. The transfer failure is mechanistic — Brooks describes continuation-with-pullback, the scanner measures pullback-as-precondition, and the two diverge on the asset class's overnight risk signature.

This is a useful result FOR THE META-AUDIT: it identifies the first concrete Brooks→aiedge transfer failure in the 27-note arc. Every previous Brooks-grounded rule replicated; this one does not. Future Brooks candidates (Q42 2× climactic burst, Q43 first MA-gap bar) should be tested with **stronger skepticism about asset-class transfer** given pt 28's finding.

---

## §5. Limitations

1. **Thin cells** — ALIGN n=6-9, OPPOSED n=9-13 at the 11:00 ET boundary. Bootstrap CIs are wide (width 1-3 R/trade). A 0.5 R/trade effect would need n ≈ 30-40 to resolve; current cell is decisively inadequate.
2. **Polarity threshold robustness not tested.** Used `≥ 2` for SPT polarity; could be `≥ 3` (stricter, fewer cells) or `≥ 1` (looser, more cells but includes singletons pt 7 already flags as weak). Picked 2 on the rule-7 analogy; a sensitivity sweep is future work if this is re-opened.
3. **Urgency ≥ 6 TFO-specific threshold not applied separately.** The rules-3/4/5-filtered variant uses rule 3's current form. A pure "polarity adds on top of rule 3" isolation is not done.
4. **No cross-ticker contamination check.** A heavy breadth day (where many tickers were SPT) could correlate the ALIGN cells across tickers. LOO-symbol would catch this; not run because the population is too small.
5. **Target choice.** Used 3R uniform. Hybrid rule 9 would likely lift NO_SPT's baseline but not change the sign of ALIGN/OPPOSED differentials. Future re-run if population grows should use hybrid.

---

## §6. Summary for PLAYBOOK

No rule change. Q41 closes **negative**: Brooks' next-session follow-through teaching does NOT replicate in the aiedge scanner data. Mechanism is a scanner-pattern/Brooks-teaching ontology mismatch (the scanner's pullback-as-signal-bar construction inverts the label of Brooks' "continuation with pullback entry"). Keep rule 6's 10:30 ET morning floor unchanged.

Opens **Q46** (post-SPT OPPOSED morning reversal cell) as a low-priority monitoring item.

---

## §7. Related

- [[small-pullback-trend]] — parent concept doc (Brooks definitions, scanner implementation).
- [[small-pullback-trend-PLAYBOOK]] — canonical policy stack; no change from this note.
- [[small-pullback-trend-INDEX]] — reading order; this is pt 28.
- [[small-pullback-trend-brooks-source-cross-reference-2026-04-19]] — pt 26, which flagged Q41 as a candidate in §3.1.
- [[small-pullback-trend-stop-and-timing-2026-04-18]] — pt 2, which filed Q3 (next-day follow-through not testable with then-available data).
- [[small-pullback-trend-empirics-2026-04-18]] — pt 1, which confirmed Brooks' 14:00 ET pullback timing; Q41 is a **second** empirical test of a Brooks-ch-57 claim, and the first one to return negative.

Script: `~/code/aiedge/scanner/scratch/spt_next_session_followthrough_2026_04_19.py` (240 LOC). Output: `_out_spt_next_session_followthrough_2026_04_19.txt`.

---

## §8. Methodology note

This is the **first clean-negative result** in the 28-note SPT arc. All 27 prior notes either (a) confirmed a Brooks teaching empirically, (b) found a Brooks-silent pure-empirical rule, or (c) rejected a speculative addition that was NOT Brooks-taught. Pt 28 is the first "Brooks explicitly said X, the data says not-X in this universe."

This matters for future pacing of the Q42/Q43 candidates from pt 26 §3.2-§3.3: both are exit-side Brooks teachings that might plausibly fail the transfer test the same way Q41 did. The adoption bar for Brooks-grounded exit overlays should now include a mandatory replication-check step (explicit ALIGN-vs-NO_SPT-style bucket comparison) before being promoted from candidate to script.
