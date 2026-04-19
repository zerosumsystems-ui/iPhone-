# SPT — pt 24: with_tail vs opp_tail asymmetry derivation (Q39)

Opened 2026-04-19 (~07:00 ET). Closes [[small-pullback-trend-signal-shape-2026-04-19|pt 23]]'s Q39: "with_tail mean is 0.049 book-wide (tiny) but opp_tail is allowed through at q75 = 0.250. Why? Likely the alignment filters (always_in / gap_direction) absorb with_tail but don't touch opp_tail."

Script: `~/code/aiedge/scanner/scratch/spt_tail_asymmetry_2026_04_19.py`
Output: `_out_spt_tail_asymmetry_2026_04_19.txt`

---

## TL;DR

- **Asymmetry confirmed, but pt 23's hypothesis was wrong.** The always_in / gap_direction rules do NOT carry the with_tail clipping. **Rule 3 (urgency) does it all.**
- At rules 1-2 (raw universe, H1/H2/L1/L2 ∩ TFO/S&C), the asymmetry already exists: with_tail mean 0.072 vs opp_tail 0.123; with_tail q75 = 0.119 vs opp_tail q75 = 0.211; **with_tail ≥ 0.25 = 3.4%, opp_tail ≥ 0.25 = 20.8%.** So the Brooks pattern detector itself clips with-trend wicks by roughly 6× relative to opposing wicks.
- Adding rule 3 (urgency) drops with_tail ≥ 0.25 from 3.4% → **0.0%**. opp_tail ≥ 0.25 is unchanged (20.8% → 21.2%). Rules 4, 5, 6, 6s are all flat on tails.
- **No symmetric rule-11 complement is available.** Book-wide, dropping `with_tail ≥ 0.25` drops zero trades. Relaxing the threshold to `with_tail ≥ 0.10` drops 22 trades for +0.085 perR — inside noise. Stacked on pt 23's rule 11 (`opp_tail < 0.25`), adding `with_tail < 0.10` gains +0.011 perR on n=49 (drops DD −4 → −2.35, which is nice but thin).
- **Policy implication: none.** Pt 23's rule 11 asymmetry is load-bearing — `opp_tail` is allowed through because the scanner has no mechanism upstream that targets it, whereas `with_tail` is already scrubbed by urgency. Don't add a mirror rule.

---

## §1. The observation (what pt 23 saw)

Pt 23 §2 tabulated the full post-stack book and reported:

| field | non-Q2 mean (n=50) |
|---|---:|
| with_tail | 0.049 |
| opp_tail | 0.132 |

The with_tail value is tiny, roughly ⅓ of the opp_tail value. Brooks' symmetric framing ("signal bars should be clean on both sides") would predict these to be comparable. Pt 23 flagged this as a low-priority derivation question.

## §2. The asymmetry exists upstream

Before any scanner filters run, the raw H1/H2/L1/L2 ∩ TFO/S&C universe (n=725) already shows:

| field | n | mean | q50 | q75 | q90 | % ≥ 0.25 |
|---|---:|---:|---:|---:|---:|---:|
| with_tail | 725 | 0.072 | 0.023 | 0.119 | 0.197 | **3.4%** |
| opp_tail | 725 | 0.123 | 0.081 | 0.211 | 0.333 | **20.8%** |

The median with_tail is 0.023 — most signal bars close at or near the with-trend extreme. The median opp_tail is 0.081. **The Brooks pattern detector encodes a structural preference for close-at-extreme bars on the with-trend side.** That's what "H1 = signal of strength" means at the bar level: the bar closes at or above (below) the previous bar's extreme with little rejection beyond the close.

The reason opp_tail is larger in the same population is that pullback lows (for longs) are the *expected* feature of a pullback — the bar is supposed to test below the prior close and rally; the lower wick IS the pullback signature. It's Brooks-consistent, not a bug.

## §3. Sequential-filter decomposition — where does with_tail shrink?

Apply filters 3 → 4 → 5 → 6 → 6s in order and log tail distributions at each stage:

| stage | n | wt_mean | wt_q75 | ot_mean | ot_q75 | wt ≥ .25 | ot ≥ .25 |
|---|--:|--:|--:|--:|--:|--:|--:|
| 1-2 (setup + day_type) | 725 | 0.072 | 0.119 | 0.123 | 0.211 | 3.4% | 20.8% |
| + r3 (urgency) | 406 | 0.060 | 0.111 | 0.129 | 0.214 | **0.0%** | 21.2% |
| + r4 (gap aligned) | 390 | 0.061 | 0.111 | 0.131 | 0.216 | 0.0% | 21.3% |
| + r5 (always_in) | 390 | 0.061 | 0.111 | 0.131 | 0.216 | 0.0% | 21.3% |
| + r6 (time window) | 267 | 0.062 | 0.112 | 0.133 | 0.211 | 0.0% | 21.7% |
| + r6s (short time) | 246 | 0.061 | 0.112 | 0.137 | 0.216 | 0.0% | 22.8% |

