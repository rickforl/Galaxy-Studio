# galaxy_studio — repo ownership & conventions

**Owned zone** (per CL_CLAUDE's `AGENT_ONBOARDING.md`). This repo — `EBB_TBC_V5/galaxy_sandbox`, deployed to GitHub Pages as **Galaxy-Studio** — is owned by the **`galaxy_studio` agent**. It is a single-file, buildless **p5.js** app: **`galaxy_studio.html` is the whole app**; everything else is docs/config. It consumes the WORLD space/time foundation by **porting** the pure modules to plain JS (`FIXPT` · `UNITS` · `COORD` · `DATUM` · `MAG`), each stamped with a `worldSpec` drift anchor — it does **not** vendor `board-world` (different toolchain: buildless p5 vs React/Vite).

## Sibling agents (WORLD, board-world, ZBE, …): propose, don't edit

- **Do NOT edit `galaxy_studio.html` directly.** Add or update a **`*_PROPOSAL.md`** in this folder (e.g. `BOARD_WORLD_UPSTREAM_PROPOSAL.md`) and let the galaxy_studio agent apply it. This happened once — a direct `DATUM` edit — traced, accepted, and fenced; please don't repeat it.
- **Ported WORLD modules are ports, not the source of truth.** Fix the canonical upstream (`…/BR_BRAINSTORMING/20260620_WORLD/modules/`) and note the commit; galaxy_studio re-ports on its own schedule (the `worldSpec` stamp tracks the gap to the current WORLD version).

## Commits

Owned zone → commit freely; no coordination needed. Sign commits in this repo with a trailer, for cross-agent provenance on the shared `Z_Gno` git identity:

```
Agent: galaxy_studio
```

(alongside the standard `Co-Authored-By`). The deployed site is served from `galaxy_studio.html` at the repo path; a push to `origin/main` redeploys GitHub Pages — treat pushes as outward-facing (they require the operator's go-ahead).
