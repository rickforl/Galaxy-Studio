# Galaxy Studio — Roadmap

> **Home:** `EBB_TBC_V5/galaxy_sandbox/` · started 2026-07-17
> **North star:** one app where **every part of the galaxy is a tunable option**, with a **"View Galaxy"** mode that renders the result live — a galaxy configurator/sandbox grown out of the sun options menu.

## Provenance
- **Seed program:** `sun_exhibit_03_options.html` — the unified sun build (every effect + live options menu + preset selector). Currently at `EBB_TBC_V4/BR_BRAINSTORMING/20260615_GALAXY/sun_exhibit/`.
- **Engine to reuse:** `galaxy_translator_01` (same folder) — working generation, lanes, traffic, economy, viewport culling, and the promoted sun renderer. Design/handoff: `galaxy_translator_design_intent_01.md`.

## Foundational decisions (settle early)
- **Config shape** — replace the flat `OPTS` with a namespaced `CFG`: `CFG.sun`, `CFG.gen`, `CFG.backdrop`, `CFG.lanes`, `CFG.traffic`, `CFG.economy`. One source of truth that every page writes and the view reads. Presets become whole-galaxy snapshots.
- **Engine reuse, not rebuild** — refactor `galaxy_translator`'s subsystems to read from `CFG` and drive them from the options UI. Most of the work is (a) make each subsystem config-driven, (b) expose its knobs as UI.
- **Endgame relationship** — likely: the studio **exports a `CFG` JSON that `galaxy_translator` loads at startup** (keeps the shipping galaxy stable while the studio evolves); merge once proven.

## Phase 0 — Shell (make the menu able to grow)
1. Namespace the config (`OPTS` → `CFG.sun`; add empty `CFG.gen`, `CFG.lanes`, …).
2. Multi-page options panel — top nav (Sun · Galaxy Gen · Backdrop · Lanes · Traffic · Economy) that swaps which schema section shows.
3. Two modes: **OPTIONS ↔ VIEW GALAXY** toggle.
4. Whole-config presets + export/import JSON.

## Phase 1 — Galaxy Gen page + basic star view  ← **START HERE**
5. **Galaxy Gen options page** — star count, galaxy radius, min spacing, **generator mode (galaxy_01 continuous vs. Interstellar-Economy)**, tier mix (T1/T2/T3 counts), spectral distribution (equal-6 + jitter, per-type weight sliders), random seed + reroll.
6. **Parameterized generator** — `generateGalaxy(CFG.gen)` → star systems (position, tier, spectral color); honors the generator-mode toggle.
7. **View Galaxy mode (stars only)** — render stars with the promoted sun renderer (`CFG.sun`), camera pan/zoom + viewport culling, Regenerate button, live re-gen on gen changes.
8. **Sun ↔ galaxy link** — tuning any sun slider updates every star in the view instantly (shared `CFG.sun`).

## Phase 2+ — Import each subsystem (options page + cull-aware render layer)
9. **Backdrop** — nebula on/off, gallery pick, hash, rotation/flip, bake resolution + baked backdrop.
10. **Lanes** — connection strategy + ranges; **three lane tiers (official / unofficial / transport-trunk), each independently colored and styled**; path shapes; lane rendering (culled). *Traffic is a separate later layer — lanes stay ship-free here.*
11. **Traffic** — rides the lanes from the Lanes tab; an **isolated Traffic view** with one **sub-tab per lane class** (+ add extra layers). Each ship is bound to a lane — galaxy_01 has no free-wandering traffic; the "random" feel comes from per-ship randomization. Ported model:
    - **Civilian (official lanes)** — dense two-way ships on the drawn beams; random per-route lane colors; per-frame twinkling ship size; ~3+ ships/route; fade over the end 15%.
    - **Unofficial (unofficial lanes)** — traffic on our unofficial lane class, kept **distinct from smugglers**; its own style / density / colors.
    - **Transport (trunk lanes)** — studio addition (hub arteries); no galaxy_01 equivalent.
    - **Smugglers (their own connections)** — a *separate*, **non-lane-bound** layer: it generates its **own** long-range links (`SMUGGLER` band `MIN_DIST 320 / MAX_DIST 640`, `LINKS_PER_SYSTEM 1`, deduped), **1 beamless ship/link**, random-colored, **slower** (`0.0012–0.0032`), size `random(2–5)` re-rolled per frame, fade at the end 15%. Ported galaxy_01 → galaxy_translator (`SmugglerRoute`/`SmugglerShip`); IE's red criminal routes were subsumed by these. **This is the "random traffic," distinct from the unofficial lanes** — the one traffic layer not bound to the lane network.
    - **+ extra layers** — more traffic layers, each either riding a lane class or (like smugglers) generating its own connections, with its own style/behavior.
    - Per-layer knobs: style (the 9 from `traffic_exhibit`), density, direction model (per-ship random phase/dir/speed/size = galaxy_01 organic flow, vs alternating/uniform), end-fade, color mode (multicolor/single/lane-match). Global: master, speed scale, **ship-count LOD-by-zoom** + culling (perf levers live here).
