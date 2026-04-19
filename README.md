# iPhone briefings

Phone-readable summaries of autonomous research runs. Each subject has its own folder with a `README.md` (latest run) and an `ARCHIVE.md` (every run ever) you can open on GitHub mobile.

## 📚 Full archives — every research post ever written

- **[spt-research/ARCHIVE.md](spt-research/ARCHIVE.md)** — all 31 small-pullback-trend notes in reading order.
- **[trend-classification/ARCHIVE.md](trend-classification/ARCHIVE.md)** — every trend-classification increment with PDF + notes.

**Canonical source for every research post: the aiedge-vault GitHub repo** → [github.com/zerosumsystems-ui/aiedge-vault](https://github.com/zerosumsystems-ui/aiedge-vault). The phone-briefing folders here are mirrors; if anything is missing, the vault is authoritative.

## Active subjects (latest-run briefings)

- **[trend-classification/](trend-classification/)** — unifying the 13 parallel trend classifiers in `aiedge-scanner` into one canonical `TrendState`. Inventory complete (12 of 12 contributors wired). **Latest:** increment 19 — contributor-degeneracy audit.
- **[spt-research/](spt-research/)** — small-pullback-trend follow-up research arc. **Latest:** pt 31 — Brooks→aiedge transfer failure taxonomy; three consecutive NEGATIVE closes in pts 28/29/30 synthesized into Inversion/Rarity/Emptiness failure modes + a zero-compute vetting checklist.

## How this repo gets updated

Scheduled tasks in `~/.claude/scheduled-tasks/` push their findings here at end-of-run. Each run:

1. Updates the subject's `README.md` (latest TL;DR + headline figures)
2. Appends a row to the subject's `ARCHIVE.md` (new research = new row, never overwrites)
3. Drops new figures into `figures/` and long-form notes into `notes/`
4. Copies the run's PDF into `pdfs/`

The autonomous git author is `Claude (autonomous) <noreply@anthropic.com>`.
