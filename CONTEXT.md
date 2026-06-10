# Warehouse Map — Session Context
_Last updated: 2026-06-10_

## What this is
The Warehouse Map is a tool built inside the Global O-Ring internal tools platform (`globaloring/tools` on GitHub). It lets warehouse staff look up item locations, enter physical counts, and reconcile on-hand quantities by aisle.

## Where the code lives
- **Repo:** https://github.com/globaloring/tools
- **Branch:** `main` (all work merged)
- **Key files:**
  - `public/warehouse-map/warehouse-map.html` — entire tool (vanilla JS + CSS, ~1900 lines)
  - `app/(dashboard)/warehouse-map/page.tsx` — Next.js page wrapper (iframe)
  - `app/api/warehouse-map/route.ts` — Metabase proxy (question 3989)
  - `components/navbar.tsx` — nav entry
  - `app/(dashboard)/home/page.tsx` — home page card

## Live URL
https://tools.globaloring.com/warehouse-map

---

## Current UI state

### Table columns (left → right)
✓ | Location | Item | Lot # | On Hand | Found | +/− | Type

### Table header buttons
- **⊟ button left of "Location"** — collapse/expand all aisle sections; icon flips to ⊞ when all collapsed; color `#8bc34a`
- **⊟ button left of "Lot #"** — collapse/expand all MULTIPLE lot expansion rows; hidden until MULTIPLE rows exist

### TYPE chip colors
| Type | Background | Text |
|------|-----------|------|
| PALLET | `#4c6aa2` | `#ffffff` |
| WALK-UP | `#8ade28` | `#38761d` |
| WAVE | `#edb300` | `#ffffff` |
| PICKER | `#e06666` | `#660000` |

### Item badge color palette (8 slots, no duplicates allowed)
| Slot | Background | Text |
|------|-----------|------|
| 1 | `#666666` | `#8ade28` |
| 2 | `#434343` | `#8ade28` |
| 3 | `#0a388c` | `#8ade28` |
| 4 | `#4c6aa2` | `#8ade28` |
| 5 | `#666666` | `#ffffff` |
| 6 | `#434343` | `#ffffff` |
| 7 | `#0a388c` | `#ffffff` |
| 8 | `#4c6aa2` | `#ffffff` |

### Key behaviors
- **8-item cap** — inline red error shown below input if 9th item attempted
- **Unique colors** — each item gets its own slot; duplicate assignments from old localStorage are auto-fixed on restore
- **Location Types side panel** — 158px wide, PALLET / WALK-UP / WAVE / PICKER toggle buttons (mute/show)
- **Aisle sections** — collapsible: A11, A12, A13/14, A15/16, A17, A18
- **Multi-lot rows** — default OPEN (expansion shown on render); pulse animation when not fully reconciled; click MULTIPLE to collapse/expand; ⊟ in Lot # header collapses all
- **Empty state** — neon green wireframe magnifying glass SVG (`#00ff41`, 90s digital grid style, glow filter)

### localStorage keys
`wh-data`, `wh-colors`, `wh-ci`, `wh-found`, `wh-muted`, `wh-mutedtypes`, `wh-lotfilters`, `wh-invfilter`

---

## Tally box (left side panel, fixed position)

Each loaded item shows:

### Item row
- **Item badge** (color-coded) + **MUTE button** (right-aligned)
  - MUTE: `#e06666` bg, `#660000` text (PICKER red), `font-size: 0.56rem`
  - SHOW (when muted): `#8ade28` bg, `#38761d` text (WALK-UP green)

### OPEN / CLOSED / ALL buttons
Three small buttons (`font-size: 0.56rem`, same size as MUTE) in a row:
- **OPEN** (`#8ade28` / `#38761d`) — show only rows where that item's location has `onHand > 0`
- **CLOSED** (`#334155` / `#94a3b8`) — show only rows where `onHand === 0`
- **ALL** (`#0ea5e9` / `#ffffff`) — show all rows (default, starts active)
- Inactive buttons at 30% opacity; active at 100%. State stored in `wh-invfilter`.

### On Hand / Found
- Labels use `#748aa8` (tally-lbl class)
- Found color: gray if nothing entered, green (`#8ade28`) if found ≥ on hand, red (`#ef4444`) if less

### Lots: N ▼ (click to expand)
- Shows count of distinct lot numbers for that item
- Click the row to toggle a cascading-down lot search panel:
  - **Search input** — live-filters the chip list below it
  - **Lot chips** — vertical list, scrollable (max-height 180px); cyan (`#4ad4de`) unselected, blue (`#1d4ed8`) selected
  - Selecting a chip filters the map to only rows where that item has that lot
  - Multiple chips = union (any matching lot is shown)
  - **RESET** button (full-width, red) clears all lot selections
  - When filter active: row shows "X filtered" in amber (`#f59e0b`) + ▲
- State stored in `wh-lotfilters` as `{ "ITEM": [...selectedLots] }`

### Header MUTE buttons (legend chips at top)
- Same PICKER red style as tally MUTE button
- SHOW state: WALK-UP green
- `.is-muted-btn` class applied when item is muted

---

## Row visibility logic
A table row is hidden if ANY of the following:
1. Its aisle section is collapsed
2. Its item is in `mutedItems`
3. Its location type is in `mutedTypes`
4. Its lot(s) don't match the active `lotFilters[item]` (if any selected)
5. Its `onHand` total doesn't match `inventoryFilter[item]` (OPEN/CLOSED)

Expansion rows (MULTIPLE lot detail) follow the same rules PLUS `collapsedMultiKeys`.

---

## State variables
```js
let data = {};            // { "ITEM": [{lot, location, onHand}] }
let colorMap = {};        // { "ITEM": colorIndex }
let nextCI = 0;
let foundValues = {};     // { "ITEM|LOC|LOT": number }
let collapsedMultiKeys = new Set(); // item|location keys with collapsed expansion (session only)
const collapsed = new Set();        // aisle section labels currently collapsed (session only)
let mutedItems = new Set();
let mutedTypes = new Set();
let lotFilters = {};      // { "ITEM": Set([...lots]) }
let openLotSearch = null; // item with lot panel open
let inventoryFilter = {}; // { "ITEM": "OPEN"|"CLOSED" } — absent = ALL
```

---

## Data source
Metabase question **3989**, parameter `item_number`. Returns rows: `Lot #`, `Location`, `On Hand`.

Valid warehouse locations: aisles 11–18 only. VLM locations (01/02/03) are filtered out.

## Location type logic
| Type | Condition |
|------|-----------| 
| PALLET | Aisles 11, 12 |
| WALK-UP | Aisle 18 levels 1–2; Aisles 13–17 levels 1–4 |
| WAVE | Aisle 18 levels 3–7; Aisles 13–17 levels 5–10 |
| PICKER | Aisle 18 levels 8–10; Aisles 13–17 levels 11–22 |

## Dev workflow
- Local dev: `pnpm dev` → http://localhost:3000/warehouse-map
- Run checks before committing: `pnpm run check-all`
- Branch → PR → squash merge (always wait for explicit "merge it")
- Pushing UI tweaks goes to the open PR branch so Vercel preview updates

## Known pending work
None as of 2026-06-10.