12. **Economy** — GALEX index rate, settlements dedup, ghost-ticker count/speed + exchange sidebar + chyron.

## Phase 3 — Make it real
13. **Perf HUD in view mode** — FPS pill + per-layer cost readout.
14. **Persistence** — named configs to `localStorage`; export/import JSON.
15. **Unify with the game** — per the endgame decision above.

Each Phase-2 subsystem repeats one move: refactor to read `CFG` → add schema page → add cull-aware render layer.

## Status
- [x] **Phase 0 — shell** (2026-07-17): `galaxy_studio.html` seeded; namespaced `CFG`, 6-page nav, OPTIONS↔VIEW GALAXY toggle, export/import CFG JSON. Galaxy view = teaser suns for now.
- [x] **Phase 1 — galaxy gen + basic star view** (2026-07-17, build .02): `CFG.gen` + Galaxy Gen page (stars, radius, min-spacing, generator toggle, star size, 6-weight spectral mix, jitter); `generateGalaxy()` (galaxy_01 min-dist + IE-uniform placeholder); VIEW GALAXY renders stars with per-star spectral color + camera pan/zoom + viewport culling; live regen + reroll. On branch `phase-1`.
- [x] **Baked galaxy renderer** (2026-07-17, build .04): per-spectral-color sun-sprite cache, toggle "Baked render" in Galaxy Gen; effect fns thread a target-graphics so the same code bakes offscreen. 1 blit/star → **300 systems @ 60 fps** (live mode capped ~80). Tradeoffs: static sprites (no per-frame anim), no zoom-LOD crossfade in baked mode.
- Perf saga (resolved): root cause was **Chrome GPU accel disabled** (environment, not code); then `pixelDensity(1)`; then baked sprites vs the per-star gradient storm.
- [x] **Phase 2 — Backdrop** (2026-07-17, builds .05–.24): composite subsystem — base fill, curated image layer (13-photo gallery + hash pick, rotate/flip/scale/opacity), procedural nebula (domain-warp/fBm/emission with a stepped, brightness-preserving cross-dissolve), grid overlay, aspect-aware radial fade; per-layer bake + per-layer parallax depth; isolated backdrop preview.
- [x] **Stars tab** (2026-07-17, builds .25–.27) *(added — not in the original roadmap)*: layered background starfield — Uniform/Halo/Field/Disc/Core/Bright layers, radial-disc density (disc overlap, no position skew), per-layer bake + parallax, zooms with the camera. Forked from `stars_exhibit.html`.
- [x] **Persistence + export** (2026-07-17, builds .28–.30, .35): full-config snapshot (adds camera framing + active preset to the CFG); localStorage autosave (debounced + on unload; `?reset` escape hatch); export filename via a toolbar field — replaced `prompt()`, which throws in the preview pane and had silently killed export.
- [x] **Phase 2 — Lanes** (2026-07-18, builds .31–.35): ported from `lanes_exhibit.html`. **Nodes = the actual galaxy systems** (world coords + tier), so lanes track whatever Galaxy Gen produces and relink on regen. Connection strategies: K-nearest (tier-scaled) / all-within-range / MST / Gabriel web, plus a long-range **unofficial** band. **Three independent lane tiers** — official, unofficial, and **transport (trunk arteries between hub systems, tier ≥ N)** — each an overlay that coexists on shared routes (additive blend), with its **own color mode** (multicolor / single / by-tier), color, and full style block (gradient/solid/glow/dashed/taper, two-way + opposite dash-flow, lane/node gap, end fade, curve, beamless). Path shapes: straight / fractal / river / road (midpoint displacement). 8 presets. Beams render screen-projected, drawn under the systems.
- **R&D exhibit forks** (standalone p5 labs — iterate a look before porting): `nebula_exhibit` (9 noise generators; domain-warp won), `stars_exhibit`, `lanes_exhibit`, `traffic_exhibit` (3×3 traffic-style lab; simplified lane selector + configurable combo cell → became the Traffic tab's styles), `lightning_exhibit` (fractal branching bolts; Tesla-coil / point-to-point layouts — may or may not ship to the galaxy).
- [x] **Phase 2 — Traffic** (2026-07-18, builds .37–.39): ported from `traffic_exhibit`. Isolated Traffic view + a layer list (render toggles + per-layer **Edit ▸** grid whose sample lane pulls that class's Lanes-tab look). Lane-bound layers (Official/Unofficial/Transport) drop ships on the real lanes; the **Smuggler** layer self-routes its own long-range connections (min/max-dist band, links/sys), beamless — distinct from the unofficial lanes. 7 ship styles, 3 direction models (random=galaxy_01 / two-way / single), 3 color modes, end-fade; **+ add layer** (retargetable/self-routing). View-bg selector (lanes/dim/full), **LOD-by-zoom**, per-layer clocks, snapshot-persisted (runtime caches stripped).
- [ ] **Phase 2+ — Economy** *(next; backdrop + stars + lanes + traffic done)*
- Refinements queued: real IE generator (uniform placeholder now); baked-mode animation (rotate the sprite) + optional LOD; qColor quantization tune (37 sprites @ /12).
