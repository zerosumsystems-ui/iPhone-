# iPhone briefings

Phone-readable summaries of autonomous research runs. Each subject has its own folder with a `README.md` you can open on GitHub mobile.

## Active subjects

- **[trend-classification/](trend-classification/)** — unifying the 13 parallel trend classifiers in `aiedge-scanner` into one canonical `TrendState`. Inventory complete (12 of 12 contributors wired). Most recent run: increment 16 — empirical contributor redundancy study.

## How this repo gets updated

Scheduled tasks in `~/.claude/scheduled-tasks/` push their findings here at end-of-run. Each run updates the relevant subject's `README.md` (latest TL;DR + headline figures), drops new figures into `figures/`, and copies the long-form research notes into `notes/`.

The autonomous git author is `Claude (autonomous) <noreply@anthropic.com>`.
