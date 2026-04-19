# SPT — pt 31: Brooks→aiedge transfer failure taxonomy (synthesis)

Synthesis pass after the pt 26 §3 candidate trio (Q41, Q42, Q43) all resolved negative in [[small-pullback-trend-next-session-followthrough-2026-04-19|pt 28]] / [[small-pullback-trend-climactic-exit-2026-04-19|pt 29]] / [[small-pullback-trend-first-ma-gap-exit-2026-04-19|pt 30]]. Pt 30's closer declared "Brooks-source well effectively exhausted for SPT US single-name equities"; this note crystallizes *why* by giving the three failure modes a taxonomy and a vetting checklist, so a future candidate Brooks rule can be pre-screened before a full backtest.

Run 2026-04-19, autonomous scheduled-task. Pure-synthesis pass — no new Python, no new data reads. **No PLAYBOOK change.** Opens nothing new; closes a documentation gap in the arc.

Companion to [[small-pullback-trend-PLAYBOOK]] and [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26]].

---

## TL;DR

- **Three failure modes observed**, each with a distinct diagnostic:
  1. **Inversion** (Q41) — scanner labeling is orthogonal-or-opposite-polarity to Brooks's discursive labeling; ALIGN and OPPOSED swap roles relative to the book's teaching.
  2. **Rarity** (Q42) — Brooks's signature fires structurally rarely on the stack-selected population because an upstream filter pre-empts the window it needs.
  3. **Emptiness** (Q43) — Brooks's *conditional* ("do X given Y is currently true") collapses because Y and X are anticorrelated on the stack-selected survivors; the signature fires but never in the regime that makes it useful.
- **The surviving Brooks content is concepts, not mechanics.** Rules 1–7, 9, 11 are all Brooks-grounded (pt 26 audit) and positive. Rules 8, 10 are pure-empirical. The three negatives were all *mechanical* (exit timing, follow-through timing) — Brooks's discretionary tape-reading teachings that depend on watching a single instrument bar-by-bar.
- **Why the asymmetry.** Concepts (what IS an SPT; what makes an H2 vs a FL2; when to demand urgency) describe properties of the bar/window Brooks saw on his chart. Those properties port to the scanner because the scanner computes the same properties. Mechanics (when to exit, when to follow through) describe actions Brooks takes conditional on subsequent bar formation — and those subsequent bars, on the scanner's selected population, have different statistics than on Brooks's raw chart stream because rules 1-9 have already removed most of the pathology.
- **Proposed vetting checklist** (§3) — three zero-compute diagnostics to run on any future Brooks-rule candidate *before* investing in a backtest. All three would have predicted the Q41/Q42/Q43 failures without running the scripts.
- **Next-direction priorities** (§4) — +R paths remaining in the arc are (a) scanner-side schema enrichments (Q1 raw scores, Q5 cross-asset, Q25 label stability), (b) conditional-rule maturation (Q33 rule-10 at n_shorts ≥ 80, Q44 rule-11 Δ stability weekly, Q45 L2-short monitoring, Q46 OPPOSED morning monitoring). No +R expected from more Brooks-source excavation on SPT US single-names.

---

## §1. The taxonomy

Three failure modes, each illustrated by the closing note that produced it.

### §1.1 Inversion — Q41, [[small-pullback-trend-next-session-followthrough-2026-04-19|pt 28]]

**Brooks teaching** (ch. 57 line 11): after an SPT day, day N+1 typically opens with continuation in the same direction for the first 30–60 minutes.

**Scanner reading**: at 9:30–11:00 ET on day N+1, the ALIGN bucket (H1/H2 longs after a bull SPT; L1/L2 shorts after a bear SPT) had n=6 perR −0.012, UNDERPERFORMING both NO_SPT (n=44 perR +1.352) and OPPOSED (n=9 perR +1.841).

**Diagnostic**: the perR sign flipped relative to Brooks's prediction. Not "smaller than expected" — literally *opposite direction*.

**Mechanism**: the scanner's H1/H2/L1/L2 setups fire at a **pullback** bar, not at a continuation bar. Brooks's "continuation with pullback entry" is what the scanner labels as its OPPOSED fade-bar (the bar that pulls back against the prior bias, which then resumes). So the noun "continuation" in Brooks's text maps to the adjective "OPPOSED" in scanner-speak. The teaching is correct — we just labeled it wrong in the backtest.

