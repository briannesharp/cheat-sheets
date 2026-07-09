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
- `scripts/update_progeny.py` — drafts new progeny black-type results from SQL into the `ProgenyDrafts` sheet for human review (see below)
- `scripts/config.py` — SQL Server connection settings and queries; **excluded from git** (contains internal server names)
- `stallion_data.xlsx` — Excel workbook with sheets: `Stallions`, `Highlights`, `ProgenyDrafts`

## Data flow

> **Writing/editing SQL against the Godolphin databases?** Use the **`godolphin-db`
> skill** (`Projects/.claude/skills/godolphin-db/SKILL.md`) — exact table/column
> names, which database each table lives in, encodings, and a read-only connection recipe.

1. **Stallion profiles & selling points** — scraped live from `darleyamerica.com` per stallion; falls back to hardcoded `_SP_FALLBACK` dict in the script if scraping fails
2. **Fee history & highlights** — loaded from SQL Server (`config.py`) if available, otherwise from `stallion_data.xlsx`
3. **Auction/sale results** — scraped live from TDN insta-tistics; falls back to `stallion_data.xlsx` `SaleResults` sheet
4. **Pedigree data** — hardcoded `PEDIGREES` dict in the script (3 generations)
5. **Conformation photos** — fetched from `cdn.darleystallions.com` URLs defined in `PHOTO_URLS`; cached locally in `img_cache/` (excluded from git)

## config.py

`config.py` is gitignored. To run against the database, it must exist locally with valid `SERVER`/`DATABASE` settings. The script gracefully falls back to Excel if the DB connection fails.

## Adding a new stallion

1. Add an entry to `PEDIGREES` in `generate_website.py`
2. Add a photo URL to `PHOTO_URLS`
3. Add fallback selling points to `_SP_FALLBACK`
4. Add rows to the relevant sheets in `stallion_data.xlsx`
5. Add the stallion to the current season in the SQL DB (for DB-sourced fee history)

## Excel sheet schemas

- **FeeHistory**: `stallion_name`, `season`, `stud_fee`, `mares_bred`, `CI`, `CPI`, `Foals`, `runners`, `black_type_winners`, `SW_percent`, `notes`
- **SaleResults**: `stallion_name`, `year`, `sale_type`, `ring`, `sold`, `average`, `median`, `top_colt`, `top_filly`
- **Highlights**: `stallion_name`, `category`, `sort_order`, `subtitle`, `text`. Categories: `general`, `selling_point`, `more_selling_point`, `pedigree_highlight`, `progeny` (the "Current Top Runners" list), `at_two`/`at_three`/`at_four`/`at_five` (Career Highlights age groups)
- **ProgenyDrafts**: machine-written staging rows from `update_progeny.py` — `stallion_name`, `horse_name`, `draft_text`, `trainer`, `race_dates`, `added_on`, `notes`. Never rendered on the site.
- **Stallions**: `name` + profile fields (fee, foaled, earnings, etc.) — used as override/supplement to scraped data

## Progeny update workflow (draft-then-review)

The `progeny` rows in Highlights (hand-edited, one row per horse) drive the
site's "Current Top Runners" lists. To draft new results automatically:

1. Close `stallion_data.xlsx` in Excel, then run `python scripts/update_progeny.py`
2. The script queries `GBSWebsite.dbo.RaceResults` for black-type top-3
   finishes by current Darley KY roster progeny since the last run, plus
   `Research.dbo.TDNRisingStars` for new TDN Rising Stars (180-day lookback —
   they're usually MSW/ALW winners and marketing-wise on par with graded
   stakes winners; their draft says "TDN Rising Star."), formats everything
   in house style ("won G3 Tampa Bay Derby at TAM 3/7 (94)"), and appends
   rows to the `ProgenyDrafts` sheet
3. Review in Excel: merge keepers into the `Highlights` sheet (category
   `progeny`), add editorial notes (Derby points, sale prices, etc.), fix any
   Beyers, delete the draft rows
4. Run `generate_website.py` as usual

**The script never touches the Highlights sheet and never runs git.** State
lives in `progeny_seen.json` (gitignored): a watermark date plus a log of
every race already drafted, so re-runs never duplicate. Each run re-queries a
7-day overlap (Beyers post late), and skips results whose horse + m/d date
already appear in a hand-written progeny row. `--dry-run` prints without
writing; `--since YYYY-MM-DD` overrides the watermark.

## Brand colours (CSS variables)

`--blue: #0037B2`, `--cyan: #00ABEE`, `--red: #E3140D` — Darley/Godolphin brand palette defined in the `CSS` string inside `generate_website.py`.
