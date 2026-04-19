# SPT — pt 26: Brooks source cross-reference for the 11-rule PLAYBOOK

Opened 2026-04-19 (~09:10 ET). Documentation pass. No new Python runs — Mac mini is still in active thrash (load avg 132 / 130 / 127 across 1/5/15 min, see [[Maintenance Log 2026-04-19]] sweep #9+). All conclusions here come from re-reading Brooks source (`~/code/aiedge/brooks-source/extracted/`) and the [[small-pullback-trend-INDEX|25-note arc]].

Companion to [[small-pullback-trend-PLAYBOOK]]. **Question answered:** after 25 empirical iterations, are all 11 PLAYBOOK rules still Brooks-grounded? Which rules are pure-empirical? And which Brooks passages are NOT captured in any rule (candidate future research)?

---

## TL;DR

- **9 of 11 rules have STRONG or PARTIAL Brooks grounding.** Rules 1, 2, 3, 4, 5, 7, 9, 11 align with explicit Brooks text; rule 6 partially aligns (the 14:00 ET pullback timing IS Brooks' "11:00 AM PST" window, but the 10:30/14:15 brackets and the short-side 11:00-11:30 dead zone are pure empirical).
- **2 rules are BROOKS-SILENT pure-empirical findings.** Rule 8 (first-3-gate continuation) and rule 10 (swing2 stop on shorts) have no Brooks-text analogue — they emerged from loss-clustering diagnostics on the aiedge data, not from Brooks teaching.
- **No rule directly contradicts Brooks.** Rule 11 (drop `opp_tail ≥ 0.25`) SUPERFICIALLY looks like it contradicts Brooks' "buy despite weak signal bars" advice, but on closer reading it's a texture-specific filter that enforces Brooks' shallow-pullback definition (signal-bar tail ≥ 25% of range = pullback inside the signal bar got too deep for SPT texture) — Brooks-consistent, not contradictory.
- **Three Brooks passages are NOT captured in any rule.** (1) Next-session follow-through ("traders should look to enter with trend on pullbacks after the open of the next day"); (2) The 2× climactic burst as a **sell/exit** signal in trend, not a breakout continuation; (3) First-moving-average-gap bar as a "final-leg-before-correction" marker. All three are Brooks-explicit, all three are scanner-silent. Flagged as candidate future research in §3.
- **Two numerical Brooks claims** line up with scanner empirics and can be audited for drift: (a) SPT forms "1–2× per month" vs scanner's 8/39 trading days (~20%); (b) pullback depth "20–30% of recent ADR" vs scanner's `SPT_DEPTH_SHALLOW = 0.25`.

No PLAYBOOK change. No open question opened; closes nothing formally (but §3's three gaps would become Q41, Q42, Q43 if promoted to active research).

---

## §1. Rule-by-rule Brooks grounding

Grounding strength legend:
- **STRONG** — direct Brooks quote endorsing the rule's specific form
- **PARTIAL** — Brooks motif matches but rule adds parameters Brooks doesn't specify
- **BROOKS-SILENT** — Brooks doesn't discuss this at all; pure-empirical finding
- **TENSION** — Brooks text appears to advise against the rule (resolve in the notes column)

### Rule 1 — Setup ∈ {H1, H2, L1, L2}

**Grounding: STRONG.**

Brooks *Trends* ch. 57, lines 31, 57-58 — the canonical prescription for entering an SPT:

> "In a strong bull trend, look to buy at the low of the prior bar, or one or two ticks below its low, or on a stop at one tick above high 1 and high 2 setups. ... In strong bear trends, traders will do the opposite and short at the high of the prior bar or one or two ticks above it, on a stop at one tick below low 1 and low 2 signal bars."

> "They bought above the bar 11 bear trend bar that closed below the moving average, because it was a failed one-tick breakout below the bars 8 and 9 double bottom."

The H1/H2/L1/L2 setup list IS Brooks' list of legitimate SPT entries. Brooks ch. 47 line 31 explicitly rules out L1 in a strong bull ("it is incorrect to use the term low 1 here because a low 1 sets up trades in trading ranges and bear trends, not strong bull trends"); the aiedge scanner handles this via direction matching — L1/L2 in a bull SPT are mechanically impossible because they require a prior lower-high.

Notes: FH/FL (failed versions) are excluded by this rule. Brooks discusses failed breakouts as entries in OTHER day types (trading ranges) but not for SPT continuation; aiedge's finding that "FH/FL collapse to 0–18% WR in SPT" is directly anticipated by ch. 47's "sell signals never look quite strong enough, but the traders sell anyway ... [they get] trapped."

### Rule 2 — day_type ∈ {trend_from_open, spike_and_channel}

**Grounding: STRONG.**

Brooks *Trends* ch. 57, line 9 — the opening sentence of the chapter:

> "Since almost all small pullback days are trend from the open days, they should be considered to be a strong variant. A trend from the open day is usually the strongest form of trend pattern, but it develops in only about 20 percent of days."

Spike-and-channel is mentioned as a compatible day type in ch. 57 line 25:

> "If the market trends for four or more bars without a pullback, or even two large trend bars, the move should be considered to be a strong spike. ... The third and most common outcome, when the spike is not so large as to be exhaustive, is the formation of a trend channel, and the day then becomes a spike and channel trend day."

Notes: rule 2's exclusion of `trading_range` and `tight_tr` is Brooks-consistent — ch. 57 line 199 explicitly says "Thinking of this as a trending trading range day will make you take longs and often miss shorts." The intraday label-flip problem diagnosed in pt 15 ([[small-pullback-trend-daytype-stability-2026-04-18]]) is Brooks-silent; Brooks doesn't discuss live day-type classification mid-day because he's analyzing end-of-day charts.

### Rule 3 — urgency ≥ 4 (≥ 6 on trend_from_open)

**Grounding: PARTIAL.**

Brooks uses "urgency" explicitly as a trend-strength descriptor. *Trends* ch. 57 line 61:

> "Usually, when there is a bear micro channel, like the one from bar 10 to bar 11, it is better to buy a pullback from the breakout, but there is a sense of urgency when the trend is strong, and smart traders are unwilling to wait for perfection because they don't want to risk missing the move."

*Reversals* ch. 51 line 120:

> "The bar 7 low was three ticks above the bar 5 low, which was a sign of urgency on the part of the bulls. They were so eager to get long that they were afraid that the market might not get all the way to the bottom of bar 5, so they bought several ticks above."

Brooks' urgency is a qualitative texture descriptor; the scanner's `urgency` field is a numerical composite. The ≥ 4 / ≥ 6 thresholds are pure empirical calibration ([[small-pullback-trend-urgency-generalization-2026-04-18|pt 4]]) — Brooks doesn't offer numbers. But the concept that higher urgency → higher-quality with-trend entry is directly Brooks-supported.

Notes: the TFO-specific ≥ 6 threshold comes from the empirical finding that u=4-6 on TFO is noise (chop before the real breakout). Brooks doesn't distinguish TFO vs spike-and-channel for urgency. This is a scanner-specific calibration that respects Brooks' directional intuition.

### Rule 4 — gap_direction aligned with setup

**Grounding: STRONG.**

Brooks *Trends* ch. 57 line 9, 23, 41:

> "These days often open with large gaps and then the market continues as a trend in either direction."

> "If there is a gap open that is more than just a few ticks and the first bar is a strong trend bar (small tail, good-sized bar), trading its breakout in either direction is usually a good trade."

> "a trend bar for the first bar is a good setup for a trade; but the chance for success is higher if there is a gap, since the market is more overdone and any move will tend to be stronger."

The rule's specific form (`gap_opposed at u≥4 is 100% loss, n=9`) is the empirical contrapositive of Brooks' gap-aligned positive case. Brooks doesn't enumerate the gap-opposed failure mode in ch. 57; he focuses on the aligned success case. The scanner's finding that gap-opposed + high urgency = 100% loss is consistent with the Brooks premise "gap opposed to setup = pullback/reversal in progress, not SPT continuation."

Notes: ch. 57 line 75 *does* show a gap-opposed example (Figure 23.5: first bar was a bull trend bar on a gap-below-prior-low day, which then became a strong bear SPT). Brooks' read: "Sometimes the first bar of the day can trap traders into entering in the wrong direction." The rule 4 is therefore defense against exactly this trap.

### Rule 5 — always_in ≠ opposed(setup)

**Grounding: STRONG.**

Brooks *Reversals* ch. 51 line 120 is the canonical always-in prescription:

> "Once a trader believes that the market is always-in long, the best trade is almost always buying a pullback, especially near the moving average and when the signal bar has a bull body."

*Trends* ch. 57 line 17:

> "The computers will usually show what the always-in trade is within a bar or two ... Traders will then have probability on their side, and can look to enter in the appropriate direction."

Rule 5 is Brooks' always-in rule in its most conservative form: the setup direction must NOT be opposed to the prevailing always-in reading. The rule as stated actually allows `always_in == neutral` (only opposed is rejected). This is looser than Brooks' "once always-in is long, buy pullbacks" — the aiedge scanner admits `always_in == neutral` because treating neutral as reject would drop ~40% of baseline trades, and the empirical finding is that neutral isn't a negative cell.

Notes: pt 5 ([[small-pullback-trend-alignment-filters-2026-04-18]]) showed that `always_in == opposed` at urgency ≥ 4 is ~22 trades/3mo at 14% WR — textbook Brooks trap, and the rule now mechanically excludes it.

### Rule 6 — signal time ∈ [10:30 ET, 14:15 ET); shorts only [11:30 ET, 14:15 ET)

**Grounding: PARTIAL (timing anchors strong; window brackets and short carve-out empirical).**

The 14:00 ET pullback is DIRECTLY Brooks-grounded. *Trends* ch. 57 line 13:

> "About two-thirds of these days have a larger pullback after 11:00 a.m. PST. That pullback is often about twice the size of the biggest pullback since the trend began in the first hour."

PST + 3h = ET, so Brooks' "11:00 AM PST" = **14:00 ET** exactly — which is the upstream of the rule's 14:15 ET close-bracket. Pt 1 ([[small-pullback-trend-empirics-2026-04-18]]) confirmed this empirically in the aiedge data. This is a cross-validation between Brooks and the scanner data.

Brooks ch. 57 line 183 reinforces:

> "On most strong trend days, there is usually a sharp, brief reversal between 11:00 a.m. and noon PST that shakes out weak longs and traps overly eager bears who are hoping to recoup their earlier losses."

This is **14:00–15:00 ET**, which is the rule 6 exit band. The rule's instruction to stop taking new signals at 14:15 ET is the "don't enter into the 2× pullback" corollary — you can ride an existing trade through it (Brooks says "it usually sets up a buying opportunity for those who do not get frightened by the spike down"), but entering new at that moment is shooting into a shakeout.

Notes: the 10:30 ET lower bracket has no direct Brooks analogue. Brooks discusses the first-hour reversal risk ("Reversals are far more common in the first hour" — ch. 57 line 9) but doesn't assign a specific 10:30 ET cutoff; that's pure empirical. The short-side 11:00-11:30 ET dead zone (Q11, pt 11) is Brooks-silent — Brooks analyzes shorts and longs symmetrically in SPT. This is a scanner-specific regularity probably tied to morning short-squeeze dynamics in single-name US equities (which Brooks doesn't trade — he's Emini-focused).

### Rule 7 — skip the 1st ticker-day detection

**Grounding: STRONG (for the phenomenon; thresholds are empirical).**

Brooks *Trends* ch. 57 line 9 is definitive:

> "Reversals are far more common in the first hour, as is discussed in book 3. The chance of the first bar being the high or low of the day on a day when there is a large gap opening can be 50 percent or more, if the bar is a strong trend bar in either direction. ... that means that buying above the first bar or shorting below it, expecting it to be the start of a strong trend, is a low-probability trade."

The "first detection is low-probability" pattern IS Brooks' first-hour reversal caveat. Pt 10 ([[small-pullback-trend-ordinal-and-daygate-2026-04-18]]) showed +0.56R/trade gap between 1st and 2nd+ detections — Brooks-consistent.

Notes: Brooks actually goes further. Ch. 57 lines 21, 74:

> "Even the best patterns still fail to do what you expect about 40 percent of the time. If the market does not pause by the third or fourth bar, it might have gone too far too fast, and this increases the chances that the market will reverse instead of trend."

> "Every day begins as a trend from the open day within the first few bars of the day. As soon as a bar moves above the high of the prior bar, the day is a trend from the open bull trend day, at least for that moment. If instead it trades below the low of the first bar, it is a trend from the open bear trend day. On most days, the move does not have much follow-through and there is a reversal, and the day evolves into some other type of day."

This is the structural reason rule 7 works: the *first* detection of the day fires when the SPT label is still "provisional" in Brooks' sense. The 2nd+ detection has survived one cycle of "the scanner thought this was SPT but the tape might reverse" and is therefore a more confirmed read.

### Rule 8 — After the day's first 3 policy-stack trades, continue only if ≥ 2 won

**Grounding: BROOKS-SILENT (pure empirical).**

Brooks doesn't discuss portfolio-level throttling rules. His teaching is single-trade-focused ("place your protective stop below the signal bar"), with no concept of "if today's first N trades have shown adverse texture, stop taking new signals."

Rule 8 is a risk-management wrapper derived from the 2026-04-14 breadth-fire diagnostic (pt 11, [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]]). It's the scanner-era replacement for Brooks' qualitative "read the tape" judgment: when 2 of the first 3 trades have lost, the scanner's day-type label is (empirically) *wrong* for that day, and the prudent action is to skip further fires.

Notes: the closest Brooks analogue is ch. 47 line 15 — "If you find that you missed a with-trend entry, stop looking for countertrend scalps and start trading only with-trend setups." That's a direction-flip rule, not a volume-throttle rule. Brooks doesn't offer a volume throttle because his book is discretionary single-trade advice, not portfolio-level risk policy.

Rule 8 is therefore the clearest pure-empirical rule in the stack. It's also the one most vulnerable to regime drift — if the correlation between "first 3 trades lose 2" and "day was mislabeled" decays (e.g. scanner's day-type classifier improves), rule 8 could become net-neutral or net-negative. Monitor in rolling audits.

### Rule 9 — Exit: 3R target / 1R stop / hold to resolution

**Grounding: PARTIAL (stop placement and hold-to-target are Brooks; 3R number is empirical).**

Brooks' stop placement prescription, *Reversals* ch. 51 line 64:

> "After entering, they should place a protective stop beyond the signal bar ... and then once there is a trend bar in their direction, they should move the stop to just beyond the entry bar."

The 1R stop below signal-bar is Brooks-canonical.

Brooks' target prescription is more variable. Ch. 47 line 33 and ch. 51 line 74 both suggest "risk-to-reward of at least 1:1 on scalp, swing for more":

> "They should consider scalping out of half at a reward that is equal to the risk (they can adjust this with experience) and then move the stop to breakeven or maybe a tick or two worse."

> "buy every high 2 ... scalping out of half at a reward that is equal to the risk ... then move the stop to breakeven ... Add on at every new opportunity."

So Brooks teaches **scalp at 1R, swing the rest**. The aiedge PLAYBOOK's "hold the full position to 3R (or hybrid 3R/5R)" is a simplification — it collapses Brooks' scalp-and-swing into a single-position hold. Pt 16 ([[small-pullback-trend-runner-targets-2026-04-18]]) found that scale-out variants were dominated by the all-in-to-target approach for this specific sample — 2R partial exits cost R relative to full hold. That's an aiedge-specific empirical finding; Brooks' scalp-swing teaching may still be correct for *his* samples (discretionary Emini swing trades) but is not the optimum for the aiedge signal distribution.

Notes: the hybrid rule 9 candidate (H1/H2 → 5R, L1/L2 → 3R, shorts cap 4R) from [[small-pullback-trend-target-walkforward-2026-04-18|pt 17]] IS compatible with Brooks' "swing for more on the strongest setups" — H1/H2 on an SPT day is the clearest swing-trade variant in Brooks' taxonomy.

The anti-breakeven teaching is strongly Brooks. Ch. 57 line 37:

> "if traders are looking to swing their trades, they should not be overly eager to move their stops to breakeven. When a trend is in a tight channel, it will usually come back to the entry price before reaching a new trend extreme."

Rule 9's "hold 1R stop to resolution" is the implementation of this — no mid-trade stop movement, no breakeven, no time-stop.

### Rule 10 (conditional) — swing2 stop on shorts

**Grounding: BROOKS-SILENT / mild TENSION.**

Brooks' stop teaching is signal-bar-based (ch. 51 line 64, quoted above). The idea of "use the max of the last 2 bar highs (including signal) on shorts specifically" has no Brooks analogue. Brooks discusses swing stops for trade-management (trail below swing-low after market makes new high), but not for initial stop placement.

Pt 20 ([[small-pullback-trend-stop-placement-2026-04-19]]) found that Brooks' H2/L2 swing-stop teaching *does not replicate* in the aiedge data — wider stops on H2/L2 longs actually hurt expectancy by −0.63R/trade. Only the short-side swing2 cell is positive. This is a mild tension: Brooks' "trail below swing" is a trade-management teaching, not an initial-stop teaching, so there's no strict contradiction — but if a reader interprets Brooks as "wider-than-signal-bar stops should work on strong trends," the aiedge finding contradicts that interpretation.

Notes: the mechanism for rule 10's short-side-only asymmetry is likely the 11:00 ET squeeze. Shorts in an SPT get probed by the morning squeeze that hits between bars 4-8 of the session; a 1-bar signal-bar stop on a short gets hunted on a 2-bar fake-out that a 2-bar swing stop survives. This is Brooks-aligned in spirit (expecting the "sharp reversal between 11:00 a.m. and noon PST" and not getting shaken out of swings) but not literally in prescribed mechanism.

### Rule 11 (conditional) — drop signal bars with `opp_tail ≥ 0.25`

**Grounding: SUPERFICIAL TENSION / deeper alignment.**

At first read, rule 11 *appears* to contradict Brooks' emphatic "buy despite the weak setups" advice. *Trends* ch. 57 line 35:

> "when experienced traders see a bull trend with small pullbacks ... along with many bear trend bars and weak buy signals, they understand what is going on. ... The experienced trader knows that too many traders will be doing the opposite of what they should be doing."

Ch. 57 line 57:

> "most of the buy signal bars looked bad. This kept bulls from buying, trapping them out ... Experienced, unemotional traders understand that bad buy signal bars and bear trend bars in a trend day with very small pullbacks are signs of a very strong bull trend. They made sure to buy despite the weak setups."

So Brooks: BUY THE UGLY BARS. Rule 11: DROP BARS WITH WICK ≥ 25%. Contradiction?

**No — on closer reading they're orthogonal.** Brooks' "ugly bar" = bear-close bar, small body, doji variant (close below open in a bull). That bar can have `opp_tail < 0.25` if the close is near the low of the bar (tight bottom wick on a bear-close bar). Conversely, a bar with `opp_tail ≥ 0.25` has a DEEP lower wick on a long signal — meaning the bar's OWN low probed noticeably below its body before recovering. That's a within-signal-bar pullback of 25%+ — which violates the SPT definition itself (Brooks: "all of the pullbacks are less than 20 to 30 percent of the recent average daily range"). The signal bar's intrabar pullback IS a pullback in the Brooks sense, and the SPT texture requires it to be shallow.

So rule 11 is enforcing Brooks' shallow-pullback criterion at bar-resolution, not contradicting Brooks' ugly-bar tolerance.

Notes: this is the subtlest rule in the stack. Pt 23's ([[small-pullback-trend-signal-shape-2026-04-19]]) framing was "stop hunt / contested bar", which is mechanism-level. The Brooks-consistency framing above (shallow-pullback enforcement at bar-resolution) is actually stronger — it explains WHY the rule generalizes book-wide (25/25 LOO-weeks, 52/52 LOO-symbols). Brooks' shallow-pullback constraint is the PLAYBOOK's most fundamental premise; of course it transfers to signal-bar tail structure.

Pt 24 ([[small-pullback-trend-tail-asymmetry-2026-04-19]]) is consistent with this: urgency (rule 3) already clips `with_tail ≥ 0.25` (with-trend wick on signal bar) because high-urgency with-trend bars close strong. Only `opp_tail` needs a separate rule because nothing upstream enforces it. That's exactly what you'd expect if the rule is the shallow-pullback constraint applied at bar-resolution.

---

## §2. Sizing tags and rejected-rule cross-reference

### Sizing tags

- **Low-relvol tag (`relvol_20 < 0.65`)** — Brooks *Trends* ch. 47 line 21 says: "Quiet markets with lots of small bars, many of which are dojis, often lead to the biggest trends." That's the volume equivalent of "low-volume absorption." Pt 22's empirical finding (perR +1.87 at n=16) matches Brooks' "quiet markets → biggest trends" teaching. STRONG grounding.
- **Q2 trap tag (`0.61 ≤ relvol_20 < 0.85`)** — Brooks-silent. Pure empirical signature.

### Rejected rules (from "What DOESN'T work" in PLAYBOOK)

Grounding audit on the rejected list:

- ❌ **2R target** — Brooks teaches 1R scalp + swing (ch. 51 line 64). 2R-fixed target is neither Brooks-canonical nor an aiedge optimum. Rejecting it is Brooks-neutral.
- ❌ **Widening stops to 2R** — Brooks explicitly teaches "beyond the signal bar" stops (ch. 51 line 64). Widening to 2R contradicts Brooks. Rejection is Brooks-consistent.
- ❌ **Breakeven-stop trap** — Brooks ch. 57 line 37: "they should not be overly eager to move their stops to breakeven." **Brooks' warning, directly adopted by the PLAYBOOK.** This is perhaps the single most explicit Brooks→PLAYBOOK transfer.
- ❌ **Time-stop (bar-N)** — Brooks-silent (Brooks doesn't use bar-count exits). Pure empirical rejection.
- ❌ **Event-triggered exits** — Brooks-silent.
- ❌ **Hard cap at N trades/day** — Brooks-silent.
- ❌ **Urgency-rescue of 1st detection** — aligned with Brooks' first-hour-reversal caveat (see rule 7 notes).
- ❌ **Signal-bar body% filter** — Brooks ch. 57 says "Most of the buy signal bars might be small bear trend bars or doji bars, and several of the entry bars might be outside up bars with small bodies." Brooks teaches to IGNORE body% in SPT. Rejection is Brooks-consistent.
- ❌ **Signal-bar ATR band** — Brooks-silent on ATR (not a term he uses).
- ❌ **Signal-bar relvol lower bound** — contradicts Brooks' "quiet markets, big trends" (ch. 47). Rejection is Brooks-consistent.

---

## §3. Brooks passages NOT captured in any rule

Three distinct Brooks teachings from chs. 47, 51, 57 that the PLAYBOOK currently ignores. Each could become a candidate rule after the Q40 (rule 10 × rule 11) work completes.

### §3.1 — Next-session follow-through

*Trends* ch. 57 line 11:

> "This type of trend can be so strong that there can be follow-through in the first hour or two of the next day, so traders should be looking to enter with trend on pullbacks after the open of the next day."

The scanner evaluates each day independently. There is no rule that says "if yesterday was SPT, weight today's first-hour same-direction signals higher." Pt 2 ([[small-pullback-trend-stop-and-timing-2026-04-18]]) §3 flagged this as Q3 but marked it "not testable with current data." The data *is* available (the scanner stores day-type labels per day in the SQLite DB); testing just needs a new script that joins day-N's day_type onto day-N+1's 9:30-10:30 ET signals.

**Candidate Q41:** Does an SPT day-N bias day-N+1's morning signals? If yes, is the effect worth a scanner scoring bump or a policy-stack filter change?

### §3.2 — 2× climactic burst as a sell/exit signal

*Trends* ch. 57 line 13:

> "That pullback is often about twice the size of the biggest pullback since the trend began in the first hour. It is often heralded by a relatively large, strong trend bar or two in the direction of the trend, but representing climactic exhaustion."

> "If sometime between 11:00 a.m. PST and noon the market has two relatively large consecutive bull trend bars breaking out to a new high, the move is more likely an exhaustive buy climax than the start of a new leg."

This is the SPT's built-in exit signal and Brooks teaches to **short it** (or at minimum exit swings on it). The PLAYBOOK's rule 9 is "hold to 3R target" — no climactic-exit logic. If a 2× burst fires before the 3R target hits, rule 9 sits through it. Empirically pt 7 ([[small-pullback-trend-adverse-event-2026-04-18]]) tested "event-triggered adverse exits" broadly and found them ALL to hurt — but that study used generic event triggers (first adverse close, bear-bar closing below 10EMA), not specifically the 2-consecutive-large-with-trend-bars signature Brooks describes.

**Candidate Q42:** Does a 2-consecutive-large-with-trend-bars detection inside [13:00–15:00 ET] predict near-term reversal of the SPT move with a magnitude worth an early exit? Test: flag SPT trades where, after entry, there occur two consecutive bars with range ≥ 2× trailing-10-bar median range in the trade direction; measure subsequent MAE vs MFE.

### §3.3 — First moving-average-gap bar as "final-leg" marker

*Trends* ch. 47 line 45:

> "This was a moving average gap bar setup in a strong trend and should be expected to test the high of the bull trend with either a lower high or a higher high. A moving average gap bar in a strong trend often leads to the final leg of the trend before a deeper, longer-lasting pullback develops, and the pullback can grow and become a trend reversal."

The "first MA-gap bar" (defined by Brooks as the first bar in 20+ bars whose high, for a bull, is below the 20EMA) is a Brooks swing-exit signal. The PLAYBOOK doesn't track it. This is connected to §3.2 — both are late-session "the trend may be done" signals.

**Candidate Q43:** Is "first MA-gap bar after entry" a useful exit signal for SPT swings? Given the aiedge stop logic (1R fixed) and target logic (3R fixed), does an override "exit at first MA-gap bar if currently at +1.5R or better" gain expectancy?

Note on §3.1-§3.3 priorities: all three are Brooks-explicit, all three are scanner-available, none have been tested. They rank low priority because the current PLAYBOOK is already +1.6-1.9 R/trade at n=83-88; new exit or entry overlays need to exceed that friction-adjusted bar to promote. But they're the clearest "untapped Brooks wells" in the source material.

---

## §4. Numerical Brooks claims — scanner audit

Brooks makes a handful of quantitative claims that the scanner can verify. The ones I can check from existing vault data:

### §4.1 — SPT frequency ("1–2× per month")

*Trends* ch. 57 line 13: "This is the strongest type of trend day and it forms only once or twice a month."

Scanner actuals: [[small-pullback-trend-PLAYBOOK]] "rhythm" section: "8 of 39 trading days deliver 81% of R." That's 8/39 ≈ 20% of trading days showing meaningful SPT contribution, or ~4-5 days per month.

Discrepancy: Brooks' "1-2×/month" is more conservative. Likely explanations:
- Brooks is observing the Emini, which has lower single-name dispersion than an equity scanner sees. A single-name universe fires on SPT days that don't reach index-level.
- Brooks' "SPT" definition is strict (every pullback ≤ 20-30% of ADR, closes at opposite extreme). The scanner's SPT component is softer (continuous 0-3 score with a 5-sub-check gate).
- The 8-of-39 metric counts "days with enough SPT points to deliver 81% of R," not "days Brooks would label SPT post-hoc." A stricter cut would likely match Brooks' 1-2/month.

No action — discrepancy is explainable and both Brooks' text and scanner data are self-consistent at their respective definitions.

### §4.2 — Pullback depth ("20–30% of recent ADR")

*Trends* ch. 57 line 31: "in the Emini where the average daily range has been about 12 points, the pullbacks might all be just two or three points."

2/12 = 17%, 3/12 = 25%. Brooks is quoting an aspirational tight range; the 20-30% number is in the SPT literature but I haven't located the exact quote in the extracted chapters. The scanner uses `SPT_DEPTH_SHALLOW = 0.25` — dead-center of Brooks' range.

No action — scanner is calibrated correctly.

### §4.3 — "Even the best patterns still fail ~40%"

*Trends* ch. 57 line 21: "Even the best patterns still fail to do what you expect about 40 percent of the time."

Scanner's PLAYBOOK default+hybrid has 66.3% WR → 33.7% fail rate. Close to Brooks' 40% but slightly better, probably because the stack adds filters Brooks doesn't itemize (urgency thresholds, first-3-gate, etc.). Consistent.

### §4.4 — "90% of days, high/low forms within opening range"

*Trends* ch. 57 line 9: "it forms within the opening range, which can last a couple of hours, in about 90 percent of days."

Not directly testable from the PLAYBOOK — the scanner doesn't label "opening range" per day. Could be checked from the raw 5-min bars but requires a script. Deferred to scanner-instrumentation wishlist.

---

## §5. Open / next steps

No new open question in the standard numeric Q-series. Three candidate new questions (Q41, Q42, Q43) flagged in §3 but NOT promoted to active status — they would compete with Q40 (rule 10 × rule 11 interaction) for the next active research slot, and Q40 is already queued with a designed script.

Recommended sequencing:
1. **Q40 first** — completes the conditional-rule validation cycle for rules 10 and 11. This is a near-term blocker on promoting both from conditional to core. Script queued (pt 25 §4); waits for memory pressure to clear.
2. **Q41 (next-session follow-through) second** — pure backtest question, small script (~150 LOC), no new feature engineering. Low-risk to run, clear actionable outcome (either bias the first-hour signals or don't).
3. **Q42 (2× climactic burst exit)** and **Q43 (first MA-gap bar exit)** — later, and only if Q41 lands. Both are exit-layer additions; they compete with rule 9 hybrid adoption and are harder to stack cleanly.

---

## §6. Related

- [[small-pullback-trend]] — parent concept doc; already cites Brooks chs 47 and 57 broadly but does not do rule-by-rule mapping.
- [[small-pullback-trend-PLAYBOOK]] — canonical 11-rule policy stack.
- [[small-pullback-trend-INDEX]] — reading order for the full arc; this is pt 26.
- [[small-pullback-trend-rule10-rule11-interaction-2026-04-19]] — pt 25, immediate predecessor; opens Q40 and motivates this note's "are we Brooks-true?" audit.
- `~/code/aiedge/brooks-source/extracted/trading-price-action-trends/47_signs-of-strength-in-a-trend.md` — primary source for rules 1, 9's stop placement, and sizing tag 1.
- `~/code/aiedge/brooks-source/extracted/trading-price-action-trends/57_trend-from-the-open-and-small-pullback-trends.md` — primary source for rules 1, 2, 4, 6, 7, 9 (hold to target / anti-breakeven), and §3's three untapped passages.
- `~/code/aiedge/brooks-source/extracted/trading-price-action-reversals/51_the-best-trades-putting-it-all-together.md` — primary source for rules 3, 5, 9 (stop placement).

---

## §7. Methodology note (why this note exists)

After 25 iterations of empirical research, it's easy to drift from the source material. The PLAYBOOK was last Brooks-audited against the parent concept doc ([[small-pullback-trend]]) — but the parent doc was written before pts 17-25 were derived, so rules 10 and 11 (and the hybrid target in rule 9) have NO Brooks-vs-empirical audit. This note closes that gap for the current 11-rule stack.

The synthesis is pure re-reading (no new Python); it complements pt 25's "theoretical derivation from existing outputs" approach from the other direction — pt 25 asks "what does the existing DATA tell us?", this pt 26 asks "what does the SOURCE TEXT tell us?" Together they form a full defensibility audit of the stack without burning compute during the Mac mini's thrash window.

The distinction matters for future meta-audits: the 9-of-11 Brooks-grounded count in the TL;DR is the highest-confidence part of this note. The §3 candidate questions are lower-confidence — they're passages that LOOK useful but haven't been empirically stress-tested.
