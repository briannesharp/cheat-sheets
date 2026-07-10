# Editing the stallion data

One YAML file per stallion in `stallions/`. Everything here is plain text —
edit in any editor (VS Code, Notepad), save, then run
`python scripts/generate_website.py` to publish.

## File layout

```yaml
name: Essential Quality        # must match the name in the site's PEDIGREES
profile:                       # overrides for scraped profile fields
  year_foaled: 2018 (8yo)
  height: 16.2½ hh
  earnings: $4.9M
  entered_stud: 2022
  first_crop_note: First crop are 3-year-olds
general:                       # Career Highlights bullets
  - ...
more_selling_points:           # "More selling points" bullets
  - ...
pedigree_highlights:           # Pedigree Highlights bullets
  - ...
career:                        # Career Highlights age groups
  at_two:
    subtitle: Champion 2YO Male   # optional heading
    items:
      - ...
progeny:                       # "Current Top Runners" — one line per horse
  - "THE PUMA (3c): won G3 Tampa Bay Derby on 3/7 (94). 106 Ky Derby points."
  - "DRAFT: NEW HORSE (2f): won MSW on debut at SAR 8/1 (85). Todd Pletcher."
```

## Rules of thumb

- **Order is display order** — move a line up or down to reorder it on the site.
- **Quote any text containing a colon** (progeny lines always do). Apostrophes
  are fine inside double quotes.
- **Lines starting with `DRAFT:` never appear on the site.** They are written
  by `scripts/update_progeny.py`. To publish one: edit the text if needed,
  then delete the `DRAFT: ` prefix. To reject it: delete the line.
- Comments start with `#` and are preserved — leave yourself notes freely.
- Adding a stallion: copy an existing file, change everything; the filename
  just needs to end in `.yaml` and be unique.

The old `stallion_data.xlsx` is retired and kept only as a pre-migration
backup (July 2026). Nothing reads it anymore.
