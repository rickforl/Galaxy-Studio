# Grid Exhibit — Design Intent

R&D lab for the galaxy's **coordinate / grid system**: the reference frame that
positions, sectors, and map-anchored effects will read from. Captures *intent and
decisions*, not implementation detail (read `grid_exhibit.html` for the running
model). Status tags: **[BUILT]** in the exhibit · **[DECIDED]** agreed, not yet
canonical · **[TBD]** open.

**Anchor:** `grid_v0.1_20260718.06` — square/polar/hex grids · cell + line labelling ·
per-axis invert · line numbering + subdivisions · label placement (anchor / offset /
polar 0° orientation / spoke-line vs midpoint) · radial labels (letters / degrees /
magnetic bearings).

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

## Label placement **[BUILT]**

Where a label sits is its own axis of control, separate from what it says:

- **Cell anchor** — a 9-position grid (centre · edges · corners) for square
  sector/index labels, with edge padding so text doesn't collide with grid lines. Lets
  a coordinate read as a centred cell name or as a corner tick.
- **Offset X / Y** — a universal pixel nudge folded into the one `label()` helper, so it
  shifts *every* label (square, polar, line numbers, world coords) together. The literal
  "nudge everything a few px" knob.
- **Polar 0°** — which compass direction the 0° spoke (and sector A) points: East /
  North / West / South, composing with the fine `Angle offset`. This is the *frame
  orientation* — the degree/bearing readouts follow it, so `0°` / `000` always lands
  where 0° points.
- **Polar labels: on spoke line vs wedge midpoint** — label the spoke itself (a
  boundary) or the centre of the wedge between spokes (the actual sector). Sector letters
  read more naturally at midpoints; bearings read more naturally on the line.

Rationale: the future coordinate system has to make deliberate choices about label
convention (which corner, which direction is "up", spoke vs sector). Keeping placement
independent from content lets us audition every combination before committing.

## Radial labels — letters vs bearings **[BUILT]**

The polar spokes can be labelled three ways (`Radial labels`), because a radial frame
answers to two different mental models:

| Mode | Reads as | For |
|---|---|---|
| **Letters (A,B,C)** | sector names around the wheel | naming / addressing |
| **Degrees** | `045°` — angle *from the 0° direction* (relative) | geometry readout |
| **Magnetic radials** | `045` — 3-digit compass bearing, CW from North (absolute) | navigation / "fly the 270 radial" |

**Magnetic** is the navigation convention: bearing = `(screen-angle + 90°) mod 360`, so
North = `000`, East = `090`, South = `180`, West = `270` — independent of where the 0°
spoke is pointed (it reads the spoke's *actual* direction, not its index). The two degree
modes are kept visually distinct on purpose: `045°` (relative, suffixed) vs `045`
(absolute, 3-digit). The **Radar** preset ships as magnetic + North-up so it behaves like
a real compass scope.

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