**Test that would have caught it cheap**: read Brooks's definition of the trigger bar, then check whether it's the bar where the scanner fires (trigger = signal) or the bar before (trigger = prior-bar polarity, signal = pullback). If the scanner fires at the pullback, Brooks's "continuation" maps to scanner's OPPOSED, not ALIGN.

### §1.2 Rarity — Q42, [[small-pullback-trend-climactic-exit-2026-04-19|pt 29]]

**Brooks teaching** (ch. 57): in the final hour of a trend day, a bar that's ≥ 2× the average bar size and makes a new HoD/LoD on its second bar is a climactic-exit signal.

**Scanner reading**: at Brooks's canonical cell (14:00–15:00 ET, range ≥ 1.5× trailing-20-bar median, 2nd bar makes new HoD/LoD), the signature fires on **2/101** baseline trades. 18-cell parameter sweep never exceeds +0.004 R/trade delta.

**Diagnostic**: `n_fired / n_baseline < 5%`. The overlay is a no-op on the selected population.

**Mechanism**: only 37% of baseline trades even have a post-entry bar in 14:00–15:00 ET. Rule 6 truncates entries at 14:15 ET; hybrid rule 9 resolves many trades before 14:00 via target or stop. Brooks's signal is real; it just doesn't co-occur with the trades the filter stack keeps.

**Test that would have caught it cheap**: before running the overlay, query `count(baseline_trades where overlay_eligible_window) / count(baseline_trades)`. If < 20%, the overlay is structurally rare and even a large perR effect per-fire would round to zero book-wide. Skip unless the overlay IS the study (e.g. testing rarity itself).

### §1.3 Emptiness — Q43, [[small-pullback-trend-first-ma-gap-exit-2026-04-19|pt 30]]

**Brooks teaching** (ch. 47 line 45): exit at the first MA-gap bar in a strong trend **when the trade is currently in a profitable swing**.

**Scanner reading**: at Brooks's canonical cell (threshold +1.5R, EMA20, exit at gap-bar close), the signature fires on **0/101** baseline trades. 12-cell sweep never positive; best non-zero cell is −0.015 R/trade at n_fired=1.

**Diagnostic**: `n_fired = 0` at the canonical cell, but `n_fired_unconditional > 0` (16/101 trades see *an* MA-gap bar; 0/16 are ≥ +1.5R at the gap-bar close).

**Mechanism**: Brooks's rule is CONDITIONAL — "exit at MA-gap bar **IF** currently profitable." On the scanner's stack-selected population, the condition (currently profitable) and the trigger (first MA-gap bar) are anticorrelated: by the time a pullback is deep enough to print an MA-gap bar, the same pullback has already pushed mtm to −1.66R mean. Of the 16 trades that see a gap bar, 0 are profitable at that moment. The conditional is structurally empty even though both halves exist in isolation.

**Test that would have caught it cheap**: compute the joint distribution `P(condition_Y AND trigger_X)` on the baseline. Mechanically: walk trades, find the first bar where X fires, check whether Y holds at that bar. If `count(Y_at_X_fire) / count(X_fires) < 10%`, the conditional is structurally empty.

### §1.4 The fourth mode that didn't happen

A possible fourth mode would be **polarity-flipped magnitude**: the teaching fires on the expected bars with the expected frequency, but the sign of the effect is opposite (e.g. Brooks says "exit"; backtest says "add"). Q41 technically qualifies but ALIGN's mean ≈ 0 was also a timing artifact (scanner fires off-window), so it's cleaner to call it inversion. A clean polarity-flip case hasn't shown up in 30 notes.

One reason: Brooks's **concepts** (signs-of-strength, always-in, signal-bar quality) have been vetted rule-by-rule in pt 26 and on individual ports back to pt 1. Every one that ported mechanically produced the expected direction. The mechanics-vs-concepts split seems to be where the failure mode lives.

---

## §2. Why the asymmetry — concepts port, mechanics don't

This is the generalizable lesson. Framing it explicitly:

- **A Brooks concept** describes a property of the bar or window the scanner has in front of it NOW. Examples: "H2 = second pullback in a bull leg" (scanner computes this from window density); "SPT day = trend from open with shallow pullbacks" (rule 2); "signs of strength = trend bars with with-trend close direction" (rule 1's setup filter); "urgency measures bar texture, not regime maturity" (rule 3). These port because the scanner computes the same property from the same bar data.

- **A Brooks mechanic** describes an action conditional on bars the scanner *hasn't seen yet* (post-entry bars, next-session bars). Examples: "exit at first MA-gap bar if profitable" (Q43); "next-session follow-through 9:30–10:30 ET" (Q41); "2× climactic last-hour exit" (Q42); "move stop to sig-bar-high breakeven" (pt 20's breakeven-trap warning). These port BADLY because the post-entry bar distribution on stack-selected trades is not the post-entry bar distribution on Brooks's raw tape.

