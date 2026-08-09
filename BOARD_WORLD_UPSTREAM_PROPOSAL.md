# Proposal for the WORLD agent (owner of `board-world`) — camera-driven multi-family grid, a records/persistence layer, and an AxisLabels↔BoardGrid seam fix

**From:** `galaxy_studio` (`EBB_TBC_V5/galaxy_sandbox/galaxy_studio.html`) — a WORLD *consumer*, **not** a `board-world` vendor: a **buildless single-file p5.js** app that ports the pure WORLD modules to plain JS (FIXPT / UNITS / COORD, `worldSpec`-stamped) rather than bundling. So the below are **reference designs to adapt into board-world's React/TS**, not drop-in code.
**To:** the **WORLD foundation agent** — `board-world`'s owner since the 2026-08-05 move of its canonical source into the WORLD foundation folder (from `area_investing`).
**Date:** 2026-08-05
**Status:** proposal / handoff — not yet applied upstream.
**Companion:** `combat_range_sandbox/BOARD_WORLD_OVERLAY_PROPOSAL.md` (next in this sequence). The two **converge** — see *Relationship* below; read them together.

**Reference implementation** (all in the one file `galaxy_studio.html`): `DATUM` (`w2s`/`s2w`), `GRID` (`VIS`, `drawSquare|Hex|Polar|Iso`, bake, `dispY`/`genCoordStr`), `FIXPT`, `COORD` (`galaxyGroup`), `datasetExport`/`Import`, `studioSnapshot`/`applySnapshot`, `autoSave`/`DATASET_KEY`/`restoreDataset`, `GALAXY_PRESETS`/`cfgMatchesPreset`/`maybeUpgradeDefault`.

> board-world is *ahead of us* on the WORLD core (magnitude ladder, floating-origin, the whole TIME dimension, region/geometry/hex math, and the y-up coordinate-frame spec galaxy_studio already conforms to — same `[Y-BOUNDARY yUpWorld<->yDownScreen]` token). These three asks are only where **galaxy_studio is ahead**, and are generic enough to belong upstream.

## Relationship to the overlay proposal (two halves of one upgrade)

Both proposals converge on **making board-world's grid camera-driven.** Mapping the overlay's asks to ours:

| Overlay proposal (combat_range) | This proposal (galaxy_studio) | Together = |
|---|---|---|
| **#1 `useZoomPan` + #2 `screenToWorld`** — the *input/controller* half (viewport state, cursor-anchored zoom, pointer→world) | — | produce the `View` |
| **#3 "let `BoardGrid` own the `view`"** + adaptive minor grid — the *render* half | **#1** camera-driven multi-family grid (+ cull + bake) | **the camera `BoardGrid`** |
| — | **#2** records / persistence layer | *orthogonal* (no overlap) |
| — | **#3** AxisLabels seam fix | *dissolved* by the camera board |

Key point: **overlay #3 and this #1 are the same feature at different depths** — overlay adds the pan/zoom transform + adaptive-minor alpha; this adds families (hex/polar/iso) + visible-rect cull + bake. **Take them as one work item.** Overlay #1/#2 feed the `View` it consumes; and once a `BoardGrid` owns the `view` (both proposals want this), ask #3's seam mismatch simply goes away.

## Separation of concerns (against WORLD's camera boundary law)

Pre-classified against the tier-1/2/3 rule (camera **geometry** + **control** = pure foundation; **pointer/input/render-state** = consumer edge). Nothing here asks `modules/` to hold input or mutable state:

