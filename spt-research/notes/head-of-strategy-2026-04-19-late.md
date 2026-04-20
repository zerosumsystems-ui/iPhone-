# Head of Strategy — late memo — Mon 2026-04-20 trade queue

**Autonomous run, Sun 2026-04-19 late.** Integrates pt 34 tier-grading into the live strategy stack. The 20:00 memo named S1 (C3 PLAYBOOK) + S2' (Tier A urg ≥ 5). This memo consolidates pt 34's A/B/C quality grading into the concrete Monday trade queue.

## Trade queue for Mon 4/20 (S1 names, priority order)

| # | Dir | Name | Frame | Tier | Stop / Target |
|---|:--:|------|:-----:|:----:|---------------|
| 1 | 🟢 | **MRK** | 30m/60m | B × 4 TFs | signal-low / 5R |
| 2 | 🟢 | **AVGO** | daily | A (pb 0.39, cls 0.99) | pullback swing / 5R |
| 3 | 🔴 | **ADBE** | 15m | **A** (only A-tier short) | rule-10 swing-2 / 4R |
| 4 | 🟢 | **AMD** | daily | A (Q49 = continuation) | pullback swing / 5R |
| 5 | 🟢 | **WMT** | 5m | B (pb 0.43, +18.69R) | signal-low / 5R |
| 6 | 🟢 | **DIS** | 60m/30m/15m | B × 3 | PLAYBOOK std |
| 7 | 🟢 | **UNH** | 60m/30m/5m | B × 3 | PLAYBOOK std |
| 8 | 🟢 | **CAT** | daily/60m/30m | B × 3 | PLAYBOOK std |
| 9 | 🟢 | **IWM / DIA** | daily/60m | B × 2 | PLAYBOOK std (ETF) |
| 10 | 🔴 | **CRM** | 15m/5m | B × 2 | rule-10 swing-2 / 4R |

Plus **S2'** fires live from the 498-sym scanner (urg ≥ 5, Tier A, no fades, bar ≤ 47), 0.5× sizing.

## What changed since the 20:00 memo (short book only)

- **DROPPED:** CVX (pb 0.97), ORCL 30m (pb 0.83), NFLX (pb 0.98), GE short, AMZN 15m short — all fail pt 34 B-gate; bounces inside the window ≥ net move.
- **ADDED / UPGRADED:** AVGO 3-TF cross (was daily-only), CAT + IWM breadth adds, AAPL on bench.
- **AMD Q49 resolved:** pullback 0.38 + closeness 0.97 = continuation (not exhaustion).

## Rule stack (unchanged)

S1 gates: setup ∈ {H1/H2/L1/L2} · day_type ∈ {TFO/S&C} · urg ≥ 4 (≥ 6 on TFO) · gap aligned · always_in ≠ opposed · time [10:30, 14:15) ET (shorts [11:30, 14:15)) · not 1st detection · rule 8 kill switch · opp_tail < 0.25. Stops: signal-low long / swing-2 short. Targets 3-5R per rule 9.

S2': same as S1 Tier A but universe = full 498 symbols and urgency floor relaxed 7 → 5 per urgency-ladder anomaly.

## Sizing (from 20:30 memo)

- Per-trade: 1 % (S1) / 0.5 % (S2')
- Max concurrent: 5 % equity
- Daily hard stop: −6 %
- Weekly hard stop: −12 %
- Rule 8 still governs S1 (3-trade gate)

## Expected Monday throughput

- S1: 0-2 trades (rare, high-conviction)
- S2': 2-3 trades (everyday fires)
- Combined after de-dupe: **2-4 trades**

Zero-fire days are real and valid. Don't force.

## Key reminders

- No trades 09:30-10:30 (both strategies block). Watch first hour for gap / always_in / day_type.
- Shorts window starts 11:30 ET.
- All entries stop at 14:15 ET (close-only after).
- De-dupe: if a symbol fires both S1+S2', it's an S1 trade at 1× sizing. Do not double-up.
- DO NOT short CVX / ORCL / NFLX / GE on bounces — pt 34 says wait for the bounce to roll over first.
- MRK single-name concentration risk: ONE position, don't stack MRK-30m + MRK-60m + MRK-5m together.

## Next research sprint (deferred)

1. S1' universe expansion (C3 rules on full 498-sym) — could 5-10× throughput.
2. Tier × perR validation (Q50) — does A beat B in realized R?
3. Sector-tilt regime read (Q51) — industrials + small-caps + defensives all newly promoted.

---

**Vault source of truth:** `vault/Meta/Head of Strategy 2026-04-19-late.md`
**Parent memos:** `vault/Meta/Head of Strategy 2026-04-19.md`, `vault/Meta/Scale-Up Strategy Post-PDT 2026-04-19.md`
**Tier source:** `vault/Brooks PA/concepts/small-pullback-trend-tier-graded-candidates-2026-04-19.md`
