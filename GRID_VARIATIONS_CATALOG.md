# Grid Exhibit — Variations Catalog (mined from the EPIC archive)

Companion to `GRID_EXHIBIT_DESIGN_INTENT.md`. This is a **shopping list**: every grid /
coordinate-system idea found across the ~60 grid programs in the EPIC tree, filtered to
the ones worth adding to `grid_exhibit.html` as **presets** or **settings**. Each entry
says what it adds, how it maps onto the exhibit's existing structure (`G` config +
`SCHEMA` + `PRESETS`), and the **source file(s)** to open when you build it.

Source paths are relative to the **EPIC root** (`…/GAMES FOLDERS/EPIC/`). The archive is
mostly AI-chat-saved iterations, so most sources have many near-identical versions — the
one cited is the best representative.

**Tags:** `[BUILT]` already in the exhibit · `[PRESET]` a config combo, no new code ·
`[SETTING]` a small addition to `G`/`SCHEMA` · `[MODE]` a new render family / bigger
engine work · `[FIT?]` useful but may belong in `galaxy_studio`, not here.

---

## 0. What the exhibit already has (don't re-add)

So the list below stays honest about what's *new*:

- Families: **square / polar / hex / isometric** `[BUILT]`  ← iso added since first draft
- Line style: solid / dashed / dotted / dash-dot, weight, opacity, background `[BUILT]`
- Square tiers: axis / major / minor / auto-hiding **subdivisions** `[BUILT]`
- Boundary: circle **or** square, radius, edge-padding, grid-inset, clip, origin marker `[BUILT]`
- Labels: **world coords / A1 sectors / (col,row) index / line-numbers**, per-axis invert,
  9-position cell anchor, universal offset `[BUILT]`
- Polar labels: **letters / relative degrees / magnetic bearings**, 0°-orientation,
  spoke-line vs wedge-midpoint `[BUILT]`
- Hex labels: **axial `(q,r)`** (center-origin, pointy- & flat-top, boundary-clipped) `[BUILT]`  ← added since first draft
- Camera: single `w2s` world→screen, pan + wheel-zoom, recenter `[BUILT]`
- Context: seeded placeholder systems (count / seed) for scale reference `[BUILT]`
- Presets: Blueprint · Radar · Galactic sectors · Hex sectors · Minimal · **Hex axial (q,r)** ·
  **Graph paper** · **Tactical** · **Deep-space** · **Nav scope** · **Galactic longitude** ·
  **Isometric floor** `[BUILT]`

---

## 1. New render families / modes `[MODE]`

The big-ticket variations — a genuinely different frame, not just a knob. Ordered by how
often the theme recurs in the archive (≈ how proven it is).

