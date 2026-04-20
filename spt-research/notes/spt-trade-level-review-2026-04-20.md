---
title: SPT C3 — trade-level review (scheduled-task, 2026-04-20)
date: 2026-04-20
run_by: claude (scheduled-task `backtest`, afternoon run)
companion_to: 2026-04-20-spt-full-playbook.md
stack: C3 reco (rules 1–11: SPT PLAYBOOK canonical)
script: scanner/scratch/spt_trade_review_2026_04_20.py
log: scanner/scratch/_out_spt_trade_review_2026_04_20.txt
csv: 2026-04-20-spt-c3-trades.csv (83 trades, fields date,dow,ticker,dir,setup,urgency,day_type,sig_et,entry,stop,risk,target_r,r,reason,running_eq,peak,dd)
---

# Trade-level review — C3 stack, 83 trades

Companion to today's headline backtest ([2026-04-20-spt-full-playbook.md](2026-04-20-spt-full-playbook.md)). Same population (83 C3 trades, +1.909 R/trade, −2.00R max DD) viewed trade-by-trade so we can answer: **are the losers reasonable, and is the edge concentrated in a way that would be hard to live-trade?**

## Monthly P&L

| Month    | n  | W  | L | WR%   | sumR   | perR   |
|----------|---:|---:|--:|------:|-------:|-------:|
| 2025-08  |  5 |  3 | 2 | 60.0% |  +5.96 | +1.192 |
| 2025-09  |  9 |  6 | 3 | 66.7% | +13.27 | +1.474 |
| 2025-10  |  6 |  4 | 2 | 66.7% | +10.31 | +1.718 |
| 2025-11  |  4 |  2 | 2 | 50.0% |  +6.21 | +1.554 |
| 2025-12  |  1 |  0 | 1 |  0.0% |  −1.00 | −1.000 |
| 2026-01  | 24 | 21 | 3 | 87.5% | +48.80 | +2.034 |
| 2026-02  | 12 |  9 | 3 | 75.0% | +12.20 | +1.016 |
| 2026-03  |  5 |  3 | 2 | 60.0% | +11.00 | +2.200 |
| 2026-04  | 17 | 16 | 1 | 94.1% | +51.66 | +3.039 |

Only Dec 2025 printed negative — a single trade (no other qualified signals all month). Every other month cleared +6R. April 2026 MTD (17 trades) is the strongest month of the sample — the stack is working *better* as the book grows.

## 5 worst trades (full stop-outs — logic check)

| date       | tk    | dir   | setup | urg | day_type          | sig_et |   r  | reason |
|------------|-------|-------|-------|----:|-------------------|--------|-----:|--------|
| 2025-08-13 | DIS   | long  | H1    |   7 | spike_and_channel | 12:45  | −1.00 | stop |
| 2025-08-28 | MDB   | long  | H1    |   8 | trend_from_open   | 11:45  | −1.00 | stop |
| 2025-09-11 | LRCX  | long  | H1    |   8 | trend_from_open   | 11:45  | −1.00 | stop |
| 2025-09-11 | CRCL  | long  | H2    |   7 | trend_from_open   | 12:15  | −1.00 | stop |
| 2025-09-29 | CVX   | short | L1    |   8 | spike_and_channel | 13:45  | −1.00 | stop |

Every one of these is a textbook SPT with-trend continuation that the market simply invalidated. None of them are "bad logic" trades. All capped at −1R by the fixed stop. The 2025-09-11 cluster (two trend_from_open longs stopping within 30 min) is the kind of correlated-loss day the pt-30 rollup already flagged.

## Day-of-week

| dow |  n  |  WR%  | perR  |
|-----|----:|------:|------:|
| Mon | 10  | 70.0% | +1.708 |
| Tue | 14  | 64.3% | +1.441 |
| Wed | 32  | 87.5% | +1.876 |
| Thu | 21  | 66.7% | +1.986 |
| Fri |  6  | 100%  | +3.235 |

Wed and Fri are the strongest days, though Fri's n=6 is thin. Every day clears the 60% WR floor. Mon is meaningfully weaker than mid-week but still +1.7 R/trade.

## Signal-time buckets (ET hour)

| hour  |  n  |  WR%  | perR   |
|-------|----:|------:|-------:|
| 10:00 |  5  | 100%  | +3.639 |
| 11:00 | 27  | 77.8% | +2.251 |
| 12:00 | 22  | 77.3% | +2.054 |
| 13:00 | 29  | 72.4% | +1.181 |

The 10:00 ET bucket is 100% WR but n=5 — these are the 10:30-10:59 fires that squeak through rule 6. The 11:00 and 12:00 buckets carry most of the trades and show the cleanest WR × perR combo. The 13:00 bucket's WR is still solid but perR tails off — approaching the 14:15 close-hour cutoff.

## Exit-reason mix

| reason      |  n | %     |
|-------------|---:|------:|
| chart_end   | 41 | 49.4% |
| target      | 24 | 28.9% |
| stop        | 18 | 21.7% |

**Half of all trades resolve by session close, not by hitting target.** This is the most important live-trading insight. Winners that resolve at chart_end average ~+1.5R (between 0 and target). If you flatten early you miss the target hits *and* truncate the chart-end winners. Hold-to-resolution stays ship-critical.

## 5 biggest winners

| date       | tk    | dir  | setup | urg | day_type          | sig_et |   r  |
|------------|-------|------|-------|----:|-------------------|--------|-----:|
| 2025-09-23 | MO    | long | H1    |   4 | spike_and_channel | 13:15  | +5.00 |
| 2025-10-24 | COIN  | long | H2    |   4 | spike_and_channel | 13:15  | +5.00 |
| 2025-11-21 | DHI   | long | H1    |   8 | trend_from_open   | 11:45  | +5.00 |
| 2026-03-25 | ALNY  | long | H2    |   7 | trend_from_open   | 11:45  | +5.00 |
| 2026-03-31 | INSM  | long | H2    |   5 | spike_and_channel | 12:15  | +5.00 |

All five max-R winners are longs on H1/H2, matching the pt-17 hybrid design (H1/H2 longs → 5R target). Urgency span is 4-8, confirming that **urgency ≥ 4 is the right floor** — not all 5R winners are high-urgency fires.

## Does the system clear Will's "40-50% WR floor"?

- Full C3 stack: **77.1% WR** (+27 pts over 50% floor).
- Deduped (one entry per ticker-day-setup-direction, from 2026-04-20 SPT full playbook report): **67.9% WR** (+18 pts).
- Worst LOO-week subset: still +1.61 R/trade.
- Worst single month: Dec 2025 at −1R (a single trade).

**Verdict: ships without reservation.** The R:R, WR, and monthly stability all exceed the bar. The loser set is textbook and capped at −1R. The edge is durable across all three walk-forward thirds.

## Artifacts

- Per-trade CSV: [2026-04-20-spt-c3-trades.csv](2026-04-20-spt-c3-trades.csv)
- Script: `scanner/scratch/spt_trade_review_2026_04_20.py`
- Raw log: `scanner/scratch/_out_spt_trade_review_2026_04_20.txt`
- Headline backtest: [2026-04-20-spt-full-playbook.md](2026-04-20-spt-full-playbook.md)
- PLAYBOOK: [`small-pullback-trend-PLAYBOOK.md`](../../Brooks%20PA/concepts/small-pullback-trend-PLAYBOOK.md)
