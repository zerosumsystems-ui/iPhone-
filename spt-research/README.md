# SPT research — phone briefing

Autonomous small-pullback-trend research notes, phone-readable. Long-form notes live in [`notes/`](notes/); phone-friendly PDFs in [`pdfs/`](pdfs/); the latest TL;DR is at the top of this README.

> 📚 **Full research archive — [ARCHIVE.md](ARCHIVE.md)** lists every SPT note ever written (33 notes in reading order + the PLAYBOOK + the INDEX). Canonical mirror at the aiedge-vault: [github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts](https://github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts).

**Last updated:** 2026-04-19 · pt 33

## Latest — pt 33: multi-timeframe SPT candidate scan

**Run:** 2026-04-19 evening, scheduled-task autonomous. First operational (not research) SPT post — takes the 11-rule PLAYBOOK off paper and scans 52 symbols × 5 timeframes for stocks whose tape currently matches the small-pullback-trend signature.

Data: Databento `XNAS.ITCH` through Fri 2026-04-17 cash close. Window-composite score = 0.5·|net_R| + 2.0·closeness-to-extreme + 2.0·pullback-penalty. Timeframe-scaled gates (intraday widens). Grade **A** = strong on all three legs; **B** = inside gate; **C** dropped.

### Stock recommendations per timeframe

**Daily** (position / swing, weeks)

1. **AVGO** · +6.61 netR · pullback 0.39 · close 0.99 · 20d +29.6% · $406.06
2. **AMD** · +6.65 · 0.38 · 0.97 · +36.9% · $277.25
3. **AMZN** · +5.94 · 0.37 · 0.91 · +21.1% · $250.70
4. **BAC** · +5.07 · 0.36 · 0.83 · +13.6% · $53.93

*No short-side daily SPT passes the gate — broad bull.*

**60m** (intraday swing, 1-3 sessions)

1. **DIA** · +7.69 netR · pullback 0.41 · close 0.78
2. **CAT** · +6.64 · 0.43 · 0.82
3. **SPY** · +5.97 · 0.38 · 0.82

*60m is the same run as daily, re-expressed.*

**30m** (intraday, hours) — **defensive rotation shows up here**

1. **MRK** · +9.84 netR · 0.48 · 0.96
2. **DIA** · +9.55 · 0.40 · 0.74
3. **HD** · +8.93 · 0.43 · 0.83
4. **DIS** · +7.86 · 0.54 · 0.98
5. **UNH** · +7.45 · 0.55 · 0.91

**15m** (scalp-to-swing, tens of min) — **software-complex divergence opens here**

LONG: WMT (A), MRK, DIS, GOOGL, CAT
**SHORT: ADBE (A), CRM (A), ORCL**

**5m** (scalp, minutes)

LONG: **WMT (A)**, COST, **MRK (A)**, **UNH (A)**, **GOOGL (A)**
SHORT: **CRM (A)**, **ADBE (A)**, GE, ORCL, MU

### Cross-timeframe picks (signal appears on ≥ 2 TFs)

| Symbol | Direction | Timeframes | Grade |
|--------|:---------:|------------|:-----:|
| **WMT**  | long  | 15m · 5m | A/A |
| **MRK**  | long  | 30m · 15m · 5m | B/B/A |
| **CRM**  | short | 15m · 5m | A/A |
| **ADBE** | short | 15m · 5m | A/A |
| **DIS**  | long  | 30m · 15m | B/B |
| **UNH**  | long  | 30m · 5m  | B/A |
| **DIA**  | long  | 60m · 30m | B/B |
| **CAT**  | long  | 60m · 15m | B/B |
| **GOOGL** | long | 15m · 5m  | B/A |
| **ORCL** | short | 15m · 5m | B/B |

### The story

Daily: broad tape is green — tech (AVGO/AMD/AMZN) and financials (BAC) in clean SPT uptrends. 60m/index ETFs confirm.

Intraday (30m and finer): **defensive rotation** — bid moved into MRK/UNH/HD/DIS/WMT in the last hours of Friday's cash.

Simultaneously: **software-complex SPT-short** at 15m/5m — CRM and ADBE both A-grade short on both timeframes, ORCL B-grade short on both. Enterprise software is diverging from the daily mega-cap-tech uptrend. Cleanest cross-timeframe divergence in the scan.

### How to use this list

- **Universe pre-filter for the live scanner** — when H1/H2 fires on a name from the long list, that's a highly-aligned cross-TF signal.
- **Timeframe-of-trade anchor** — scalping 5m? pick from the 5m list. Position swinging? daily list.
- **Divergence flag** — CRM/ADBE/ORCL longs are a dangerous cell; shorts on those three names are setup-selection-favored.

### Caveats

- No live `day_type` label (PLAYBOOK rule 2) — static scan doesn't know Monday's day-type. Live scanner adds that layer.
- Last close is Fri 04-17; Monday's tape can invalidate. The 5m list is hours-old by open.
- Universe is 52 hand-picked names; smaller active SPTs outside the universe are invisible here.

### Follow-ups

- **Q47** — cross-timeframe-SPT backtest. Does requiring SPT on ≥2 TFs lift perR? Needs per-bar TF stacking in scanner.
- **Q48** — software-complex SPT-short persistence. Re-scan EOD Monday to see if CRM/ADBE/ORCL cluster holds.

Full vault note: [small-pullback-trend-multi-tf-candidates-2026-04-19.md](https://github.com/zerosumsystems-ui/aiedge-vault/blob/main/Brooks%20PA/concepts/small-pullback-trend-multi-tf-candidates-2026-04-19.md)

---

## Previous — pt 32: rule-11 setup-level concentration

(pt 32 headline archived in [ARCHIVE.md](ARCHIVE.md)). Zero-compute decomposition of pt 27 §5. Rule 11's +0.19 R/trade lift concentrates at H1-long (+0.30) and L1-short (+0.15); neutral on H2-long; L2-short n=3 artifact. Conditional-variant analysis: C3ʹ (rule 11 on H1-long + L1-short only) gives identical perR at +11% throughput but **widens DD floor 50%** (−2 → −3). Decision: keep uniform C3. Rule 11's primary operational value is DD improvement.