| Mode | What it adds | Best source to copy from | Notes |
|---|---|---|---|
| ✅ **BUILT** — **Isometric / 2.5D floor** | 30° `iso2s` projection (yaw + pitch) over the same camera; square lattice as an iso floor, boundary→ellipse, stars on the floor. Ships as the *Isometric floor* preset. | `AI DISCUSSIONS/BATTLE_SIM/canvas_00_BSI_v202512_13_04.txt` (full iso engine, XYZ labels); `AI DISCUSSIONS/Ships_Display/fleet_202512_18_0349_dualColors_01.txt` (`toIso` + `drawGrid()` iso floor, 13 iterations); `STABLE_PROGRAMS/canvas_Battle_Sim_w_Fleet_Gen_01.txt` | Needs a new projection alongside `w2s` and a camera-rotation control. A 4th `type`. |
| **Spherical / globe (lat-lon)** | Latitude/longitude gridlines on a sphere — reads as a **galactic-longitude** frame, the natural "distance + bearing on a curved sky." | `AI DISCUSSIONS/PITRI_DISH_V251200/Pitri_003_Current_Stats_01.txt` (WEBGL globe, 15° gridlines, sin/cos projection, live lat/lon readout) | WEBGL; heaviest lift. Ties directly to the polar frame's "galactic longitude" reading in the design doc. |
| **Filled-cell / tile grid** | Instead of *lines between* cells, color the **cells themselves** (biome / heat / ownership). Turns the grid into a field renderer. | `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/Faction_Field_03.html` (`idx(x,y)=y*GRID+x`, per-cell `fillRect`, territory); `EBB_TBC_V4/.../DESIGN_MAP/field_heatmaps_01.html` (`computeGrid()` scalar field + gridline overlay toggle); `STABLE_PROGRAMS/can_Layout_Debugger_Themes_01.txt` (biome tiles) | Could be a `[SETTING]` "cell fill: none / heat / checker" over the square grid rather than a full mode. Pairs with a scalar-field source. |
| **Perspective / horizon grid** | A TRON-style floor receding to a vanishing point (a "front view" as opposed to top-down). | `01_26_JAN/JAN/MISC/battlemap_01.txt` (`drawFrontViewGrid()` / `drawGroundViewGrid()`); `01_DEC/LLM/NOT_GENRE_NEUTRAL/old_battle_sim_01.txt` | Mostly a *view* variant of square; lower priority than iso. |
| **Fractal / zoom-adaptive LOD grid** | Grid lines **appear/disappear by tier as you zoom**, so density stays legible across all zoom levels (true level-of-detail, beyond the current subdivision auto-hide). | `EBB_TBC_V3/reference_code_01.txt` ("fractal square grid, major/minor"); `EBB_TBC_V4/BR_BRAINSTORMING/20260521_ORDERS/ebb_tbc_v4_canvas_battlegrid_01_minimap_01.html` ("Fractal Grid Drawing Logic"); `03_2026/epic_basic_V2 Digital Guide/canvas_ideal_game_look_013_02.txt` (the most complete tactical engine, 30 iterations) | Half-built already (subdivisions auto-hide). Upgrading to full LOD is a `[SETTING]` on the square renderer. |

**Recommended order:** Isometric first (most reused, biggest visual payoff), then
filled-cell (cheap, high utility for a *galaxy* map), then the rest.

---

## 2. New settings that drop into the existing schema `[SETTING]`

Small additions — a new key on `G`, one row in `SCHEMA`, a few lines in a renderer.

