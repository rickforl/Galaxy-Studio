# Grid Exhibit — Design Intent

R&D lab for the galaxy's **coordinate / grid system**: the reference frame that
positions, sectors, and map-anchored effects will read from. Captures *intent and
decisions*, not implementation detail (read `grid_exhibit.html` for the running
model). Status tags: **[BUILT]** in the exhibit · **[DECIDED]** agreed, not yet
canonical · **[TBD]** open.

**Anchor:** `grid_v0.1_20260718.04` — square/polar/hex grids · cell + line labelling ·
per-axis invert · line numbering + subdivisions.

## Why this exists

`galaxy_studio` already has effects that want to be pinned to the *map* rather than the
*screen* — the fade / nebula / image **"Map track"** options. Today those anchor to
`gen.radius` (galaxy origin + radius) as a **stand-in**. This exhibit is where the real
coordinate system gets designed before it replaces that stand-in. The boundary circle
here **is** the galaxy edge (`Radius`); world (0,0) is the galaxy core.

Everything is drawn in **world space** and projected through one camera
(`w2s = center + (cam + world)·zoom`), so panning/zooming never changes the coordinates —
only the view. That is the whole point: a coordinate is a property of the galaxy, not of
where the user happens to be looking.

## Grid families **[BUILT]**

Three candidate frames, switchable live — we haven't committed to which becomes canonical:

- **Square (Cartesian)** — the workhorse. `x,y` lattice, the natural fit for A1-style
  sector references and the galaxy_01 heritage (fixed lattice, sector border).
- **Polar (rings + spokes)** — radius/bearing. Reads as a radar / galactic-longitude
  frame; sectors become ring × spoke wedges.
- **Hex tiles** — pointy- or flat-top. Equidistant neighbours; a tiling frame rather
  than a labelling one (no coord scheme wired yet).

## Line tiers (square) **[BUILT]**

Three weights, so structure is legible at a glance without reading numbers:

- **Axis** — the `x=0` / `y=0` lines through the core, heaviest.
- **Major** — every `Major every` cells, emphasised.
- **Minor** — the base `Spacing` lattice.
- **Subdivision** — `Subdivisions` (1–10) inserts finer lines *inside* each base cell,
  drawn faint and **only when they're legible** (auto-hidden once they'd turn to mud on
  zoom-out). These are visual density only — not part of the coordinate count.

## Labelling — cells vs lines **[BUILT]**

Two fundamentally different things can carry a label, and the exhibit keeps them
distinct because the future system must choose deliberately:

| Mode | Anchored to | Reads as |
|---|---|---|
| **Sectors (A1)** | cell centre | `colLetter + (row+1)` — galaxy_01-style sector names |
| **Cell index** | cell centre | `(col,row)` — raw integer cell address |
| **World coords** | major lines | world distance from core (signed) |
| **Line numbers** | grid lines | integer index *of the line itself* |

**Edge-anchored, not origin-anchored.** Sector/index/line labels count from the
galaxy's edge inward (`0` / `A` at the first interior line), not from the core outward.
Rationale: a coordinate system needs a **stable zero that doesn't move when the galaxy
grows** — the edge (Radius) is that reference, and it keeps sector `A1` in a consistent
corner. World-coords mode is the exception (it prints true signed distance) and exists
as the "physics" readout.

### Line numbering **[BUILT]**

Numbers the grid lines themselves (distinct from numbering the cells between them).
**Major and minor tiers toggle independently** (`Number major lines` / `Number minor
lines`) — number just the majors for a sparse reference frame, or both for graph-paper
density. Minor numbers auto-suppress when lines are too close to read. Placed along the
axes (X numbers on the x-axis, Y numbers on the y-axis) so they read like numbered
graph axes.

## Per-axis invert **[BUILT]**

`Invert cols (X)` / `Invert rows (Y)` reverse the counting direction of each axis
independently, for all edge-anchored modes (sectors, index, line numbers). Lets the
frame match whatever convention the final galaxy wants (Y-up vs Y-down, which corner is
`A1`) without touching world space.

## Context layer **[BUILT]**

Placeholder systems (seeded LCG, count/seed controls) scattered inside the boundary —
purely for **scale reference**, so grid density can be judged against a realistic star
field. Not the real generator.

## Open questions **[TBD]**

- **Which family is canonical?** Square is the front-runner for sector naming; polar may
  win for "distance from core" gameplay. Possibly both (square for addressing, polar for
  a secondary readout).
- **How `galaxy_studio` consumes it.** The end state: Map-track effects and system
  positions read from this coordinate system instead of the `gen.radius` stand-in. Needs
  a shared definition both files import.
- **Sector naming scheme.** A1 letters vs numeric `(col,row)` vs polar wedges — tied to
  the family decision above.
- **Sub-sector addressing.** If subdivisions become meaningful (not just visual), how a
  cell + subdivision compose into an address (e.g. `G7.2`).
