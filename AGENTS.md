# AGENTS.md — working on SKADIS 3.0 Planner

Guidance for AI agents and developers modifying this repository.

## Project shape

The app is a **single self-contained HTML file**: `index.html` (HTML + CSS + vanilla JS,
no build step, no npm dependencies, no bundler). It must keep working offline by opening
the file or serving it statically.

| File | Purpose |
|---|---|
| `index.html` | entire planner (UI, i18n, layout engine, canvas renderer) |
| `README.md` | user-facing docs — update it for every user-visible change |
| `docs/edge-inset-future.md` | notes on the abandoned edge-inset feature and layout traps |
| `AGENTS.md` | this file |

Do not introduce frameworks, package managers, generated assets or a build pipeline
unless explicitly requested.

## Mental model of the layout engine

`go()` is the entry point. It reads the UI state (variant, width, height, density),
then runs:

1. `computeMountPts(wC, hC, gapH, gapV)` → `pickBounds(total, gap)`
   Builds a **symmetric mount grid**. `pickBounds` creates palindromic spans between
   mounts (via `balancedPalindromic`) and searches the number of spans to minimise the
   total panel count. It returns boundary coordinates `[0, ..., total]`.
2. `computeLayout(wC, hC, pts)`
   - `bestAxisMasks(...)` chooses one allowed **column-size set** and one **row-size set**.
     Top/bottom borders use the column set; left/right borders use the row set, so seams
     stay aligned across the whole board.
   - `segsWithAllowed(...)` tiles each span. Mirrored span pairs get mirrored tilings;
     the central span uses `palTile` and is therefore a palindrome.
   - `computeConnectors(...)` / `computeSeams(...)` derive ornaments from actual modules.
3. `computeBOM(lay, pts)` groups modules, counts brackets and connectors, and resolves
   plate numbers via `plateFor()` / `connectorInfo()`.
4. Rendering: `drawBoard`, `drawMount`, `renderBOM`, `renderCatalog`,
   `renderExclusions`, `renderSizeExclusions`, `renderPalette`, `drawColorView`.
   `drawColorView` is the interactive paint canvas for the **Filament colors** tab.

### Critical invariants

- **Coverage**: every board cell is covered exactly once. `sum(m.wC * m.hC) === wC * hC`.
- **Symmetry**: for symmetric mount exclusions, the module layout is a mirror image
  both horizontally and vertically. Mount spans are palindromes; mirrored spans get
  mirrored tilings; the centre span is palindromic.
- **Large panels first, then symmetry**: the span-count search minimises the total
  number of panels; tie-breaks prefer larger panels. Do not add symmetry constraints
  that increase the panel count unnecessarily.
- **Size exclusions** (`EXCL_SIZES`, keys `role|wCxhC`) must never appear in
  `G.lay.mods`. The 1-cell (40 mm) size is always allowed because mount cells and short
  forced spans depend on it; the UI hides 40×40 toggles for this reason.
- **Connectors** must only be placed at real module corners. **Seams** must only follow
  real module edges. Never go back to a global boundary cross-product: it creates
  floating dots/lines when rows and columns do not align.
- `computeMountPts` drops stale exclusions if a position no longer exists. Keep that
  behaviour: exclusions are stored in cell coordinates and density/board changes can
  invalidate them.
- The dead-zone filter in `computeMountPts` (positions 1 / `total-1`) is load-bearing —
  see `docs/edge-inset-future.md`. Do not remove it without re-testing all variants.

## Persistence and config format

`localStorage` keys:

| Key | Contents |
|---|---|
| `skadis-variant` | `mini` / `standard` / `xl` |
| `skadis-width`, `skadis-height` | board size in cells |
| `skadis-density` | density slider value (1–5) |
| `skadis-lang` | `en` / `ru` |
| `skadis-theme` | `light` or empty for dark |
| `skadis-excl` | JSON `{x:[], y:[], p:["x,y"]}` mount exclusions |
| `skadis-excl-sizes` | JSON array of `role|wCxhC` panel-size exclusions |
| `skadis-colors` | JSON `{palette:["#rrggbb"], selected, paint:{"role|x,y,wCxhC":"#rrggbb"}}` |

Exported config (`exportConfig`, version 1) contains: `variant`, `widthCells`,
`heightCells`, `density`, `mountExclusions {columns, rows, points}`,
`sizeExclusions`, and `colors {palette, selected, assignments}`. `importConfig`
parses a file, then `applyConfig` validates/clamps values and rebuilds the layout.
Keep both import and export in sync if the schema changes; bump `version` when
incompatible.

### Filament colour rules