| Setting | What it does | Source to copy from |
|---|---|---|
| ✅ **BUILT** — **Hex axial (q,r) labels** | Center-origin axial `(q,r)` labels for pointy- & flat-top, boundary-clipped, zoom-gated. Ships as the *Hex axial (q,r)* preset. (Reconstructed the axial math; True Zen's `Func_ZenGridDraw.js` was never found.) | `EBB_TBC_V4/AR_ARCHIVE/pres_truezencanvas_jsx_01.txt` (True Zen: hex/axial `(q,r)`, `showGridCoordinates` toggle) — **note:** its core math file `Func_ZenGridDraw.js` was **not found anywhere in the EPIC tree**; you may need to reconstruct the axial math. |
| **Live cursor readout** | Show world `(x,y)` + current **sector** under the mouse in the top bar (today it shows only fps/zoom). Requires a `screenToWorld` inverse of `w2s`. | `03_2026/canvas_Battle_Map_02.txt` (`logicalToScreen`/`screenToLogical`, cursor coord readout); `AI DISCUSSIONS/Ships_Display/Miner 2049er/Mini_Type_04_Galaxy_004.txt` (`Sector: x, y` readout) |
| **Grid glow / neon lines** | `drawingContext.shadowBlur` on the grid for a hologram / scope look — a whole visual family in the archive's HUDs. | `AI DISCUSSIONS/TECH_STACK/2026/Legacy_Code/CMC_Part01_01.txt`; radar-style `AI DISCUSSIONS/Ships_Display/Miner 2049er/Mini_Type_06_Bio_Arc_001.txt` |
| **Animated grid drift / scroll** | Slow parallax scroll of the grid synced to motion (screen-space), for a "moving through space" feel. A toggle + speed. | `AI DISCUSSIONS/TECH_STACK/2026/Legacy_Code/CMC_Part01_01.txt` (`drawStars()` scrolling grid) |
| **Range-ring overlay** | Labeled concentric **distance bands** (e.g. every N units) drawn *over any* family — a radar range scale independent of the polar rings. | `EBB_TBC_V4/BR_BRAINSTORMING/3v3_SIM_canvas_20260429_04_range_rings.txt`; `Mini_Type_06_Bio_Arc_001.txt` (concentric range rings) |
| **Grid-snapped context systems** | Toggle placeholder systems between the current random scatter and **snapped to grid intersections** (the `USE_GRID_LAYOUT` lattice look). | `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/galaxy_01.txt` (`USE_GRID_LAYOUT`, `GRID_DIMENSION`, `GRID_SPACING`, `GRID_LINE_INTERVAL`); `psj5 Files/canvas_Aspiring_Quasar_01.txt` (`gridPoints[]`) |
| **Bracket / reticle frame** | Corner-bracket HUD frame around the boundary instead of a plain circle/square outline. | `PYTHON_ONE/Archive/EPIC/251121/AIPATH_3D_02/aipath_3d_02.py` ("bracket-frame HUD") |
| **Edge fade / vignette** | Radial fade tied to `radius` — the design doc's "Map-track fade" stand-in, auditioned here. | `galaxy_studio.html` (`bd_grid` + map-anchored radial fade) — same folder |
| **Minimap inset** | A small `LocalMinimap` radar inset showing the whole boundary + a viewport rectangle. | `EBB_TBC_V4/BR_BRAINSTORMING/20260521_ORDERS/ebbtbc_canvas_20260521_01_orders_02.html` (`LocalMinimap`, polar→minimap coords) `[FIT?]` |
| **Resolution / DPI scaler** | A clean `toScreen`/`toLogical` scale-offset helper if you want crisp coords across pixel densities. | `AI DISCUSSIONS/GALAXY_v2.0/CLASSES/class_ResolutionScaler_01.txt` (`ResolutionScaler`) |

---

## 3. New coordinate / labelling conventions `[SETTING]`

Feeds the design doc's open **"sector naming scheme"** and **"sub-sector addressing"** questions.

- **Hex axial `(q,r)`** — see §2; the corpus's only real hex coord scheme (True Zen).
- **Live sector readout** — see §2; turns any frame into an addressable one at the cursor.
- **Nested / hierarchical sectors** — a coarse sector letter + a fine index inside it
  (`G7.2`-style), matching the design doc's TBD sub-sector question. Sources with a
  built 2-level sector array: `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/DESIGN_MAP/galaxy_resource_table_01.html`
  (100 sectors × resources, `g[r][c]`); `AI DISCUSSIONS/GALAXY_v2.0/GEN_NODES/GEN_NODES_v202512_09_1234.txt`.
- **Wormhole / route links between cells** — connective lines between grid nodes; more a
  *studio* feature than a grid one. `PYTHON_ONE/Archive/EPIC/251121/SYSGEN_01/sysgen.py`;
  `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/Inter_Econ_Sim_01.txt` `[FIT?]`.

---

## 4. Ready-to-add presets `[PRESET]`

Config combos achievable with the **current** code — drop straight into the `PRESETS`
object in `grid_exhibit.html`. Each evokes a recurring archive theme. (Keys verified
against the live `G` config.)

```js
// --- add to PRESETS in grid_exhibit.html ---

graphpaper: {name:'Graph paper (square)', ov:{
  type:'square', color:[90,130,170], opacity:0.5, weight:1, lineStyle:'solid',
  sqSpacing:60, sqMajor:5, sqSubdiv:1, sqLabelMode:'line',
  lineNumMajor:true, lineNumMinor:true, labels:true,
  showBoundary:false, showSystems:false }},
// evokes: canvas_ideal_game_look (03_2026), reference_code_01 (EBB_TBC_V3)

tactical: {name:'Tactical (world coords)', ov:{
  type:'square', color:[80,180,140], opacity:0.4, weight:1,
  sqSpacing:120, sqMajor:5, sqLabelMode:'world', labels:true,
  showBoundary:true, boundaryShape:'square', boundaryColor:[120,255,200] }},
// evokes: 3v3_SIM battle grid, canvas_Battle_Map (03_2026)

deepspace: {name:'Deep-space (dotted, faint)', ov:{
  type:'square', color:[110,130,170], opacity:0.3, lineStyle:'dotted', dashScale:1.4,
  sqSpacing:100, sqMajor:5, labels:false, showBoundary:false, showOrigin:false }},

navscope: {name:'Nav scope (compass, dense)', ov:{
  type:'polar', color:[60,210,160], opacity:0.45, ringSpacing:100, spokeCount:24,
  labels:true, radialMode:'magnetic', polarZero:'north', polarLabelAt:'line',
  boundaryColor:[110,255,200], showBoundary:true }},
// evokes: radar / Bridge Skeleton scope, Ships_Display battle_map

longitude: {name:'Galactic longitude (letters, wedges)', ov:{
  type:'polar', color:[130,150,240], opacity:0.4, ringSpacing:140, spokeCount:12,
  labels:true, radialMode:'letters', polarLabelAt:'mid', angleOffset:15,
  boundaryColor:[150,170,255] }},
```

Presets that need a §1/§2 feature first (listed for when it lands): **Isometric floor**,
**Hologram (glow)**, **Globe / galactic-longitude (spherical)**, **Heatmap field**,
**Star lattice** (grid-snapped systems + faint grid, the `galaxy_01` look).

---

## 5. Source quick-reference — "if you build X, open Y"

The single best implementation in the archive per feature, so you're not re-searching:

| Feature | Open this |
|---|---|
| Full tactical square engine (LOD, labels, zoom) | `03_2026/epic_basic_V2 Digital Guide/canvas_ideal_game_look_013_02.txt` |
| Isometric `project`/`toIso` + camera rotate | `AI DISCUSSIONS/Ships_Display/fleet_202512_18_0349_dualColors_01.txt` |
| Iso battle engine w/ XYZ coord labels | `AI DISCUSSIONS/BATTLE_SIM/canvas_00_BSI_v202512_13_04.txt` |
| Hex axial `(q,r)` coords | `EBB_TBC_V4/AR_ARCHIVE/pres_truezencanvas_jsx_01.txt` (core math file missing) |
| Spherical lat/lon globe grid | `AI DISCUSSIONS/PITRI_DISH_V251200/Pitri_003_Current_Stats_01.txt` |
| Filled-cell / heat field | `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/DESIGN_MAP/field_heatmaps_01.html` |
| Cellular territory grid (`idx(x,y)`) | `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/Faction_Field_03.html` |
| `screenToWorld` + cursor readout | `03_2026/canvas_Battle_Map_02.txt` |
| Grid-snapped star lattice | `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/galaxy_01.txt` |
| Perspective / front-view grid | `01_26_JAN/JAN/MISC/battlemap_01.txt` |
| Animated / glow HUD grid | `AI DISCUSSIONS/TECH_STACK/2026/Legacy_Code/CMC_Part01_01.txt` |
| Minimap / radar inset | `EBB_TBC_V4/BR_BRAINSTORMING/20260521_ORDERS/ebbtbc_canvas_20260521_01_orders_02.html` |
| Resolution/DPI coord scaler | `AI DISCUSSIONS/GALAXY_v2.0/CLASSES/class_ResolutionScaler_01.txt` |

---

*Generated from a full sweep of the EPIC tree (~1,600 doc files, excl. node_modules/.git).
Full raw inventory of all ~60 grid programs and their version-groups is available on
request if you want the exhaustive file list rather than this curated shortlist.*