| Item | Tier | Lives in | Purity |
|---|---|---|---|
| Consume `View` + project (`clickToPixel`), visible-rect cull (`VIS`), family line-geometry (hex/polar/iso positions), readout **formatter** (world→string, range/bearing, frame) | **1 · geometry** | `space/units` (+ pure helpers) | pure |
| `fitBoardView` (fit-bounds) · cursor-anchored zoom (overlay #1) · pan/clamp | **2 · control** | proposed `space/camera` (View→View) | pure; overscan / zoom-limits / clamp-bounds = app **policy** params (datum→world→genre) |
| Bake buffers (cache/map) · pointer/hover/drag · readout **targets** (mouse/selection) · localStorage (`DATASET_KEY`) | **3 · input/state** | `board/` React + consumer adapter | impure — **never** `modules/` |

So the grid's **math** (project / cull / family-geometry / readout-format) and the **records model** are pure; **bake, pointers, and storage stay at the edge.** The tier-2 transforms in play — `fitBoardView` here + cursor-anchored zoom + clamp in the overlay proposal — are the concrete surface that would define a `space/camera` module, with feel (overscan, zoom limits, inertia) passed in as policy.

**Honest caveat on the reference impl:** galaxy_studio is a single-file p5 app that does **not** name this wall — `cam{x,y,zoom}` is a mutable global that IS the consumer-held `View` (origin + scale), and `mouseWheel`/`touchMoved` are tier-3 handlers in the same file. So when porting, extract the tier-1/2 **math** (`DATUM`, `VIS`, `fitBoardView`) and leave galaxy_studio's tier-3 handlers behind. `cam.x/cam.y/cam.zoom` is **not** proposed as a space primitive — it's the edge's `View`, spelled p5-style.

## Ask #1 — camera-driven, multi-family grid *(merge with overlay #3)*

**Today:** `BoardGrid` is square-only, static (no `View`, no camera), un-baked; numbers live in a separate `AxisLabels` overlay.

**galaxy_studio's `GRID`:**
- **Families** — square / hex (pointy+flat) / polar (rings+spokes) / iso, one projection. board-world has the hex math (`space/hex`) but renders none of these.
- **Camera + cull** — renders through `DATUM.w2s` (≈ `units.clickToPixel`) at live `cam{x,y,zoom}`, culling to the visible world-rect via `GRID.VIS()`. *Y-up trap:* `s2w` flips screen→world y, so the corner rect arrives reversed — **normalize to min/max** or `y0..y1` loops draw nothing (`[Y-BOUNDARY yDownScreen->yUpWorld, range]`).
- **Bake** — `cache` (re-bake on camera move) / `map` (pan-free re-blit, re-bake crisp on zoom).
- **Readout** — targets (mouse / waypoint / selected record) × fields (world / px / screen-% / grid-vs-gen datum / Y-up / relative range+bearing); complements `AxisLabels`' edge numbers with "what's under my cursor, in the frame I asked for."

**Shape upstream:** a `BoardGrid` that consumes a `View`, takes a `family` prop + optional `bake` mode + a `readout(view, config, targets)` helper — keeping board-world's render-prop hooks + pointer ergonomics (better than ours). This is the render side of the camera board that **overlay #1/#2 drive.**

## Ask #2 — records + persistence layer *(orthogonal to the overlay proposal)*

**Today:** `metrics.ts` is unbound and `fixedpoint` (de)serializes *one* position; there is **no records/Group layer** (the exhibit's `coordinateDb` never came into `world/`) and **no persistence/migration** — every consumer reinvents it.

**galaxy_studio** (top three are original, not in the exhibit or board-world):
1. **Records → relocatable Groups** (`COORD`, ported from `coordinateDb`): id + exact-BigInt canonical position + fields; Group = records + `anchorMode` + `version` + `meta`; `placeGroup` = exact BigInt add. `galaxyGroup()` exposes live objects as a Group. → **fold `coordinateDb` into `world/`.**
2. **Lossless datasets** (`datasetExport`/`Import`): the Group as JSON, `unit:'click'`, `worldSpec`-stamped (drift/migration anchor), exact round-trip.
3. **1:1 config snapshot** (`studioSnapshot`/`applySnapshot`), **kept separate from the record dataset** so the export contract stays record-free.
4. **Pristine-default migration** — the generic bit any versioned board app needs: authored instances persist to their **own** key (`DATASET_KEY`), never in the export contract; a **structural match** (`cfgMatchesPreset` over the preset's own keys, volatile fields ignored) flags a *pristine old default*; `maybeUpgradeDefault` upgrades it to current, **guarded** so it never touches a customized / user-picked / hand-authored instance; old defaults are housed as **versioned presets**. Net: a new default reaches returning users without clobbering real work.

**Shape upstream:** (a) bring `coordinateDb` into `world/`; (b) a pure `snapshotEquals(saved, preset, {ignoreKeys})` + pristine/authored/user-picked classifier + versioned-preset registry — only the storage adapter is app-specific.

## Ask #3 — the `AxisLabels`↔`BoardGrid` seam mismatch *(dissolved by ask #1)*

`AxisLabels`' header says mount it as a sibling of the transformed `BoardGrid` (same `View`) — but the shipped `BoardGrid` is **static and takes no `View`**. So two frames coexist: `board/geometry.sy()` (SW-corner, static px) vs `space/units.clickToPixel()` (centre-origin ENU camera). `AxisLabels` uses the latter, `BoardGrid` the former — they **don't compose**, violating the module's *own* `COORDINATE FRAME` rule (*"the y-flip belongs at EXACTLY ONE seam"*).

**Fix:** ship the camera `BoardGrid` of ask #1 (+ overlay #3) — both then ride one seam and `AxisLabels` genuinely pairs with it. Minimum, if the static board is intentional: correct `AxisLabels`' header (it pairs with an *app-provided* camera board) and name `geometry.sy` as a deliberate **separate SW-corner frame**, so "one seam" is scoped per-frame instead of silently contradicted.

## Sequencing

1. **Ask #3** — tiny correctness/doc fix; unblocks the rest.
2. **Ask #1 ⨝ overlay #3** — the camera `BoardGrid` (families + cull + bake, on the `View` overlay #1/#2 drive). Do both proposals' grid asks as one item.
3. **Ask #2** — largest surface; `coordinateDb` into `world/` first, then the pristine-default helper.

Reference code (plain JS, one file) available on request; porting direction is **JS reference → your React/TS** (galaxy_studio stays buildless and won't vendor `board-world` wholesale).
