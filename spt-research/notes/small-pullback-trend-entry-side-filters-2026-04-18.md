# SPT — Entry-Side Filter Research (2026-04-18, pt 8)

Ninth follow-up to [[small-pullback-trend]]. Picks up the "what remains open" note at the end of [[small-pullback-trend-adverse-event-2026-04-18]] §8:

> Further gains likely come from *entry-side* filtering — e.g., a volatility / ATR quality check or a time-of-day bucket finer than the 14:00 ET band — rather than mid-trade exit refinements.

Two entry-side splits tested on the urgency-gated SPT with-trend population:

1. **Time-of-day — fine-grained 30-min ET buckets.** The existing `suppress 13:45–14:45 ET` policy is too narrow. The true trap zone extends to the close.
2. **Signal-bar ATR ratio** (signal-bar true range vs prior 14-bar ATR). No clean filter — mild U-shape but not actionable.

Headline: a refined time-of-day rule — **allow entries only between 10:30 ET and 14:15 ET** — adds **+0.35R per trade** (+24R/quarter, n=102 kept of 128) versus the current `suppress 13:45–14:45` policy.

Population: setup ∈ {H1,H2,L1,L2}, day_type ∈ {trend_from_open, spike_and_channel}, urgency ≥ 4, gap_direction aligned with setup, result ∈ {WIN, LOSS, SCRATCH}, chart_json present. **n=153** resolved at 3R target / 1R stop.

Snapshot: `~/code/aiedge/scanner/db/pattern_lab.sqlite` @ 2026-04-18 afternoon.
Script: `~/code/aiedge/scanner/scratch/spt_entry_side_filters.py`.

## 1. 30-min ET bucket table (urgency ≥ 4, all TFO + S&C)

| Bucket (ET) | n | WR% | Sum R | Per-trade |
|-------------|--:|----:|------:|----------:|
| 10:30–11:00 | 21 | 81.0 | +39.46 | **+1.88R** |
| 11:00–11:30 | 26 | 26.9 | −9.44 | −0.36R |
| 11:30–12:00 | 26 | 65.4 | +36.97 | +1.42R |
| 12:00–12:30 | 19 | 78.9 | +38.30 | **+2.02R** |
| 12:30–13:00 | 10 | 50.0 | +8.74 | +0.87R |
| 13:00–13:30 | 10 | 40.0 | +8.39 | +0.84R |
| 13:30–14:00 | 9 | 77.8 | +17.63 | +1.96R |
| 14:00–14:30 | 7 | 42.9 | +1.57 | +0.22R |
| 14:30–15:00 | 9 | **0.0** | −9.00 | −1.00R |
| 15:00–15:30 | 4 | 25.0 | +0.00 | 0.00R |
| 15:30–16:00 | 8 | 25.0 | +0.00 | 0.00R |
| 16:00+ | 4 | 0.0 | −4.00 | −1.00R |

Two observations worth pulling out:

- **Lunch is not a dead zone for SPT.** Classic intraday lore says fade the lunch period. In this urgency-gated SPT population the 11:30–13:30 ET band is the single strongest performer (63% WR, +1.42R/trade on n=65). The grind is the grind; institutional fills don't stop for lunch.
- **The 11:00–11:30 dip** (26.9% WR, −0.36R on n=26) is a single-bucket outlier inside an otherwise healthy morning block. Likely noise (small n at this resolution), not a real regime — worth watching as data accumulates but not yet actionable.

## 2. Session macro-windows

Collapsing to five macro-windows:

| Window | n | WR% | Sum R | Per-trade |
|--------|--:|----:|------:|----------:|
| B mid-morning 10:30–11:30 | 47 | 51.1 | +30.02 | +0.64R |
| C lunch 11:30–13:30 | 65 | 63.1 | +92.40 | **+1.42R** |
| D early-PM 13:30–14:45 | 16 | 62.5 | +19.20 | +1.20R |
| E late-PM 14:45–15:30 | 13 | **7.7** | −9.00 | **−0.69R** |
| F close 15:30–16:00 | 8 | 25.0 | +0.00 | 0.00R |
| Z outside | 4 | 0.0 | −4.00 | −1.00R |

**The real trap zone is 14:45–16:00 ET, not just the 14:00 ET "big pullback" band.** Window E (14:45–15:30) is actually *worse* than the existing 13:45–14:45 suppression band ever was. Window F (close) is flat — no edge, and is where SPT turns into a 15-min-to-MOC gamble.

