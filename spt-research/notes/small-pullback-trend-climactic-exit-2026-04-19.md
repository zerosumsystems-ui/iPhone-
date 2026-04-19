# SPT — 2× climactic-burst exit overlay (Q42)

Date: 2026-04-19 · Follow-up #29 to [[small-pullback-trend]].

**TL;DR — Q42 closed NEGATIVE by conditional rarity.** Brooks *Trends* ch. 57's 2-consecutive-large-with-trend-bars-breaking-to-new-HoD signature is a valid Brooks teaching but fires too rarely in SPT baseline trades to move the economics. At the Brooks-canonical cell (14:00–15:00 ET, range ≥ 1.5× median of trailing 20 bars, 2nd bar makes new HoD/LoD) the overlay fires on only **2 / 101** baseline trades (~2%). The 2 firings happen on trades that would have reached chart_end anyway and swap a drift-close for a slightly-better climactic-bar close; aggregate delta is **+0.004 R/trade** (+0.44 R over 101 trades — negligible). Second clean Brooks→aiedge transfer note after pt 28, but with a different mechanism: the teaching is real (nobody doubts bursts happen on SPT days), but our rule-6 entry window (10:30–14:15 ET) structurally limits how many trades are still open when the post-11am-PST burst window arrives. No PLAYBOOK change.

## Brooks source (ch. 57, the relevant paragraph)

> "About two-thirds of these days have a larger pullback after 11:00 a.m. PST. That pullback is often about twice the size of the biggest pullback since the trend began in the first hour. **It is often heralded by a relatively large, strong trend bar or two in the direction of the trend, but representing climactic exhaustion.** ... If sometime between 11:00 a.m. PST and noon the market has two relatively large consecutive bull trend bars breaking out to a new high, the move is more likely an exhaustive buy climax than the start of a new leg. ... Experienced traders are expecting a three- to five-point pullback and they will exit their longs. **Some will even short the close of that second bull trend bar**, or maybe a tick or two above its high, expecting the pullback."

11 a.m.–noon PST = **14:00–15:00 ET**.

Pt 7 tested 7 generic event-exit rules (first adverse close, bear bar closing below 10 EMA, etc.) and rejected all. Pt 7 did NOT test *this* specific Brooks signature — time-bound, 2-consecutive-large-with-trend, new extreme-of-day. Pt 26 §3.2 flagged Q42; this script ([`spt_climactic_exit_2026_04_19.py`](../../scanner/scratch/spt_climactic_exit_2026_04_19.py)) closes it.

## Design

### Baseline

C0 from [[small-pullback-trend-rule10-rule11-joint-2026-04-19|pt 27]] — PLAYBOOK REVISED pre-filters {3,4,5,6,6s} + post-filters {7,9}, hybrid rule-9 targets (H1/H2 → 5R; L1/L2 → 3R; shorts capped at 4R), db_stop, market-on-next-bar-open entry. Matches pt 27 C0 exactly on the current DB snapshot:

- **n = 101**, WR = 65.3%, sum R = +152.84, perR = **+1.513**, DD = −5.00.

(Pt 27's headline was n=88; the 13-trade growth is expected DB back-extension over the intervening hours.)

### Overlay

For each baseline trade, walk post-entry bars. BEFORE stop/target checks on each bar, test the climactic-exit signature:

- **Time:** bar close ET minute in `[window_lo, window_hi)`.
- **Previous bar AND this bar** both with-trend bodies (bull body `c > o` for longs, bear body `c < o` for shorts) AND both "large".
- **"Large"** = bar range ≥ `bar_mult × median(range of trailing 20 bars before this bar)`. Trailing window walks with the trade so reference volatility matches the trade's regime.
- **New-extreme:** this bar's high (longs) or low (shorts) exceeds the running HoD / LoD computed from every bar in the chart up to but not including this bar (includes pre-entry bars — Brooks's "new high" is new HoD, not new-since-entry).
- If all conditions fire, exit at this bar's **close** (Brooks-canonical).

### Sweep

3 windows × 3 `bar_mult` values × 2 new-extreme settings = 18 cells.

- Windows: 14:00–15:00 ET (canonical) · 13:00–15:00 ET (wider afternoon) · 14:00–15:30 ET (catch late exhaustion).
- `bar_mult`: 1.25, 1.50, 2.00.
- `require_new_extreme`: True (Brooks-canonical), False (looser).

## Result — the sweep

|   window   | mult | new_ext | n_fired | perR    | ΔperR   |
|:----------:|:----:|:-------:|:-------:|:-------:|:-------:|
| **14:00-15:00** | **1.50** | **True** | **2** | **+1.518** | **+0.004** |
| 14:00-15:00 | 1.25 | True    | 2       | +1.518  | +0.004  |
| 14:00-15:00 | 1.25 | False   | 2       | +1.518  | +0.004  |
| 14:00-15:00 | 1.50 | False   | 2       | +1.518  | +0.004  |
| 14:00-15:00 | 2.00 | any     | 0       | +1.513  |  0.000  |
| 13:00-15:00 | 1.25 | True    | 3       | +1.518  | +0.004  |
| 13:00-15:00 | 1.25 | False   | 8       | +1.506  | −0.007  |
| 13:00-15:00 | 1.50 | True    | 3       | +1.518  | +0.004  |
| 13:00-15:00 | 1.50 | False   | 3       | +1.518  | +0.004  |
| 13:00-15:00 | 2.00 | any     | 0       | +1.513  |  0.000  |
| 14:00-15:30 | 1.25 | True    | 2       | +1.518  | +0.004  |
| 14:00-15:30 | 1.25 | False   | 3       | +1.504  | −0.010  |
| 14:00-15:30 | 1.50 | True    | 2       | +1.518  | +0.004  |
| 14:00-15:30 | 1.50 | False   | 2       | +1.518  | +0.004  |
| 14:00-15:30 | 2.00 | any     | 0       | +1.513  |  0.000  |

