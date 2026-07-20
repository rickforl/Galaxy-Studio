# UID Scheme — Spec & Build Handoff

> **Status:** design **LOCKED** (2026-07-19). **Not yet implemented.** This is a self-contained
> handoff for a separate agent to build against — you should not need the originating chat.
>
> **What it is:** a hierarchical, collision-proof, near-unlimited unique-ID scheme for addressing
> *everything* in the EBB/TBC world — from a reality down to a single star system, and below as
> needed. **First consumer:** galaxy_studio **permanence** — stable system IDs that replace the
> current array-index references so systems can be recorded / hand-authored / edited without
> breaking lanes, traffic, or economy. Also the interface the **panels drop-in** and other tools
> mint against.

---

## 1. Goals
- **Collision-proof** — a "universe of IDs" (QR-code-scale variety) with a *hard* guarantee, not just improbability.
- **Self-describing & hierarchical** — the id *is* the path; each rung names its own kind.
- **Stable identity** — survives edits / moves; **never re-encodes position**.
- **Versioned & provenanced** — the top rung resolves to the version/ruleset that governs it.

## 2. The ladder (top → down)

| Rung | Prefix + body | Example | Resolves to |
|---|---|---|---|
| **Sequencer** | `E` + 2× base-36 | `E01` | a **version** record (registry) |
| **Creator** | `C` + type + 5× base-36 | `CT3K9F2` | a **creator/user** record (registry) |
| **Dimension** | `D` + `YYYYMMDD` + `.` + 2× base-36 | `D20260719.03` | — (self-contained) |
| **Universe** | `U` + base-36 counter | `U1` | — |
| **Galaxy** | `G` + base-36 counter | `G1` | — |
| **System** | `S` + base-36 counter | `S0A` | — |
| *(below, as needed)* | `B` body · `N` node · `L` lane · … | `B3` | — |

Creator **type** letters: `A` Agent · `U` User · `M` Machine · `T` Tool. The `C`-wrapper deliberately
keeps `U`=User from clashing with `U`=Universe.

## 3. Full address format
Dash-delimited path. The **dot is reserved** to the dimension's `date.sequence` (it appears nowhere else).

```
E01-CT3K9F2-D20260719.03-U1-G1-S0A
└┬┘ └──┬──┘ └─────┬────┘ └┬┘└┬┘└┬┘
 seq   creator    dimension U  G  system
```

= sequencer `E01` · creator `CT3K9F2` (a **T**ool, code `3K9F2`) · dimension born 2026-07-19, **3rd that day** · universe 1 · galaxy 1 · system `0A`.

- **Short form:** within an established context, drop the ancestors — inside galaxy `G1`, a system is just `S0A`.
- **Parsing:** split on `-`; each rung is `<prefix><body>`; the dimension rung carries an internal `.`; the creator's 2nd char is its type.

## 4. Rung-by-rung
- **Sequencer `E##`** — `E` + **2 base-36** (`E01` … `EZZ` ≈ 1,296 roots; widen the width when we outgrow it — "for starters"). **Registry-resolved to a version** (schema + engine range + ruleset). **Doubles as the "run rules" pointer** for embeds/players: a config's own top id says which version to render/run under.
- **Creator `C<type><#####>`** — `C` + type ∈ {A,U,M,T} + **5 base-36** (`C` `T` `3K9F2`). ≈ 36⁵ ≈ **60M per type**. **Registry-resolved to a creator/user record.** One-time registration.
- **Dimension `DYYYYMMDD.##`** — `D` + 8-digit creation date + `.` + **2 base-36 sequence that resets per day** (`.03` = "3rd dimension minted that day, by this creator"). The date carries the ordering. Self-contained (no registry pointer).
- **Universe / Galaxy / System `U/G/S<counter>`** — level letter + a **base-36 counter unique within its immediate parent**. Widths per-rung (see §14 — TBD, zero-padded once chosen).
- **Below system** — `B`/`N`/`L`/… same "prefix + per-parent base-36 counter" rule; extensible.

