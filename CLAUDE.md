# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project does

Generates `index.html` — a single-page responsive website ("Darley America Stallion Cheat Sheets") used internally by sales staff. Run the generator:

```bash
python scripts/generate_website.py            # generates, commits, pushes
python scripts/generate_website.py --no-push  # generates only (local preview)
```

After generating, the script automatically stages `index.html`, `site.webmanifest`, and `sw.js`, commits with today's date, and pushes to GitHub (GitHub Pages hosts the site) — unless `--no-push` is passed.

## Key files

- `scripts/generate_website.py` — the entire pipeline in one file: data loading, HTML/CSS/JS generation, git push
- `scripts/update_progeny.py` — drafts new progeny black-type results from SQL into the YAML files as `DRAFT:` entries for human review (see below)
- `scripts/config.py` — SQL Server connection settings and queries; **excluded from git** (contains internal server names)
- `data/stallions/*.yaml` — one hand-edited file per stallion: profile overrides, highlight bullets, and the progeny list (see `data/README.md` for the schema and editing rules). Tracked in git — everything in it renders on the public site.
- `stallion_data.xlsx` — **retired** (July 2026). Pre-migration backup only; nothing reads it. Python deps for the YAML pipeline: `pyyaml` (generator, read) and `ruamel.yaml` (update_progeny, comment-preserving writes).

## Data flow

> **Writing/editing SQL against the Godolphin databases?** Use the **`godolphin-db`
> skill** (`Projects/.claude/skills/godolphin-db/SKILL.md`) — exact table/column
> names, which database each table lives in, encodings, and a read-only connection recipe.

0. **Stallion list (which stallions the site shows)** — the current-season Darley Kentucky roster from `Research.dbo.Stallions` (`Farm='Darley' AND State='KY' AND Hemisphere='N' AND Season=current`); falls back to the cached roster, then to `PEDIGREES` keys. A stallion dropped from the season table drops off the site.
1. **Stallion profiles & selling points** — scraped live from `darleyamerica.com` per stallion; falls back to `cache.json` (last successful scrape) if the page can't be fetched or parsed
2. **Fee history** — loaded from SQL Server (`config.py`), falling back to `cache.json`; **highlights & profile overrides** — loaded from `data/stallions/*.yaml`
3. **Auction/sale results** — scraped live from TDN insta-tistics; falls back to `stallion_data.xlsx` `SaleResults` sheet
4. **Pedigree data** — hardcoded `PEDIGREES` dict in the script (3 generations)
5. **Conformation photos** — fetched from `cdn.darleystallions.com` URLs defined in `PHOTO_URLS`; cached locally in `img_cache/` (excluded from git)

## config.py

`config.py` is gitignored. To run against the database, it must exist locally with valid `SERVER`/`DATABASE` settings. The script gracefully falls back to Excel if the DB connection fails.

## Adding a new stallion

Fully automatic: once he's in the current season of `Research.dbo.Stallions`
as Darley/KY, the next generate scrapes his darleyamerica.com page for the
pedigree, conformation photo, profile, and selling points, and auto-creates a
stub `data/stallions/<name>.yaml`. Manual work is only:

1. Fill in the stub YAML (highlights, progeny) — until then those sections
   are empty
2. Optional overrides if the scrape is wrong or missing (the generator warns):
   hardcoded `PEDIGREES` / `PHOTO_URLS` entries always beat scraped values

Scraped pedigrees/photos are persisted in `cache.json`, so a stallion whose
page later disappears (e.g. deceased) keeps rendering.

## YAML data schema

One file per stallion in `data/stallions/`. Keys (all optional except `name`):
`profile` (field overrides: `year_foaled`, `height`, `earnings`,
`entered_stud`, `first_crop_note`, plus reference-only fields), `general`,
`selling_points`, `more_selling_points`, `pedigree_highlights`, `career`
(`at_two`…`at_five`, each with optional `subtitle` + `items`), and `progeny`
(the "Current Top Runners" list — one quoted string per horse, list order =
display order). Progeny entries starting with `DRAFT:` are review-pending
output from `update_progeny.py` and are never rendered. Full editing guide:
`data/README.md`.

## Progeny update workflow (draft-then-review)

The `progeny` entries in the YAML files (hand-edited, one line per horse)
drive the site's "Current Top Runners" lists. To draft new results
automatically:

1. Run `python scripts/update_progeny.py` (no need to close anything —
   it's all text files)
2. The script queries `GBSWebsite.dbo.RaceResults` for black-type top-3
   finishes by current Darley KY roster progeny since the last run, plus
   `Research.dbo.TDNRisingStars` for new TDN Rising Stars (180-day lookback —
   they're usually MSW/ALW winners and marketing-wise on par with graded
   stakes winners; their draft says "TDN Rising Star."), formats everything
   in house style ("won G3 Tampa Bay Derby at TAM 3/7 (94)"), and appends
   `DRAFT:` entries to each stallion's `progeny:` list
3. Review in the YAML: edit the text, add editorial notes (Derby points,
   sale prices, etc.), fix any Beyers, then delete the `DRAFT: ` prefix to
   publish the line (or delete the whole line to reject it)
4. Run `generate_website.py` as usual

**The script only appends `DRAFT:` entries — it never edits hand-written
lines — and never runs git.** It uses `ruamel.yaml` round-trip mode, so
comments and formatting in the files survive. State lives in
`progeny_seen.json` (gitignored): a watermark date plus a log of every race
already drafted, so re-runs never duplicate. Each run re-queries a 7-day
overlap (Beyers post late), and skips results whose horse + m/d date already
appear in an existing progeny entry. `--dry-run` prints without writing;
`--since YYYY-MM-DD` overrides the watermark.

## Brand colours (CSS variables)

`--blue: #0037B2`, `--cyan: #00ABEE`, `--red: #E3140D` — Darley/Godolphin brand palette defined in the `CSS` string inside `generate_website.py`.
