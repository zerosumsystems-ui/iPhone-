# Head of Strategy memo — 2026-04-19 (for Monday 4/20 open)

Phone mirror of [Meta/Head of Strategy 2026-04-19.md](../../aiedge-vault-mirror) in the vault. Read before the open.

## TL;DR

Two backtest-validated live strategies. Run **both**, de-duped. Target 2-5 trades/day.

| Strategy | WR | R/trade | Role |
|---|---:|---:|---|
| **S1** C3 PLAYBOOK (SPT 11-rule stack) | **74.6 %** | **+1.84R** | Primary — rare but high-conviction |
| **S2** Top-tier gate Tier A+B (early + urg 7-8 + no fades) | **71.4 %** | **+1.14R** | Complementary — fires on full 498-sym universe |

## Monday 4/20 playbook

**Pre-market (09:00-09:30)**
- Scan overnight news on MRK, ADBE, ORCL, AMD. Kill any with earnings/M&A gap.
- Wait for first scanner print (~09:45) for SPY day_type tag.

**No-trade zone: 09:30-10:30 ET.** Both strategies block.

**Prime window: 10:30-13:30 ET** — primary execution band for both strategies.
- **S1 priorities (watchlist order):** MRK 30m/60m long · WMT 5m long · ADBE short · ORCL short · AMD daily pullback long
- **S2 priorities:** take anything passing the gate on the full 498-sym universe. Tag day_type; if `spike_and_channel`, size up 1.5×.

**13:30-14:15 ET:** S1 only. S2 blocks past bar 47.

**14:15-close:** exits only. No new entries.

## Key gates (memorize)

**S1 (SPT stack) — all must be true:**
- Setup ∈ H1/H2/L1/L2 · day_type ∈ trend_from_open / spike_and_channel · urgency ≥ 4 (≥ 6 on TFO)
- Gap aligned with setup · always_in ≠ opposed(setup) · time 10:30-14:15 ET (shorts 11:30-14:15)
- NOT 1st ticker-day detection · daily kill-switch after 3 trades if < 2 won · opp_tail < 0.25
- **Stops:** long = signal-bar low · short = max of last-2-bar highs
- **Targets:** H1/H2 long 5R · H2/L1/L2 short 4R · L1/L2 long 3R · H1 short 5R

**S2 (top-tier gate Tier A+B):**
- Top-tier fire (urgency ≥ 7.0) · setup ∉ {FH1/FH2/FL1/FL2} · bar ≤ 47 · urgency in [7.0, 7.9]
- Size 1.5× when day_type = spike_and_channel
- 1R stop / 2R cap · size = 0.5× S1 size

## De-dup rule

Both fire same symbol same hour → take as S1 (S1 sizing, S1 exit). Never double-size on correlated signals.

## Daily risk limits

- Max concurrent open risk: **3R**
- Daily hard stop: **−4R** (walk away)
- S1 rule 8 still governs S1 trades only

## What we're NOT doing Monday

- No range-day MR strategy (no backtest = speculation)
- No S1 rules on 15m/30m/60m (empirics are 5m-native; pt 33 overlay is monitoring only)
- No urgency ≥ 9 size-up (ladder anomaly says it's WORSE)
- No dropping rule 7 / rule 8 to chase volume

## Risk disclosure

S1 and S2 are correlated (same with-trend SPT regime). Combined DD is NOT sqrt(2) × single. Regime flip (sustained downtrend) breaks the long/short asymmetry. First 20 live trades will shake the point estimates — don't take one bad day personally, but DO walk at −4R.

## Canonical source

Full memo in the vault: `Meta/Head of Strategy 2026-04-19.md`. Scanner live at [aiedge.trade](https://aiedge.trade).