## 5. Base-36 & counter convention
- **Alphabet:** `0-9A-Z` (36 symbols), **UPPERCASE** canonical. (`toString(36).toUpperCase()`.)
- **Fixed width per rung, zero-padded** for sort/readability (`S00A`, not `SA`). Widths in §14.
- **Counters are monotonic, start at 1** (0 reserved — see §14), **never decremented**. A deleted entity's id is **retired** — the counter keeps climbing, so deletes leave gaps and ids are never reused.
- **Dimension date** is plain decimal `YYYYMMDD`; only the `.##` sequence is base-36.

## 6. Grammar / validation
Regex for a full system-level UID (uppercase base-36; widths per §14 — shown here as `{2}`/`{5}`/`+`):
```
^E[0-9A-Z]{2}-C[AUMT][0-9A-Z]{5}-D[0-9]{8}\.[0-9A-Z]{2}-U[0-9A-Z]+-G[0-9A-Z]+-S[0-9A-Z]+(-[BNL][0-9A-Z]+)*$
```
- Each rung self-identifies by its leading letter(s). Only the **creator** (`C`+type) and **dimension** (`D`+date`.`seq) deviate from the plain `<letter><base36>` shape.
- A **short-form** id is any suffix of the path from some rung down (validate against the parent context you're in).

## 7. Identity vs. address — the load-bearing rule
- The **UID is a stable identity.** Lanes / traffic / economy reference entities **by UID**, so they survive moves and deletes.
- **Position is a separate address** — the grid **sector** (`E3`, ring/bearing) and world / unit coordinates. These *change* when the entity moves.
- **Never let the UID encode position.** (Prior art `galaxy_translator` had a name-hash but still referenced by array index — the exact thing that breaks under editing. Fix: UID = identity, sector = location, orthogonal.)

## 8. Uniqueness — how the guarantee holds
Three layered guards → collision is *impossible*, not merely unlikely:
1. **Hierarchical counters** — each local id is unique within its parent, so the full path is unique. Below the creator, **no coordination is needed**.
2. **Creator namespace** — everything a creator mints lives under its tag, so independent creators can't clash. **Uncoordinated-safe.**
3. **Temporal anchor** — the dimension's `YYYYMMDD` stamps the branch in time (provenance + extra resistance).

**The only coordination cost is one-time registration** (mint a sequencer or creator tag once, into the registry). After that, every creator mints freely with its own **persistent, monotonic, per-namespace counters** — stored with the owning entity, never reused across sessions/tools.

## 9. Minting — rules & algorithm
```js
// ---- one-time, into the shared registry ----
registerSequencer(versionFields) -> pick next 'E##', write {type:'version', ...}, return 'E##'
registerCreator(kind, name, contact) -> pick next 'C<kind><#####>', write {type:'creator', ...}, return tag

// ---- when a galaxy is created (its dimension is minted) ----
dimSeq  = nextDailySeq(creator, today)                 // per creator, per day, base-36, 2-wide, resets daily
context = `${seq}-${creator}-D${today}.${dimSeq}-U${u}-G${g}`   // fixed prefix down to the galaxy
galaxy.counters = { S:1, B:1, N:1, L:1 }               // next-value per child namespace (persisted)

// ---- mint a child id within a namespace (monotonic, gaps on delete) ----
function mint(owner, prefix) {
  const n = owner.counters[prefix];
  owner.counters[prefix] = n + 1;                      // never decremented
  return prefix + n.toString(36).toUpperCase().padStart(WIDTH[prefix], '0');
}

const systemLocalId = mint(galaxy, 'S');               // e.g. 'S0A'
const systemUID     = `${context}-${systemLocalId}`;   // E01-CT3K9F2-D20260719.03-U1-G1-S0A
```

## 10. The unified registry
**One registry**, keyed by tag, with **typed records** (a single store doubles for versions *and* creators/users):
```json
{
  "E01":     { "type": "version", "schema": "1", "engineRange": "...", "ruleset": "...", "createdOn": "20260719", "desc": "..." },
  "CT3K9F2": { "type": "creator", "kind": "T", "name": "galaxy_studio", "contact": "...", "registeredOn": "20260719" }
}
```
Human-readable / PII data (names, emails) lives **in the registry**, keeping the ids themselves short and PII-free. Extensible to more record types later.

## 11. Data model (what the studio stores)
- **System record** (was `galaxy[] = {x,y,tier,color,seed}` — add a stable id, and it becomes explicit data):
  `{ id: 'S0A', x, y, tier, spectral/color, seed?, name? }`
- **References become ids, not indices:** lanes `{ a:'S0A', b:'S1B' }`; traffic pair-building and any economy references key on `id`.
- **Galaxy context + counters** (stored with the galaxy / in `CFG`):
  `context: 'E01-CT3K9F2-D20260719.03-U1-G1'` · `counters: { S: <next>, ... }`
- **Explicit systems** become part of the export (the "baked scene") — the same data the panels drop-in wants.

## 12. Studio build steps (the actual task — ordered)
1. **Reference by ID (keystone).** Give each system a stable local `id` (`S`-rung). Refactor **lanes** (`{a,b}` indices → ids), **traffic** pair-building, and any **economy** refs to key on `id`. Relink logic keys on `id`. *Nothing else works until this lands.*
2. **Explicit stored systems.** Promote the systems array from derived-each-load to stored data; the generator *writes* ids + positions once, edits *mutate* it; it's the source of truth. Include it in `studioSnapshot()` export.
3. **Context + counters.** Store the galaxy's fixed context prefix + per-namespace next-counters with the galaxy; persist so ids never reuse across sessions.
4. **Registry + registration.** Add the shared registry store; register the studio once as a **Tool** creator, and register/point the sequencer at a version. Resolve `E##`→version and `C…`→creator through it.
5. **Backward-compat.** `applySnapshot`'s default-merge already tolerates missing keys; on loading an old (recipe-only) export, bake a context + ids once (a one-time migration).

*Downstream consumers, not part of this build:* the **datum "1 X = 1 unit"** calibration, and the **record/author editor UI** — both just consume the stable ids.

## 13. Edge cases & rules
- **Deletes leave gaps** — counters never decrement; a retired id is never reused.
- **Width overflow** — if a counter exceeds its fixed width, widen the field (fixed width is for readability/sort; variable width is acceptable — implementer's call, see §14).
- **Case** — UPPERCASE base-36 is canonical; store uppercase (accept case-insensitive input if convenient).
- **Backward-compat** — old exports carry no systems/context; migrate on load (§12.5).

## 14. Open / TBD (not locked — confirm or defer during build)
- **Counter widths** for `U`/`G`/`S` (and below) — set by expected max counts, zero-padded. (E.g. `S` at 3 base-36 = ~46k systems.)
- **Counter start value** — 1 (recommended) vs 0; and whether `00`/value-0 is **reserved** (null / core / system).
- **Version record fields** — exactly what a "version" bundles (schema ver · engine range · ruleset · …).
- **Sector-address format** — whether a standardized positional address ships alongside the UID (it's the separate "address" from §7).
- **Below-system rungs** (`B`/`N`/`L`/…) — define concretely when first needed.

## 15. Related
- **Galaxy permanence** — first consumer; the §12.1 reference-by-id refactor is the prerequisite for record/author of explicit systems.
- **Panels drop-in port** (owned by another agent) — mints against this scheme; the sequencer's version pointer ⇄ the embed's "run rules".
- **Datum / unit** ("1 X = 1 unit") — orthogonal to identity: UID = who/which, datum + sector = where + how far.
