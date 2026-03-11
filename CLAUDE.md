# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this project does

Generates `index.html` — a single-page responsive website ("Darley America Stallion Cheat Sheets") used internally by sales staff. Run the generator:

```bash
python generate_website.py
```

After generating, the script automatically stages `index.html`, `site.webmanifest`, and `sw.js`, commits with today's date, and pushes to GitHub (GitHub Pages hosts the site).

## Key files

- `generate_website.py` — the entire pipeline in one file: data loading, HTML/CSS/JS generation, git push
- `config.py` — SQL Server connection settings and queries; **excluded from git** (contains internal server names)
- `stallion_data.xlsx` — Excel workbook with sheets: `Stallions`, `FeeHistory`, `SaleResults`, `Highlights`

## Data flow

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
- **Highlights**: `stallion_name`, `category` (`more_selling_point` or `pedigree_highlight`), `text`
- **Stallions**: `name` + profile fields (fee, foaled, earnings, etc.) — used as override/supplement to scraped data

## Brand colours (CSS variables)

`--blue: #0037B2`, `--cyan: #00ABEE`, `--red: #E3140D` — Darley/Godolphin brand palette defined in the `CSS` string inside `generate_website.py`.