- `PANEL_COLORS` maps `colorKey(m)` (`role|x,y,wCxhC`) to a `#rrggbb` string.
  Colours only apply to a module with exactly the same position, size and role,
  so a re-layout never leaks a colour onto the wrong panel.
- The palette is `FILAMENT_PALETTE`; the active paint colour is `PAINT_COLOR`.
  `ERASE_MODE` switches painting to deletion.
- Presets (`applyColorPreset`) must stay mirror-symmetric: use distances from the
  board centre (`Math.abs(cx - wC/2)`, `Math.abs(cy - hC/2)`), never raw `x`/`y`.
- Brackets (`role === 'mount'`) are intentionally left unpainted by presets.
- Removing a palette colour must also delete every assignment using it.
- Any new colour UI string needs EN + RU entries.

## UI / i18n / theme rules

- Every user-visible string must be added to **both** `I18N.en` and `I18N.ru`.
  Static labels use `data-i18n="key"`; dynamic text calls `t('key', ...args)`.
  `applyI18n()` writes `textContent`, not HTML.
- On first visit `detectLang()` picks RU when the primary browser locale is
  `ru*`, otherwise EN. An explicit RU/EN choice is stored in `skadis-lang` and
  always wins. Keep `document.documentElement.lang` in sync.
- Theme colors live in `THEME_COLORS.dark` / `THEME_COLORS.light`, accessed via `TC()`.
  Add every new color to both themes.
- Use existing classes (`.panel`, `.ptitle`, `.var-seg`, `.var-btn`, `.szchip`, `.cfgbtn`,
  etc.) and inline styles only for one-off layout, matching the surrounding code.

## How to test changes

### 1. Syntax check (fast)

```bash
node -e "const fs=require('fs');const h=fs.readFileSync('index.html','utf8');
const m=h.match(/<script>([\s\S]*)<\/script>/);fs.writeFileSync('/tmp/check.js',m[1]);"
node --check /tmp/check.js
```

### 2. Browser smoke test

`file://` is blocked by the Playwright CLI, so serve the folder:

```bash
python3 -m http.server 8765
```

Then use `playwright-cli` (see the `playwright` skill) and evaluate invariants:

```js
// coverage + overlap
const occ = new Int32Array(G.wC * G.hC);
for (const m of G.lay.mods)
  for (let y = m.y; y < m.y + m.hC; y++)
    for (let x = m.x; x < m.x + m.wC; x++) occ[y * G.wC + x]++;
occ.every(v => v === 1);

// symmetry (excluding asymmetric mount exclusions)
const map = new Map(G.lay.mods.map(m => [m.x + ',' + m.y, m]));
G.lay.mods.every(m => {
  const o = map.get((G.wC - m.x - m.wC) + ',' + (G.hC - m.y - m.hC));
  return o && o.role === m.role && o.wC === m.wC && o.hC === m.hC;
});

// excluded sizes must not be produced
G.lay.mods.every(m =>
  m.role === 'mount' || m.role === 'corner' ||
  !EXCL_SIZES.has(sizeKey(m.role, m.wC, m.hC)));

// no connector inside a module
G.lay.conns.every(c =>
  !G.lay.mods.some(m =>
    c.x > m.x && c.x < m.x + m.wC && c.y > m.y && c.y < m.y + m.hC));
```

Also sweep variants (`mini`, `standard`, `xl`), several board sizes, all densities,
and a few exclusion sets. Test the export/import round-trip via
`exportConfig()`/`applyConfig()`. Check RU and EN and both themes.

### 3. Definition of done

- [ ] `node --check` passes and the browser console has no errors (favicon 404 is fine).
- [ ] No holes, overlaps or out-of-board modules.
- [ ] Excluded panel sizes never appear; print list/catalog counts match the layout.
- [ ] Layout is mirror-symmetric when mount exclusions are symmetric.
- [ ] Connectors/seams contain no floating ornaments.
- [ ] Config export/import round-trips all plan fields, including filament colours.
- [ ] Colour presets are mirror-symmetric; removing a palette colour clears its panels.
- [ ] RU and EN labels are present; both themes render.
- [ ] `README.md` updated for user-facing changes.
- [ ] Temporary artefacts (`.playwright-cli/`, screenshots, servers) cleaned up.

## Conventions

- Keep edits minimal and consistent with the existing vanilla-JS style.
- Prefer helper functions over inline logic inside render functions.
- Do not commit `.playwright-cli/` or local test files; `.gitignore` only covers
  `models/`, `CLAUDE.md` and `.claude/`.
- Do not commit unless the user asks.
