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
progeny:                       # "Current Top Runners" — machine-maintained
  - horse: THE PUMA (3c)       #   auto-refreshed (results, age)
    auto: won G3 Tampa Bay Derby on 3/7 (94), 2nd by a nose G1 Florida Derby 3/28 (100).
    note: 106 Ky Derby points. # yours — the machine never touches note
```

## Rules of thumb

- **Progeny entries update themselves** — `update_progeny.py` runs at the
  start of every site generation and prepends new qualifying results to
  `auto`. Don't edit `auto` unless you must (the machine rewrites it);
  put your editorial (Derby points, sale prices, trainer color) in `note`.
- An entry with an **empty note** disappears automatically once its newest
  result is older than 180 days. An entry **with a note** stays until you
  delete it.
- To reject a result the machine added, delete its clause from `auto` — the
  seen-log stops it coming back. To reject a horse's entry entirely, delete
  the whole entry (his *next* new result will recreate it, though).
- **Order is display order** — move an entry up or down to reorder it.
- Comments start with `#` and are preserved — leave yourself notes freely.
- Adding a stallion: happens automatically when he joins the SQL roster;
  a stub file is created for you to fill in highlights.

The old `stallion_data.xlsx` is retired and kept only as a pre-migration
backup (July 2026). Nothing reads it anymore.