- **Why the distribution differs**. Brooks watches one instrument continuously and enters trades of mixed quality. The scanner applies 9 stacked filters, each cutting ~30-50% of the population. The surviving population is the top 1-2% of raw trades by filter quality. Its post-entry bars look qualitatively different from Brooks's raw-tape post-entry bars — MFE/MAE distributions are shifted right, time-to-target is compressed (hybrid rule 9 resolves early), and rare-tape-reading signals (climactic bursts, MA-gap bars) fire at different base rates because the surviving trades are "clean."

- **The Brooks-concept surplus is already consumed.** Pt 26's rule-by-rule audit found 9/11 PLAYBOOK rules with STRONG or PARTIAL Brooks grounding. The Brooks-mechanic well produced 0/3 additions in the Q41/Q42/Q43 trilogy. The asymmetry is not accidental — it tracks the concept-vs-mechanic split.

**The lesson is not "Brooks is wrong."** Brooks's mechanics are correct for a single-instrument discretionary trader. They're just the wrong tool for a filter-heavy multi-symbol backtest, because the statistical object they operate on (post-entry bar distribution on a single-instrument stream) is not the one the backtest has (post-entry bar distribution on a stack-selected multi-symbol survivor set).

---

## §3. Vetting checklist for future Brooks-rule candidates

Before writing a `spt_<topic>_2026_<date>.py` for a new Brooks teaching, run these three zero-compute diagnostics. All three would have predicted Q41/Q42/Q43 negative. Two would have predicted each.

### Check 1 — Label-polarity reconciliation

**Question**: does Brooks's trigger bar coincide with the scanner's signal bar, or is one bar off?

**How to run**: in one sentence, write down what Brooks says the trader observes at the moment of entry. Then in one sentence, write what the scanner records as the signal bar. Compare.

- If Brooks's "continuation bar" = scanner's signal bar → ALIGN is correct.
- If Brooks's "pullback bar" = scanner's signal bar → Brooks's "continuation" = scanner's OPPOSED. Invert the label before backtesting.
- If Brooks describes watching N bars of price action before a decision, the scanner's single-bar signal is probably a different object entirely — step back and rethink whether the teaching is compatible.

**Would have caught**: Q41 (Brooks's "continuation with pullback entry" = scanner's OPPOSED). The first run should have tested OPPOSED, not ALIGN.

### Check 2 — Base-rate on selected population

**Question**: how often does the trigger signature fire on the baseline trade population, before any conditional?

**How to run**: write the trigger as a bar predicate (e.g. `bar.h < EMA20` for an MA-gap bar after a long entry). Walk the baseline trades, count trades where *any* post-entry bar satisfies the predicate. Compute `n_fired_unconditional / n_baseline`.

- `> 50%` → signature is common; a conditional filter will meaningfully narrow it.
- `10–50%` → signature is selective; ok to proceed.
- `< 10%` → signature is structurally rare; even a large per-fire effect will round to noise book-wide. Skip unless rarity is the study.

**Would have caught**: Q42 (`n_fired_unconditional = ~2/101 at canonical cell` — rarity). Arguably also Q43 at looser sweep cells (16/101 ≈ 16%, borderline), but the conditional gate is what killed it — see Check 3.

### Check 3 — Conditional joint-probability

