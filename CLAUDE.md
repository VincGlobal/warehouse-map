# Warehouse Walk-Path Map Tool — Project Context

## What this is
A single-file web app (`warehouse-map.html`) that replaces a Google Sheets / Apps Script workflow for tracking lost inventory in a warehouse. ERP CSV exports are dropped onto the page, the user enters an item number, and the tool builds a sorted "walk-path map" warehouse staff can follow physically to check locations.

## The file
Everything is in `warehouse-map.html` — vanilla JS, no framework, no build step. Open it directly in a browser.

## Reference scripts (on this machine at work, if needed)
The original Google Sheets scripts are at:
`C:\Users\<work-user>\Desktop\vinc_code_home\inventory_discrepancy\`
Google Apps Script project ID: `1z6bOuDnK3wIkZlM7CE_5RzTl7HRklqI0RxaTZEn8D19YifYJsJu6vhpv`
Key file: `Aisle Manager.js` — defines canonical section order and sort algorithm.

---

## Warehouse layout rules

### Location format
- `AA-BB-CC` (3-part): **A13 and A18 only**
- `AA-BB-CC-DD` (4-part): **A11, A12, A14, A15, A16, A17**
- AA = Aisle, BB = Bay/Rack, CC = Level, DD = Position
- VLM locations (01/02/03) and anything outside A11–A18 are excluded

### Aisle groupings (match physical walk path)
| Section | Aisles | Notes |
|---------|--------|-------|
| A11 | 11 | One-sided |
| A12 | 12 | One-sided |
| A13/14 | 13 + 14 | Two-sided, interleaved by bay |
| A15/16 | 15 + 16 | Two-sided, interleaved by bay |
| A17 | 17 | One-sided |
| A18 | 18 | One-sided, 3-part format |

### Location categorization (by Level = CC)
- AA 11 or 12 → **PALLET**
- AA 13–17: levels 1–4 = **WALK-UP**, 5–10 = **WAVE**, 11–22 = **PICKER**
- AA 18: levels 1–2 = **WALK-UP**, 3–7 = **WAVE**, 8–10 = **PICKER**

### Sort algorithm
Mirrors `=RIGHT(C, LEN(C)-2)` from Aisle Manager.js: strip the 2-char aisle prefix, sort alphabetically. Section order is the primary sort key so sections never interleave.

---

## Features built so far

### Core
- Drag-and-drop or click-to-upload CSV (always named `export.csv`)
- Modal prompts for item number on every drop
- Multiple items loaded simultaneously, each color-coded
- New CSV for same item replaces it; new CSV for new item adds to map
- All state persists in localStorage (survives page refresh)

### Table columns (left to right)
`✓ | Location | Item | Lot # | Type | On Hand | Found | +/−`

### MULTIPLE lots
- Same item + same location with >1 lot → shows **MULTIPLE** in Lot# column
- MULTIPLE cell pulses (background: row-color → green) until all Found fields are filled
- Clicking MULTIPLE expands an inline sub-row showing per-lot Found inputs + diffs
- When collapsed, the +/− column shows aggregate **▲** (found > on-hand) or **▼** (found < on-hand)

### Check column
- Green ✓ appears in col 0 when all Found values for that row are filled
- For MULTIPLE rows: all lots must have entries

### Floating tally box
- Fixed to left side of screen, glides with scroll
- Shows per-item: On Hand total vs Found running sum
- Found color: green if found ≥ on-hand, red if less, gray if nothing entered yet

### Mute toggle
- Each item chip in the header has a **MUTE** button
- Muted items: rows hidden in table, chip turns grey, item stays in tally box
- Persisted in localStorage

### Visual design
- Background: `#0f172a` | Row background: `#192844`
- Location / Lot# / On Hand text: `#4ad4de`
- Drop zone: solid `#8ade28` border
- Item names: colored chip badges
- Category badges: PALLET / WALK-UP / WAVE / PICKER
- Found input: centered, no spinner arrows

---

## localStorage keys
`wh-data`, `wh-colors`, `wh-ci`, `wh-found`, `wh-muted`

---

## How to push updates to GitHub
```bash
git add warehouse-map.html
git commit -m "describe what changed"
git push
```

## GitHub Pages URL (if enabled)
`https://vincglobal.github.io/warehouse-map/warehouse-map.html`
