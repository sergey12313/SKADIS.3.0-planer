# SKADIS 3.0 Planner

Interactive layout calculator for the **IKEA SKADIS Modular Pegboard 3.0** — a 3D-printable modular pegboard system by [AdamKozakGrafika](https://makerworld.com/ru/models/2572477-ikea-skadis-modular-pegboard-3-0).

🔗 **[Open Planner →](https://deart2k.github.io/SKADIS.3.0-planer/)**

---

## What it does

- **Board layout** — visual front view of your board with all module types color-coded, mirror-symmetric around the board centre
- **Wall mounting** — rear view showing bracket placement
- **Parts catalog** — every part with exact Bambu Studio plate number
- **Print list** — complete bill of materials, auto-calculated for your board size
- **Interactive highlights** — hover a part in the list to highlight it on the board, and vice versa
- 🆕 **Panel size exclusions** *(added 2026-11-15)* — click any module size (e.g. 240×240mm) to exclude exactly that panel from the layout, without touching the other sizes
- 🆕 **Filament colors** *(added 2026-11-15)* — paint every panel with a filament color, build a palette, use symmetric colour presets; saved in the config
- 🆕 **Config export / import** *(added 2026-11-15)* — save the whole plan (variant, board size, density, all exclusions, colours) to a JSON file and load it back later or on another device
- 🆕 **Mount exclusions** *(added 2026-04-11)* — skip individual mount points, whole rows, or whole columns (e.g. when a wire runs behind that part of the wall)

## How to use

1. Choose **print bed variant** — Mini, Standard, or XL (see below)
2. Set board **width** and **height** using the sliders (in 40mm cells)
3. In **Panel sizes**, click any size your printer can't handle to exclude it (see below)
4. Adjust **mount density** — how frequently the board is anchored to the wall
5. Switch between **Board layout**, **Wall mounting**, **Parts catalog** and **Filament colors** tabs
6. Paint panels with filament colours in **Filament colors** (or apply a preset)
7. Use the **Print list** in the sidebar as your shopping/printing checklist
8. Use **Configuration → Export / Import** to save or restore the whole plan
9. Toggle **RU / EN** in the top-right corner to switch language (the first visit follows the browser locale)
10. Toggle **☀ / ☾** to switch between dark and light themes

## Print bed variants

The 3D model comes in three variants for different printer build plate sizes. The planner supports all three — switch via the selector at the top of the sidebar.

| Variant | Max module size | Build plate | Module file | Plates |
|---|---|---|---|---|
| **Mini** | 160×160mm (4×4 cells) | ~180×180mm | `mini_comp.3mf` | 27 plates |
| **Standard** | 240×240mm (6×6 cells) | ~260×260mm | `standard_comp.3mf` | 27 plates |
| **XL** | 320×320mm (8×8 cells) | ~350×350mm | `XL_comp.3mf` | 35 plates |

Standard includes all mini plates (for modules ≤160mm) plus larger sizes. XL includes mini + standard + extra-large sizes.

### Panel size exclusions 🆕

> **New in 2026-11-15** — drop individual panel sizes your printer can't produce, without reducing everything else.

The **Panel sizes** panel lists every module size currently used by the layout, grouped into *Center*, *Top & bottom* and *Side*. Click a size to exclude exactly that panel — it is struck through and the planner re-tiles the board using the remaining sizes. Click it again to allow it back.

For example: a 240×200mm print bed can handle 240×200mm and 200×240mm panels, but not 240×240mm. Exclude **240×240** and the planner will re-tile that area from other sizes (for example 200×240 and 120×240) while 240×40mm top/bottom modules stay available.

Exclusions are stored in `localStorage` (`skadis-excl-sizes`) and survive board-size and variant changes. If a size becomes impossible in a smaller module set it is simply hidden until you switch back.

Technically, the layout picks per-axis module sizes whose cross-product avoids every excluded `width×height` pair. Top/bottom and side borders share the same column/row sets as the center, so seams stay aligned across the whole board.

The mount grid is built so that the spans between mounts form a palindrome around the board centre. The number of mount spans is chosen to minimise the total panel count, then larger panels win. Mirrored spans reuse mirrored tilings and the central span is tiled symmetrically, so the front view is left-right and top-bottom symmetric. Connectors and seam lines are drawn only at real module corners/edges, so a mixed-size layout never shows floating connectors.

### Config export / import 🆕

> **New in 2026-11-15** — save a plan and restore it later, or move it between browsers/devices.

**Configuration → Export** downloads a `skadis-config-<width>x<height>.json` file containing:

- module set (Mini / Standard / XL)
- board width and height
- mount density
- mount exclusions (columns, rows, points)
- excluded panel sizes
- filament palette, selected colour and per-panel paint assignments

**Configuration → Import** loads such a file, validates it, clamps board dimensions to the slider range and rebuilds the layout. Invalid or non-JSON files are rejected with an error message instead of changing the current plan.

### Filament colours 🆕

> **New in 2026-11-15** — plan which filament each printed panel uses.

The **Filament colors** tab shows the front view with paintable panels:

- the palette starts with eight common filament colours; use the colour picker + **Add color** to add your own and `×` on a swatch to remove it (removing a colour also clears its painted panels);
- select a colour and **click or drag** over panels to paint them; **Erase** removes paint, **Clear paint** resets everything;
- panel assignments are stored as `role|x,y,width×height`, so a repaint only applies to a panel that still exists in the current layout;
- presets paint a whole symmetric scheme in one click: **Gradient**, **Rings**, **Frame**, **Checker**, **Stripes**, **Solid**. Brackets stay unpainted so the mounting points remain visible.

Colours are saved in `localStorage` (`skadis-colors`) and included in the exported config. Like all other plan data, they survive reloads.

### Plate mapping per variant

**Mini** (`mini_comp.3mf`):

| Role | Plates | Sizes |
|---|---|---|
| Standard module | 01–16 | 40–160mm combinations |
| Top & bottom | 17–20 | 40–160mm wide |
| Side | 21–24 | 40–160mm tall |
| Corner TL+BL / TR+BR | 25 / 27 | 40×40mm |
| Connector | 26 | 20×20mm, 25 per sheet |

**Standard** (`standard_comp.3mf`) — adds:

| Role | Plates | Sizes |
|---|---|---|
| Standard module | 01–20 | 200–240mm combinations |
| Top & bottom | 21–22 | 200–240mm wide |
| Side | 23–24 | 200–240mm tall |
| Corner TL+BL / TR+BR | 26 / 27 | 40×40mm |
| Connector | 25 | 20×20mm, 25 per sheet |

**XL** (`XL_comp.3mf`) — adds:

| Role | Plates | Sizes |
|---|---|---|
| Standard module | 01–28 | 280–320mm combinations |
| Top & bottom | 31–32 | 280–320mm wide |
| Side | 29–30 | 280–320mm tall |
| Corner TL+BL / TR+BR | 34 / 35 | 40×40mm |
| Connector | 33 | 20×20mm, 50 per sheet |

## Module types

The system uses four different border module types — they are not interchangeable:

| Color | Type | Description |
|---|---|---|
| 🟡 Gold | Corner module | 4 corners only, 40×40mm |
| 🟢 Green | Top & bottom module | Top and bottom borders |
| 🔵 Blue | Side module | Left and right borders |
| ⬛ Dark | Standard module | Center fill |
| 🟠 Amber hatch | Wall bracket | Replaces module at mount point |

## Wall mounting

### Mount exclusions 🆕

> **New in 2026-04-11** — skip mount points where you can't anchor to the wall (e.g. hidden wires or pipes).

Pick a mode (`Column` / `Row` / `Point`) in the sidebar and click a mount point on the front view to toggle it. Click an excluded point again to bring it back. The selected items appear as chips below the mode buttons; press *Clear* to reset everything. Exclusions are persisted in `localStorage` and reset automatically when the board size changes.

### Mount parts

Each mount point uses **2 parts** from `SKADIS_modular_board_3.0_mounting.3mf`:

| Part | Plates |
|---|---|
| Corner mounts | pl. 01 |
| Side mounts | pl. 02 |
| Top & bottom mounts | pl. 03 |
| Mid board mount | pl. 04 |
| Corner module distance connector | pl. 05 |
| Side, top & bottom module distance connector | pl. 06 |
| Mid board module distance connector | pl. 07 |

## Repository layout

The planner is a **single self-contained HTML file** — no dependencies, no build step, works offline.

| File | Description |
|---|---|
| `index.html` | The planner app |
| `README.md` | This file |
| `AGENTS.md` | Developer/agent notes: architecture, invariants, testing checklist |
| `docs/edge-inset-future.md` | Postponed edge-inset feature notes |

## 3D model

**IKEA SKADIS Modular Pegboard 3.0** by AdamKozakGrafika
→ [MakerWorld](https://makerworld.com/ru/models/2572477-ikea-skadis-modular-pegboard-3-0)

The planner references these `.3mf` files by name, but they are **not** shipped in this repository — download them from MakerWorld:

| File | Contents |
|---|---|
| `SKADIS_modular_board_3.0_mini_comp.3mf` | Mini variant modules (≤160mm) |
| `SKADIS_modular_board_3.0_standard_comp.3mf` | Standard variant modules (200–240mm) |
| `SKADIS_modular_board_3.0_XL_comp.3mf` | XL variant modules (280–320mm) |
| `SKADIS_modular_board_3.0_mounting.3mf` | Wall mounting brackets (shared across variants) |

---

*This planner is an unofficial fan-made tool and is not affiliated with IKEA or AdamKozakGrafika.*
