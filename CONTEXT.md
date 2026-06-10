# Warehouse Map — Session Context
_Last updated: 2026-06-10_

## What this is
The Warehouse Map is a tool built inside the Global O-Ring internal tools platform (`globaloring/tools` on GitHub). It lets warehouse staff look up item locations, enter physical counts, and reconcile on-hand quantities by aisle.

## Where the code lives
- **Repo:** https://github.com/globaloring/tools
- **Branch:** `main` (all work merged)
- **Key files:**
  - `public/warehouse-map/warehouse-map.html` — entire tool (vanilla JS + CSS, ~1400 lines)
  - `app/(dashboard)/warehouse-map/page.tsx` — Next.js page wrapper (iframe)
  - `app/api/warehouse-map/route.ts` — Metabase proxy (question 3989)
  - `components/navbar.tsx` — nav entry
  - `app/(dashboard)/home/page.tsx` — home page card

## Live URL
https://tools.globaloring.com/warehouse-map

## Current UI state

### Table columns (left → right)
✓ | Location | Item | Lot # | On Hand | Found | +/− | Type

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
- **Multi-lot rows** — expand inline; pulse animation when not fully reconciled
- **Empty state** — neon green wireframe magnifying glass SVG (`#00ff41`, 90s digital grid style, glow filter)
- **localStorage keys:** `wh-data`, `wh-colors`, `wh-ci`, `wh-found`, `wh-muted`, `wh-mutedtypes`

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
- Local dev: `cd C:\vinc_code\tools && pnpm dev` → http://localhost:3000/warehouse-map
- Run checks before committing: `pnpm run check-all`
- Branch → PR → squash merge (always wait for explicit "merge it")
- Pushing UI tweaks goes to the open PR branch so Vercel preview updates

## Known pending work
None as of 2026-06-10.