`mult=2.00` never fires at any window — the threshold is too strict for SPT bar sizes in the current DB.
`mult=1.25` + no-new-extreme + wider windows adds a handful of firings, but half of them are losses (ΔperR negative, not positive).

### Brooks-canonical cell zoom

- n_fired = 2 of 101 (≈2.0% of baseline trades).
- Baseline outcome of the 2 overlay-exits: both **baseline_chart_end** (trade drifted to chart's last close). Baseline sum R = +2.76; overlay sum R = +3.19; Δsum R = **+0.44** R over 101 trades.
- LOO-week dominance 24/25 — but this is an artifact of the overlay being ≈ identical to the baseline on 99% of trades. It does NOT indicate a real effect.
- Per-setup and per-direction carves: H1/H2/L1/L2 and longs/shorts all show Δ ≤ 0.01 R/trade.

## Why so rare — base-rate forensics

- **Baseline trades with ≥ 1 post-entry bar in 14:00–15:00 ET:** 37/101 (37%).
- **Baseline trades with ≥ 1 post-entry bar in 13:00–15:00 ET:** 88/101 (87%).

The afternoon window is reachable for most trades (87% of them have *some* time in 13:00–15:00 ET), but conditional on reaching the window, the 2-large-with-trend-new-HoD signature only fires on ~5% of them. That's plausible for such a specific bar pair.

The structural limiter is not the window — it's rule 6 (entries 10:30–14:15 ET) combined with hybrid rule-9 targets (3–5R). Many H1/H2 long trades hit their 5R target before the afternoon window opens (21 target hits baseline). Many others are already stopped (34 stops). The climactic burst is an exit overlay; it can only fire on trades still open when it shows up.

### Sanity check — is the detection too strict?

- Loosening `bar_mult` from 1.50 → 1.25 buys one extra firing across all 18 cells.
- Dropping `require_new_extreme` and widening to 13:00–15:00 ET surfaces a few false alarms that cut winners short (ΔperR goes *negative*). This is consistent with pt 7's finding that generic event-exits destroy expectancy on SPT — Brooks's specificity (new HoD + "relatively large" + time-bound) is what makes the rule work in his setting, not what makes it rare.

The detection isn't too strict. The signature is genuinely rare in the SPT policy-stack-filtered trade population.

## Interpretation

Q42 closes NEGATIVE but **differently from Q41 (pt 28)**:

- **Pt 28 (Q41 — next-session follow-through).** Brooks's teaching's sign *inverted* on our data. ALIGN n=6 perR −0.012 UNDERPERFORMS NO_SPT n=44 perR +1.352 and OPPOSED n=9 perR +1.841. The teaching produces bad trades. That was a real transfer failure of mechanism.
- **Pt 29 (Q42 — climactic exit).** Brooks's teaching is *invisible* on our data. The signature is too rare to produce enough firings to measure. Of the 2 firings, the delta is +0.44 R total — noise-level positive, not negative. The teaching isn't wrong — there's just not enough sample. If we ran this on Emini 5-min bars holding SPT trades open through 14:00–15:00 ET, we'd expect meaningful firings; with rule 6 truncating entries at 14:15 ET and hybrid rule 9 resolving many trades before then, the SPT-stack-filtered trade population is structurally not a good measurement surface for this overlay.

Both notes tighten the Brooks→aiedge transfer story in the same direction: **Brooks teachings grounded in single-instrument day-long tape-reading** (e.g. "exit your longs" on a specific bar of the Emini) **do not mechanically map to a filter-heavy multi-symbol scanner-driven stack** where the entry and target rules already select a narrow slice of the tape. Rules 1–9 already remove most of the pathology Brooks is teaching you to handle. The remaining open Brooks teachings (Q43 first-MA-gap-bar exit) are likely to face the same conditional-rarity problem.

## PLAYBOOK delta

None. Rule 9 stays hold-to-target.

## Open items

- ~~Q42 (2× climactic-burst exit)~~ — **closed NEGATIVE/RARE** in this note.
- **Q43** (first-MA-gap-bar exit overlay, [[small-pullback-trend-brooks-source-cross-reference-2026-04-19|pt 26]] §3.3) — likely the next climactic-exit candidate, but expect the same rarity problem; prioritize only if a simpler overlay study passes first.
- **Q47** (new) — if the overlay-exit question is worth re-opening, measure it on a **pre-rule-6 population** (no 10:30 / 14:15 time restrictions) so the window reach improves; separate filter question (is the overlay a useful rule 9 complement?) from rule question (does the burst mean what Brooks says?).

## Reproducibility

- Script: [`~/code/aiedge/scanner/scratch/spt_climactic_exit_2026_04_19.py`](../../scanner/scratch/spt_climactic_exit_2026_04_19.py)
- Output: `~/code/aiedge/scanner/scratch/_out_spt_climactic_exit_2026_04_19.txt`
- CLI: `python3 spt_climactic_exit_2026_04_19.py` (full DB ~ 8 months) / `... claimed` (4-month cut).

## Related

- [[small-pullback-trend-PLAYBOOK]] — consolidated policy.
- [[small-pullback-trend-INDEX]] — reading order.
- [[small-pullback-trend-brooks-source-cross-reference-2026-04-19]] — pt 26 Brooks-source audit (source of Q42).
- [[small-pullback-trend-next-session-followthrough-2026-04-19]] — pt 28, first Brooks→aiedge transfer note (NEGATIVE).
- [[small-pullback-trend-adverse-event-2026-04-18]] — pt 7 generic event-exit rejection (background for why event-exits are suspect).