The prior 14:00 ET suppression was derived in [[small-pullback-trend-empirics-2026-04-18]] §1 from Brooks' "11 PST / 14:00 ET larger pullback" claim. This note extends it: the drag on SPT entries continues well past 14:45 because any late-day entry has limited bars left to run to a 3R target.

## 3. Current vs proposed time-of-day policy

The existing consolidated policy from the prior notes is: `urgency ≥ 4` (≥ 6 on pure TFO), with-trend only, `gap_direction` aligned, `always_in != opposed`, **suppress entries 13:45–14:45 ET**. A/B against the proposed `enter only 10:30–14:15 ET`:

| Variant | n | WR% | Sum R | Per-trade |
|---------|--:|----:|------:|----------:|
| urgency≥4, no time filter | 153 | 51.0 | +128.62 | +0.84R |
| urgency≥4, current (suppress 13:45–14:45) | 138 | 50.0 | +112.42 | +0.81R |
| urgency≥4, **proposed (10:30–14:15 only)** | **121** | **59.5** | **+140.04** | **+1.16R** |
| policy-stack (TFO≥6, S&C≥4), no time filter | 128 | 53.1 | +123.61 | +0.97R |
| policy-stack + current (suppress 13:45–14:45) | 115 | 52.2 | +109.41 | +0.95R |
| policy-stack + **proposed (10:30–14:15)** | **102** | **61.8** | **+133.04** | **+1.30R** |

Delta on the policy stack: **+24R/quarter, +0.35R/trade** by switching from the narrow 13:45–14:45 suppression to the wider "entry window" rule. The *current* rule barely helps (−0.02R/trade vs no filter) because it lets the 14:45–15:30 trap and the close-hour entries back in.

Counter-intuitively, the current rule is **close to break-even with doing nothing** — the 13:45–14:45 zone it filters out (n=13) has a baseline of ~−1.2R/trade (big pullback bars at 14:30 ET are −1R n=9 all losses; 14:00 is marginal at +0.22R). The reason it still helps a hair is the 14:30 bucket alone carries most of the damage. Extending the cut to 14:15 (and past 14:45 to the close) catches the rest.

## 4. Signal-bar ATR ratio — no actionable filter

The bar-level volatility check: compute the signal bar's true range, divide by the ATR-14 of the bars that preceded it. Buckets:

| Ratio band | n | WR% | Sum R | Per-trade |
|------------|--:|----:|------:|----------:|
| < 0.75 (quiet) | 41 | 56.1 | +45.85 | +1.12R |
| 0.75–1.25 (normal) | 69 | 43.5 | +40.36 | +0.58R |
| 1.25–2.00 (loud) | 30 | 50.0 | +22.77 | +0.76R |
| 2.00–3.00 (explosive) | 13 | 76.9 | +19.64 | **+1.51R** |

Equal-count quartiles give a similarly flat picture (+1.07 / +0.68 / +0.83 / +0.78R across Q1–Q4). The U-shape — quiet bars (<0.75) and explosive bars (≥2.0) both outperform the middle — is suggestive but does not survive as a policy rule:

- Adding a "ratio in band [0.5, 3.0]" filter on top of the window filter yields +1.04R/trade (vs +1.11R without ATR filter). ATR is **additive-negative**.
- The explosive bucket is n=13 — too small to bet on.
- The mid-band (0.75–1.25) contains the plurality of trades and is still positively expected.

Verdict: **don't filter on signal-bar ATR ratio.** The edge from time-of-day does not replicate in volatility quality.

## 5. Session-bar position

Orthogonal check — how many bars into the session does the entry sit?

| Position | n | WR% | Per-trade |
|----------|--:|----:|----------:|
| 1–2hr | 62 | 50.0 | +0.69R |
| 2–3.5hr | 47 | **78.7** | **+1.93R** |
| 3.5–5hr | 24 | 33.3 | +0.29R |
| 5hr+ | 20 | **10.0** | **−0.60R** |

This is the time-of-day finding expressed in bars-from-open rather than wall-clock time: entries that land 2–3.5hr into the session (≈ 11:30–13:00 ET) are best; 5hr+ (≈ 14:30 ET+) is the trap. Adds no new information beyond §2 but confirms the effect isn't an ET-clock artifact.

## 6. Direction × window cross-tab

Revisits Q11 (short-side weakness) with time-of-day overlay:

| Window | Long WR% / per-R | Short WR% / per-R |
|--------|-----------------:|------------------:|
| B mid-morning | 71.0% / +1.32R (n=31) | **12.5% / −0.68R** (n=16) |
| C lunch | 60.0% / +1.31R (n=40) | 68.0% / +1.60R (n=25) |
| D early-PM | 57.1% / +1.09R (n=7) | 66.7% / +1.29R (n=9) |
| E late-PM | 12.5% / −0.50R (n=8) | 0.0% / −1.00R (n=5) |
| F close | 50.0% / +1.00R (n=4) | 0.0% / −1.00R (n=4) |

Two findings:

- **Short-side early-session weakness is the real asymmetry.** In the 10:30–11:30 ET bucket (B), longs crush (71% WR) while shorts collapse (12.5% WR). In C (lunch) the asymmetry disappears — both sides run 60–68% WR. This suggests the 2026-01 → 2026-04 window's "bull regime" short-side weakness (Q11) is concentrated in the opening hour, not a whole-day effect.
- **The late-PM trap is direction-agnostic.** Both sides lose in E/F/Z. Good sign for the policy — the time filter is a clean regime guard, not a direction-specific bandage.

Tentative direction-aware refinement: on shorts, further tighten the window to **C+D only (11:30–14:15 ET)**, skipping the B bucket's 12.5% WR pocket. On longs, keep B+C+D (10:30–14:15). Worth re-checking once data spans a bear leg — if short-side B normalizes in a bear regime, this becomes an undue restriction.

## 7. Revised consolidated entry policy

Stacked from prior notes, with §3 replacing the time-of-day rule:

**Gate stack** (ALL must hold for entry):
1. Setup ∈ {H1, H2, L1, L2} (with-trend only; FH/FL countertrend rejected — [[small-pullback-trend-targets-and-urgency-2026-04-18]] Q8).
2. `day_type ∈ {trend_from_open, spike_and_channel}`.
3. `urgency ≥ 4`; tighten to `urgency ≥ 6` on `trend_from_open` (regime-dependent — [[small-pullback-trend-urgency-generalization-2026-04-18]] §4).
4. `gap_direction` aligned with setup (no gap_opposed at urgency ≥ 4 — [[small-pullback-trend-alignment-filters-2026-04-18]] §1).
5. `always_in != opposed(setup)` ([[small-pullback-trend-alignment-filters-2026-04-18]] §2).
6. **NEW — time window: signal time ∈ [10:30 ET, 14:15 ET).** Replaces `suppress 13:45–14:45 ET`.
7. **Tentative — short-side tighter window: [11:30 ET, 14:15 ET).** Revisit when data spans a bear leg.

**Exit stack**:
- 3R fixed target, 1R stop, **hold to resolution** (no bar-N time-stop — Q13 rejected; no event-triggered adverse exit — Q14 rejected).

**Expected return on the window-filtered policy stack**: +1.30R per trade, 61.8% WR, n ≈ 34 trades/month in this 2-month window. Projects to +44R/month at current detection cadence before any frictions.

## 8. What remains open

- **Q1 (blocked)** — SPT-component-raw bucketed WR. Still blocked on schema.
- **Q5 (open)** — cross-asset SPT. Unchanged; waits on multi-asset expansion.
- **Q11 (partially closed)** — short-side weakness is concentrated in the **opening hour**, not whole-day. Full closure pending bear-regime data.
- **NEW — 11:00–11:30 ET dip**: 26.9% WR / −0.36R on n=26 inside an otherwise healthy block. Could be the "mid-morning reversal hour" pattern Brooks describes (ch. 56, *Trends*) or noise. Revisit when the bucket doubles in size.
- **NEW — ATR quality**: explored and rejected. The U-shape is real but too small-n to harden into a rule.

## Related

- [[small-pullback-trend]] — parent concept. Q15 (time-of-day refinement) to be added to the queue when the policy ships.
- [[small-pullback-trend-adverse-event-2026-04-18]] — predecessor; flagged entry-side filters as the remaining unexplored edge.
- [[small-pullback-trend-empirics-2026-04-18]] §1 — original 14:00 ET suppression derivation (now superseded by §3 here).
- [[small-pullback-trend-urgency-generalization-2026-04-18]] — urgency regime-dependent thresholds.
- [[small-pullback-trend-alignment-filters-2026-04-18]] — gap + always_in filters, short-side asymmetry (Q11).
- [[small-pullback-trend-time-stop-2026-04-18]] — rejected exit refinement.
- Script: `~/code/aiedge/scanner/scratch/spt_entry_side_filters.py`.