**Question** (only if Brooks's rule is conditional — "do X given Y"): does the condition Y hold at the moment the trigger X fires, on the stack-selected population?

**How to run**: extend Check 2. At each trade where X fires, record Y's state. Compute `n_Y_and_X / n_X`.

- `> 50%` → conditional is well-populated; backtest meaningful.
- `10–50%` → conditional is selective but alive.
- `< 10%` → conditional is structurally empty. The rule cannot help because the regime it targets is absent on survivors.

**Would have caught**: Q43 (`P(currently ≥ +1.5R | first MA-gap bar after entry) = 0/16 = 0%`). This is the most specific test of the three and closes the conditional-emptiness failure mode cleanly.

### Running the checklist

All three checks run from the C0 baseline trade list that every SPT script already constructs. A single helper `spt_brooks_vetting.py` with three functions (`check_label_polarity(candidate)`, `check_base_rate(candidate)`, `check_joint_probability(candidate, condition)`) would reduce future Q-closings from half-day script runs to minutes. Flag this as the next scanner-scratch utility to build when Brooks candidates are considered again. (Not Python-I-write today — note this as a candidate utility for when the next Brooks candidate actually appears.)

**A note on false negatives**: the checklist rejects candidates conservatively. A candidate that passes all three checks is not guaranteed positive — rule 8 (`market_ord > 2`) passed Brooks-source audit (PARTIAL grounding, pt 26) and Check 2 (fires ~40% of the time) but was net-negative at full stack (retired in pt 14). The checklist prevents wasted compute on *structurally broken* candidates; it doesn't promote *structurally alive* candidates to production without the backtest.

---

## §4. Next-direction priorities for the SPT arc

Given "Brooks-source exhausted," where does +R come from next?

### §4.1 Scanner-side schema enrichments (blocked on migrations)

- **Q1 — raw component scores**. Scanner currently stores composite scores; not per-component (bar-quality, window-density, alignment, etc.). A `spt_component_raw ≥ 2.0 vs < 2.0` bucketed study is blocked. Listed since pt 0; still the single largest blocked direction. Would unlock per-component ablation and per-segment re-optimization (Q38, Q34).
- **Q25 — day_type label stability**. Scanner stores the live day_type label at detection time, but not the history. Can't distinguish "label has been stable since 10:00 ET" from "label just flipped 5 minutes ago." Pt 15 showed rule 2's live label is 47% noisy on breadth days; a stability requirement is structurally impossible today.
- **Q5 — cross-asset SPT**. Brooks text is Emini-focused; the scanner is tuned to US single-names. Multi-asset expansion (futures, forex) has not started. Listed on memory as project_multi_asset_scanner.

**Action**: these are schema work, not research. Add to the scanner's next migration batch rather than generating more Q-numbered notes.

### §4.2 Conditional-rule maturation (in-flight)

- **Q33 revisit** — rule 10 (short-side swing2 stop) was adopted as conditional at n_shorts=35 with wide CI. Revisit at **n_shorts ≥ 80** (~6–9 more months of data). Promote to core stack if +R holds ≥ +0.15 / demote if decays below +0.10.
- **Q44** — rule 11 solo Δ stability. Measured +0.31 at pt 23 → +0.21 at pt 27 in a 5h window. Weekly tracking proposed for 4 weeks. *First weekly data point not yet captured.* Low-cost script run: reproduce pt 27's rule-11 solo delta at end-of-week and record.
- **Q45** — L2-short sparse cell (n=3). Monitor only, not a rule question. Revisit at n_L2_short ≥ 12.
- **Q46** — OPPOSED morning reversal cell (n=9 at 9:30–11:00 ET, perR +1.841 raw / +2.933 with rules 3/4/5 at n=5). **Tempting but dangerously thin.** Revisit at n_opposed ≥ 25 (probably 6–12 months of further data). Promote to sizing-tag at n=25 if edge survives; never promote to filter without CI tightening.

**Action**: these are data-growth-gated. The right cadence is one weekly Q44 reading and checkpoints at cell-growth milestones for the rest, not continuous research pressure.

### §4.3 What won't generate +R

- More Brooks-source excavation on SPT US single-names. Pt 26 audited the three canonical chapters (47, 57 *Trends*; 51 *Reversals*); the three mechanical residuals all closed negative. Unlikely there's a 4th mechanic that ports.
- Scanner hyperparameter tuning without a schema change. The existing 11 rules are CI-tight. Further filter tightening trades trade count for noise-level perR improvements.
- More target/stop sweeps. Pt 17 (target) and pt 20 (stop) sweeps were exhaustive; hybrid rule 9 + sig-bar stop is near-optimal.

### §4.4 Where +R probably lives

- **Schema migration unblocks Q1/Q25** → per-component + label-stability rules → +R expected, magnitude unknown.
- **Multi-asset (Q5) expansion** → parallel SPT edge in forex/futures if Brooks concepts generalize → requires scanner port first.
- **n-growth milestones** (Q33, Q46) → conversion of "conditional" rules to "core" rules after CI tightens → marginal perR gains (+0.05–0.10 R/trade).

None of these produce a +0.3 R/trade weekend-study gain like the pt 22–23 opp_tail discovery did. Expect the SPT arc to slow from ~1 note/day to ~1 note/week as the well shifts from discovery-mode to monitoring-mode.

---

## §5. What this note doesn't do

- Does NOT change the PLAYBOOK. Rules 1–11 unchanged; sizing tags unchanged; economics unchanged.
- Does NOT open a new Q. The three failure modes are *documented*, not *hypothesized* — the hypothesis work is already done in pts 28–30.
- Does NOT propose a Python run. The checklist in §3 is a design for a future utility; flagging for build-when-needed, not build-now.
- Does NOT retire the sibling notes. Pts 28/29/30 stay as-is; this note is a third-level synthesis, not a replacement.

---

## §6. What's next, mechanically

When the next Brooks-teaching candidate comes up (from pt 26 cross-reference, from a new Brooks chapter, or from a user read), route it through §3's checklist FIRST. If it fails Check 1 — re-label and re-check; if it fails Check 2 or Check 3 — document the failure-mode-class in one paragraph and don't write the script. The documentation-only failure note is a valid arc contribution (pt 30 style, abbreviated).

When the arc's center of gravity shifts to scanner-side work (Q1/Q25 post-migration, Q5 post-expansion), this note becomes the forward-reference for "how we vetted the Brooks transfer before moving to scanner enrichments."

When Q44's weekly rule-11 tracking produces its 4-week readings, revisit §4.2 with the noise-floor estimate. If ±0.15 R/trade confirms, the adoption threshold for conditional rules tightens from "+0.20 R/trade, CI tolerated" to "+0.35 R/trade, CI must exclude noise floor." That's a meaningful governance change even without a new rule.

---

## §6a. TL;DR in plain English

We have a 9-rule playbook for trading "small pullback trend" days that makes about **+1.84 of risk per trade** (bet $100, average win/loss is +$184).

For months we've been pulling new rules from Al Brooks's books. This week we finished the last three candidates he gave us. **All three failed.** Each for a different reason:

1. The first rule pointed the wrong way — we called the bar "buy" when Brooks meant "sell" (different labeling conventions).
2. The second rule almost never happened — Brooks's signal fires in the last hour, but our playbook already filters out late-day trades.
3. The third rule was a paradox — "exit if you're winning when the warning appears," but the warning only appears after the trade is already losing.

**Big picture:** Brooks's ideas about *what makes a good trade* work great (that's why 9 of our 11 rules come from him). Brooks's ideas about *when to get out of a trade* don't work for us — because our filters already removed most bad trades, so his exit signals fire on a picked-over group that behaves differently than his raw charts.

