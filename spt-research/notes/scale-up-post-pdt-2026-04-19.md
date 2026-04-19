# Scale-up strategy post-PDT — 2026-04-19

Phone mirror of `Meta/Scale-Up Strategy Post-PDT 2026-04-19.md` in the vault. Follows up on the Head of Strategy memo.

## Blunt answer on 10-20 trades/day

**Not feasible on the US-equity scanner alone.** Supply caps at ~9 detections/day across ALL setups/urgencies. Even at maximum dilution, we can't hit 10-20 without multi-asset expansion.

| Gate | Trades/day | WR | E[R] |
|------|--------:|---:|---:|
| S1 PLAYBOOK | ~0.8 | 74.6 % | +1.84R |
| S2 (urg ≥ 7 Tier A) | ~1.2 | 50.7 % | +0.52R |
| **S2' revised (urg ≥ 5 Tier A) ← upgrade** | ~2.1 | 54 % | +0.60R |
| Everything Tier A (urg ≥ 3) | ~3.5 | 49 % | +0.46R |
| Full supply cap | ~9 | mixed | mixed |

**Realistic ceilings:**
- This week: 2-4 trades/day at high edge
- End of month: 5-8/day (add S1 universe expansion)
- Month 2: 10-15/day (futures scanner online)
- 10-20/day = feasible by month 2-3 IF all expansion lines validate. Not guaranteed.

## Why urgency-ladder anomaly is a free lunch

The 5-6.99 urgency band has HIGHER WR (57.6 %) than 7+ (50.7 %). Dropping S2's floor from 7 → 5 doubles supply without eroding edge.

## Leverage (the "within reason" question)

Full Kelly on S1 = 65 % equity/trade = insane. Use fractional Kelly:

| Fraction | Risk/trade | Ruin risk |
|---------:|-----------:|----------:|
| ¼ Kelly | 16 % | 2 % of −50% DD |
| ⅛ Kelly | 8 % | <1 % |
| **Recommended: 1 %/trade** | 1 % | ~0 | 

**Per-trade risk: 0.5-1 %** of equity. **Max concurrent open risk: 5 %.** **Daily hard stop: −6 %.**

**"Leverage" post-PDT is really about holding more CONCURRENT positions** (5-10 at once), not bigger single bets. Broker margin enables it; sizing caps it. Don't max margin.

## Roadmap

**Week 1 (now):** S1 + S2' (urg ≥ 5 revision) at 1 %/trade. 2-4 trades/day expected.

**Week 2-3:** S1 universe expansion (498 syms) → +1-3/day if WR holds. Urgency-ladder anomaly diagnosis.

**Week 3-4:** Futures scanner (ES/NQ) online → +3-6/day. Target 8-12/day total.

**Week 6-8:** FX scanner + day_type stability gate → 10-15/day ceiling.

## What I REFUSE to recommend under scale pressure

- Drop urgency to 3 (edge decays fast)
- Trade past 13:30 ET (18-28 % WR absolute floor)
- Size up on urgency ≥ 9 (ladder anomaly says WORSE)
- 3-5 %/trade to "scale faster" (1 %/trade is the human-execution ceiling)
- Force trades on zero-fire days (40 % of days produce zero S2' signals — sit out)
- Port 5m rules to 15m / 30m without recalibration (pure speculation)

## Post-PDT behavioral risk

PDT was also a risk-control — it forced selectivity. Remove it, self-enforce quality. Audit daily counts weekly. First 2-3 weeks post-PDT, expect overtrading temptation.

## Canonical source

Full memo + tables: vault `Meta/Scale-Up Strategy Post-PDT 2026-04-19.md`. Both memos (Head of Strategy + this one) on GitHub aiedge-vault.
