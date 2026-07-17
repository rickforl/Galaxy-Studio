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
10. **Lanes** — neighbors/system, official/unofficial ranges, beam width/opacity/colors, lane gap + lane rendering (culled).
11. **Traffic** — civilian ship count/speed/size/twinkle, smugglers, **ship-count LOD-by-zoom** + traffic rendering (culled). Perf levers live here.
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
- [ ] Phase 2+ — backdrop, lanes, traffic, economy *(next: pick one)*
- Refinements queued: real IE generator (uniform placeholder now); baked-mode animation (rotate the sprite) + optional LOD; qColor quantization tune (37 sprites @ /12).