The single biggest clipper is **rule 3**. After urgency, no bar in the book has with_tail ≥ 0.25. Rules 4, 5, 6, 6s shave almost nothing off either tail — their selectivity is entirely on urgency, gap, always_in, and time-of-day, not bar shape.

**Pt 23's hypothesis was that always_in / gap_direction were the absorbers. That is wrong.** The always_in rule (r5) drops zero trades on the full-db window (-0 in the decomposition) because every gap-aligned bar with an urgent signal already has aligned always_in in this universe. The gap_direction rule (r4) drops 16 trades but doesn't touch with_tail distribution. **Urgency does it.**

This is Brooks-coherent. Urgency in our scoring includes trend-bar density, signal-bar body-direction alignment, and gap-to-close consistency. A with-trend signal bar with a 25%+ upper wick (for a long H1) is almost by definition not a strong trend bar — it failed to close at the range high. Urgency penalizes those directly; opposing wicks are invisible to the urgency score because the bar still closed at the with-trend extreme.

### Leave-one-rule-in cross-check

Applying one rule at a time on top of 1-2 (not cumulative):

| rule | n | wt_mean | wt_q75 | ot_mean | ot_q75 |
|---|--:|--:|--:|--:|--:|
| rule 3 alone | 406 | 0.060 | 0.111 | 0.129 | 0.214 |
| rule 4 alone | 591 | 0.070 | 0.115 | 0.127 | 0.213 |
| rule 5 alone | 725 | 0.072 | 0.119 | 0.123 | 0.211 |
| rule 6 alone | 550 | 0.074 | 0.122 | 0.123 | 0.211 |
| rule 6s alone | 678 | 0.070 | 0.115 | 0.125 | 0.213 |

Only rule 3 moves with_tail meaningfully. Rule 5 (always_in) by itself drops no rows (the 725 entire universe is already always_in ≠ opposed — this is a latent guarantee of H1/H2/L1/L2 detection on TFO/S&C days). Rules 4, 6, 6s have roughly the same mean/q75 as the raw universe; they're not tail-selective.

## §4. Post-stack WIN vs LOSS distribution

On the final n=88 book (pt 23's baseline):

| field | WIN (n=59) | LOSS (n=29) | delta (W−L) |
|---|--:|--:|--:|
| with_tail mean | 0.044 | 0.073 | **−0.028** |
| with_tail q75 | 0.083 | 0.114 | — |
| opp_tail mean | 0.146 | 0.154 | −0.008 |
| opp_tail q75 | 0.211 | 0.260 | — |
| opp_tail ≥ 0.25% | 20.3% | **34.5%** | — |

Two small signals:

1. **opp_tail separates WIN from LOSS at the q75 / tail**: 20% of wins have opp_tail ≥ 0.25 vs 34% of losses. This is exactly what pt 23 already exploited — rule 11 filters it.
2. **with_tail mean is lower on WINs** than LOSSes (0.044 vs 0.073, −0.028). Small but directionally real. The q90 of LOSSes (0.159) is under 0.25 — no loss has extreme with_tail — but the LOSS distribution is systematically wider than the WIN distribution in the 0–0.15 band.

The second signal is what motivates §5 (could we filter on with_tail < X at a low threshold?).

## §5. Symmetric `with_tail ≥ X` filter — does it add anything book-wide?

On the final n=88 baseline (pt 23 §4):

| threshold | dropped n | kept n | kept perR | delta | kept DD |
|---|--:|--:|--:|--:|--:|
| baseline | 0 | 88 | +1.615 | — | −4.00 |
| drop with_tail ≥ 0.10 | 22 | 66 | +1.700 | +0.085 | −3.00 |
| drop with_tail ≥ 0.15 | 10 | 78 | +1.647 | +0.032 | −4.00 |
| drop with_tail ≥ 0.20 | 2 | 86 | +1.574 | −0.041 | −4.00 |
| drop with_tail ≥ 0.25 | 0 | 88 | +1.615 | 0.000 | −4.00 |
| drop with_tail ≥ 0.30 | 0 | 88 | +1.615 | 0.000 | −4.00 |

**Compare pt 23: `opp_tail ≥ 0.25` drops 22 trades for +0.31R/trade.** The symmetric `with_tail` drop at 0.10 threshold matches on drop count but captures only +0.085R/trade — noise.

Why the 0.10 threshold matches on n? Because **the with_tail distribution is compressed**. To drop the same 22-trade count as opp_tail at its q75 cutoff, with_tail needs a 0.10 cutoff — which is its own q75 equivalent after urgency's clipping. The symmetry is in distribution rank, not in absolute cutoff value.

But the delta is 4× smaller. That says: even after mapping to the same population percentile, with_tail carries much less signal than opp_tail. Urgency has already used most of the with_tail information.

## §6. Does with_tail add residual edge on top of rule 11?

Starting from pt 23's rule-11 kept book (opp_tail < 0.25, n=66, perR +1.924):

| add | dropped | kept n | kept perR | Δ vs r11 | kept DD |
|---|--:|--:|--:|--:|--:|
| + with_tail < 0.10 | 17 | 49 | +1.935 | +0.011 | **−2.35** |
| + with_tail < 0.15 | 7 | 59 | +1.916 | −0.007 | −4.00 |
| + with_tail < 0.20 | 1 | 65 | +1.876 | −0.047 | −4.00 |
| + with_tail < 0.25 | 0 | 66 | +1.924 | 0.000 | −4.00 |

The with_tail < 0.10 cell has a perR delta of +0.011 — statistical zero — BUT **cuts max DD from −4.00 to −2.35** on n=49. That's an interesting DD-only effect at the cost of 17 trades of scale (~25% of book). Not enough lift on perR to promote; the DD improvement is almost certainly noise at n=49 (single-trade addition at the peak would swing it).

**Conclusion: no rule-11 complement. `with_tail` is not a filter candidate.**

## §7. Direction check — is the asymmetry direction-specific?

| direction | stage | n | wt_mean | wt_q75 | ot_mean | ot_q75 |
|---|---|--:|--:|--:|--:|--:|
| long | rules 1-2 | 429 | 0.071 | 0.118 | 0.126 | 0.200 |
| long | + full pre-filters | 161 | 0.058 | 0.107 | 0.136 | 0.200 |
| short | rules 1-2 | 296 | 0.074 | 0.125 | 0.118 | 0.213 |
| short | + full pre-filters | 85 | 0.067 | 0.122 | 0.140 | 0.260 |

Both directions show the same asymmetry at 1-2 and at full pre-filter. Notable: short-side opp_tail q75 GROWS from 0.213 → 0.260 after pre-filters (short book has less raw material, more concentrated in the tail band). This is consistent with the rule-6s time concentration documented in [[small-pullback-trend-short-side-and-daily-gate-2026-04-18]] — short-side setups in the narrower 11:30–14:15 band are operating in a structurally noisier tape.

No direction-specific rule needed.

## §8. Body-direction interaction — trend-bar vs doji bars

On the full pre-filter book (n=246), sliced by signal-bar body direction:

| body slice | n | wt_mean | wt_q75 | ot_mean | ot_q75 |
|---|--:|--:|--:|--:|--:|
| body_dir ≥ 0.5 (strong trend bar) | 246 | 0.061 | 0.112 | 0.137 | 0.216 |
| 0 ≤ body_dir < 0.5 (weak trend) | 0 | — | — | — | — |
| body_dir < 0 (doji/against) | 0 | — | — | — | — |

The pre-filter book is entirely strong trend bars (body_dir ≥ 0.5 for all 246). The scanner's rules collectively reject any weak or doji signal bar. **This is the upstream mechanism:** urgency ≥ 4 (≥ 6 on TFO) requires a with-trend close and decent body, which mechanically caps with_tail. Opp_tail has no analogous upstream check because a shallow pullback can be rejected while the bar still closes strongly (that's literally the SPT pattern).

