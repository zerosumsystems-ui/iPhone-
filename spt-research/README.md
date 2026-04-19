# SPT research — phone briefing

Autonomous small-pullback-trend research notes, phone-readable. Long-form notes live in [`notes/`](notes/); phone-friendly PDFs in [`pdfs/`](pdfs/); the latest TL;DR is at the top of this README.

> 📚 **Full research archive — [ARCHIVE.md](ARCHIVE.md)** lists every SPT note ever written (31 notes in reading order + the PLAYBOOK + the INDEX). Canonical mirror at the aiedge-vault: [github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts](https://github.com/zerosumsystems-ui/aiedge-vault/tree/main/Brooks%20PA/concepts).

**Last updated:** 2026-04-19 · pt 31 · [Download this run's PDF](pdfs/spt-research-2026-04-19-pt31.pdf)

## Latest — pt 31: Brooks→aiedge transfer failure taxonomy (synthesis)

**Run:** 2026-04-19, scheduled-task autonomous. Synthesis pass after pts 28/29/30 closed the Brooks-source trilogy negative.

### Headline

Three Brooks→scanner transfer failures in a row (Q41, Q42, Q43) are not isolated — they're three distinct **modes** of the same underlying problem. This note names them, proposes a three-check zero-compute vetting checklist that would have pre-screened all three, and states where the arc's remaining +R actually lives.

**No PLAYBOOK change.** The PLAYBOOK still reads **+1.84 R/trade at −2R max DD** on the C3 combined stack.

### The three failure modes

| Mode | Q (pt) | Signature | Mechanism |
|---|---|---|---|
| **Inversion** | Q41 · pt 28<br/>next-session follow-through | perR sign **opposite** Brooks's prediction. ALIGN n=6 −0.01 vs OPPOSED n=9 **+1.84** | Scanner's pullback-as-signal-bar inverts Brooks's "continuation with pullback entry" label. Brooks's ALIGN = scanner's OPPOSED. |
| **Rarity** | Q42 · pt 29<br/>2× climactic-burst exit | `n_fired / n_baseline < 5%`. Only **2/101** fires at canonical cell | Rule 6 truncates entries at 14:15 ET; hybrid rule 9 resolves most trades before 14:00. Brooks's last-hour window is pre-empted. |
| **Emptiness** | Q43 · pt 30<br/>first MA-gap bar exit | `n_fired_conditional = 0`. 16/101 see a gap bar; **0/16** are ≥ +1.5R | Trigger X and condition Y are anticorrelated on survivors. By the time pullback is deep enough, mtm averages −1.66R. |

### Why concepts port and mechanics don't

- **Concepts** describe properties of the bar/window the scanner has NOW (H2 = second pullback; urgency = bar texture; SPT = shallow pullbacks). Scanner computes the same properties → **port clean**.
- **Mechanics** describe actions conditional on bars the scanner *hasn't seen yet* (post-entry exits, next-session follow-through). The stack-selected post-entry bar distribution ≠ raw single-instrument tape distribution → **port badly**.
- Pt 26's Brooks-source audit: **9/11 PLAYBOOK rules are Brooks-grounded and empirically positive** — all concept ports. Rules 8, 10 are pure-empirical. Three Brooks-mechanic candidates closed **0/3 positive**.

**Brooks-source well effectively exhausted** for SPT US single-name equities.

### The vetting checklist (zero-compute, runs on C0 baseline)

Run BEFORE writing the next backtest script. All three would have predicted Q41/Q42/Q43 negative.

1. **Label-polarity reconciliation.** Does Brooks's trigger bar coincide with the scanner's signal bar, or is one bar off? If Brooks's "pullback bar" = scanner's signal bar, invert the ALIGN/OPPOSED label before backtesting.
2. **Base-rate on selected population.** `n_fired / n_baseline` — **< 10% → rare, skip**.
3. **Conditional joint-probability.** If Brooks's rule is conditional ("do X given Y"), compute `n(Y AND X) / n(X)` on the baseline — **< 10% → empty, skip**.

Flagged as a future `spt_brooks_vetting.py` utility — build-when-needed.

### Where +R lives next

- **Scanner-side schema enrichments** (blocked on migrations): Q1 raw component scores, Q5 cross-asset, Q25 day_type label stability.
- **Conditional-rule maturation** (data-growth-gated): Q33 rule-10 at n_shorts ≥ 80; **Q44 weekly tracking of rule-11 solo Δ** (measured +0.31 at pt 23 → +0.21 at pt 27, need 4 weekly readings to establish noise floor); Q45 L2-short; Q46 OPPOSED morning reversal cell.
- **NOT** more Brooks-source excavation on SPT US single-names.

### Adoption decision

**No PLAYBOOK change.** This is a governance-layer note — it names the failure modes for future Brooks ports and states where the arc's center of gravity moves next. Rules 1-11 unchanged; sizing tags unchanged; economics unchanged.

---

## Why this matters for trading

You don't need to do anything. The PLAYBOOK still delivers +1.84 R/trade at −2R max DD. What changed is the *governance* layer — future Brooks-candidate rules get routed through the three-check checklist first, so we stop burning compute on structurally impossible ports. When this note's checklist predicts the next candidate negative, we skip the backtest and move on.

---

## What's next — needs your nod

Nothing from this run. Synthesis notes don't propose rule changes.

Open from earlier runs (still pending your nod):
- **Hybrid rule 9** adoption (H1/H2 longs & H1 shorts → 5R, H2 shorts → 4R, L1/L2 → 3R). Walk-forward positive, +0.36R/trade over 3R uniform. From pt 17.
- **Conditional rule 10** (2-bar swing stop on shorts). 24/25 LOO-weeks favor. From pt 21.
- **Conditional rule 11** (drop signal bars with `opp_tail ≥ 0.25`). 25/25 LOO-weeks + 52/52 LOO-symbols favor. DD floor lifts from −4 to −2. From pt 23.

---

## Source

- Long-form note: [`notes/small-pullback-trend-brooks-transfer-taxonomy-2026-04-19.md`](notes/small-pullback-trend-brooks-transfer-taxonomy-2026-04-19.md)
- PDF: [`pdfs/spt-research-2026-04-19-pt31.pdf`](pdfs/spt-research-2026-04-19-pt31.pdf)
- PLAYBOOK: `~/code/aiedge/vault/Brooks PA/concepts/small-pullback-trend-PLAYBOOK.md`
- Reading-order index: `~/code/aiedge/vault/Brooks PA/concepts/small-pullback-trend-INDEX.md`
- Sibling failure notes: pt 28 (Q41 inverted), pt 29 (Q42 rare), pt 30 (Q43 empty) — see `notes/`.

---

## Run history

- **2026-04-19 · pt 31** — Brooks transfer failure taxonomy (synthesis). Names Inversion/Rarity/Emptiness failure modes; proposes 3-check vetting checklist; states arc's forward priorities. No PLAYBOOK change.
- **2026-04-19 · pt 30** — Q43 first MA-gap-bar exit overlay closed NEGATIVE/EMPTY. Brooks-canonical cell fires 0/101; 16/101 trades ever see a gap bar with mtm averaging −1.66R. Third Brooks→aiedge transfer failure.

---

## TL;DR in plain English

We keep a 9-rule playbook for trading "small pullback trend" days. It makes about **+1.84 of risk per trade** — bet $100 to win/lose, average **+$184**.

For months we've been mining Al Brooks's books for new rules to add. This week we tested the last three candidates he gave us. **All three failed.** Not by a little — by a lot. Each failed for a different reason, and that's the interesting part:

1. **The first rule fired on the wrong side.** Brooks said "keep going"; our data said "turn around." It turns out we labeled the same moment in the chart with the opposite word Brooks used. Lost in translation, not lost in the market.
2. **The second rule almost never happened.** Brooks's signal only shows up in the last hour of the day. Our playbook already filters out late-day trades. So his rule fired 2 times out of 101 trades — too rare to matter.
3. **The third rule was a paradox.** Brooks said "exit if you're winning when the warning sign appears." Problem: by the time the warning sign appears in our data, the trade is already losing. You can't lock in a win that doesn't exist yet.

**Big lesson:** Brooks's *ideas about what a good trade looks like* work great — that's why 9 of our 11 rules are his. Brooks's *ideas about when to get out of a trade* don't work for us — because our filters already threw away most of the bad trades, so his exit signals fire on a picked-over population that behaves differently than his raw charts.

**Now what:**

- **Stop** asking Brooks's books for more rules. That well is dry for this setup.
- **Next time** a Brooks-rule candidate comes up, run it through a simple 3-question checklist *before* writing any code. The checklist would have caught all three of this week's failures in minutes instead of days.
- **Real gains from here** will come from (a) upgrading what data the scanner remembers — we can't ask a lot of questions right now because some info is thrown away, (b) waiting for more trades so the rules we've already tested get sharper numbers, and (c) expanding past US stocks into futures and forex.
- **Nothing to change today.** The trading rules you're using still work. This was a "sharpen the saw" week, not a "cut a new tree" week.