**Now what:**

- **Stop** mining Brooks's books for more rules on this setup. That well is dry.
- **Next Brooks candidate gets a 3-question pre-check** before any code gets written. Would have saved a week of compute on these three.
- **Real gains from here** come from (a) smarter scanner data, (b) waiting for more trades to tighten the existing rules, (c) expanding to futures and forex.
- **Nothing to change today.** The trading rules you're using still work. This was a "sharpen the saw" week, not a "cut a new tree" week.

---

## §7. Generalization beyond SPT

This taxonomy probably extends to any Brooks→scanner port on any pattern, not just SPT. The concept-vs-mechanic split is structural, not SPT-specific. A future scanner pattern (say, always-in reversals, FL2, opening-range failure) doing a Brooks cross-reference should route through the same checklist.

**Suggested vault placement when that happens**: promote §3's checklist to `vault/Meta/brooks-transfer-vetting-checklist.md`, referenced from this note and from any future pattern's own Brooks-cross-reference note. Current location (in concept-specific file) is fine until the second pattern needs it; premature promotion would orphan the context.

---

## §8. Closing

Pts 28/29/30 produced three negative results and pt 30's closer called the Brooks well exhausted. Without this synthesis note, that conclusion sits in three places with different framings and no generalizable lesson for future ports. With this note, the arc has:

- a named taxonomy (inversion / rarity / emptiness),
- a zero-compute vetting checklist that would have pre-screened all three,
- a clear statement of where the remaining +R lives (schema, n-growth, multi-asset — not more Brooks excavation),
- and a forward-pointer for when the next Brooks-port opportunity arises.

The PLAYBOOK is still the operational document. This note is the *meta* layer — how we decide whether to extend the PLAYBOOK when new Brooks material appears.

No PLAYBOOK change. Opens nothing. Closes the documentation gap between pt 30's conclusion and the arc's governance layer.