## §9. Brooks interpretation

Pt 23's rule 11 is asymmetric **by Brooks construction**. A with-trend signal bar in an SPT is supposed to be "a strong bar closing at the high (for longs)." That's what urgency scores. Any bar with a meaningful `with_tail` has failed that test and has already been filtered.

A pullback bar, on the other hand, is *supposed* to test below recent lows and rally — the `opp_tail` IS the pullback. The scanner doesn't penalize opp_tail because that would kill the signal entirely. But there's a threshold above which the opposing wick is no longer "shallow pullback" and becomes "contested bar / failed counter-trend probe" — that's where pt 23's 0.25 cutoff earns its keep.

The shape of the rule is right: **penalize opp_tail (it's unbounded from below, the scanner doesn't touch it), don't bother with with_tail (it's already bounded by urgency).** Rule 11's asymmetry is the correct response to the underlying asymmetry in signal-bar expectations.

## §10. What this closes and what it doesn't

### Closes
- **Q39 (why is opp_tail allowed through but with_tail clipped?)** — Urgency (rule 3) clips with_tail. Always_in / gap_direction do not, contrary to pt 23's stated hypothesis. The asymmetry is by Brooks construction: strong with-trend signal bars (urgency's target) imply low with_tail; shallow pullbacks (the SPT diagnostic) imply moderate opp_tail. No policy change.

### Does not open new questions
- A pt 23-style follow-up on with_tail is not warranted. The residual signal is noise-level and would require n ≥ 300 to see a reliable delta, at which point other regime effects dominate.

### Reference note for future work
- If a future regime produces signal bars with meaningful with_tail despite urgency (e.g., a regime where urgency composition drifts), revisit this — the empirical mechanism would mean urgency is degrading as a with_tail proxy. Watch at the next regime-scoring recalibration.

## §11. Artifacts

- Script: `~/code/aiedge/scanner/scratch/spt_tail_asymmetry_2026_04_19.py`
- Output: `~/code/aiedge/scanner/scratch/_out_spt_tail_asymmetry_2026_04_19.txt`
- Parent: [[small-pullback-trend]], [[small-pullback-trend-PLAYBOOK]]
- Prior note: [[small-pullback-trend-signal-shape-2026-04-19]] (pt 23)
- Closes: Q39 (no policy change)

---

Fallback: none required — no rule change proposed.
