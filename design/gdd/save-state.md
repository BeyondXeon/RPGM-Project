# Save-State & Persistence

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: The City Remembers · Short, Dense, Finished

## Overview

Save-State & Persistence is the data substrate of CHROME HEART CITY: a versioned `Game_System` schema holding every cross-session fact the reactive city needs — district unlock flags, turf states, heat/rep values, gig completion and secret discovery flags, crew roster, implant tiers, faction standing. All custom fields initialize through aliased defaults (stale saves migrate forward, future saves refuse — old progress never breaks), serialize through `JsonEx`-safe plain data only, and stamp a save version on every write. If this system vanished, the city would forget everything between sessions — no memory, no legend.

## Player Fantasy

None — pure infrastructure. The player never meets this system; they meet what it enables: a city that remembers their gigs, their heat, their allegiances across every session. Its success metric is invisibility — no lost progress, no broken saves, no "where was I?" Every design choice below serves that invisibility.

## Detailed Design

### Core Rules

1. All custom state lives in a single `Game_System.ch` namespace with this canonical schema (types fixed — implementers must not invent shapes):

| Field | Type | Content |
|---|---|---|
| `version` | int | Schema version, stamped every write |
| `districts` | {id: {unlocked: bool, turf: enum, tags: {wardens: int, chrome: int, ghosts: int}}} | Per-district state; turf ∈ contested/owned_wardens/owned_chrome/owned_ghosts/resolved |
| `heat` | int 0–100 | Live pressure value |
| `rep` | int (lifetime points, never decays) | Rank derived via heat-rep-turf thresholds — never stored |
| `gigs` | {id: {status: enum, approach: enum\|null, retries: int}} | status ∈ available/active/resolved/failed; approach ∈ ghost/loud/mixed/messy/null |
| `secrets` | string[] | Discovery flags as `"district:key"` (e.g., `"sump:cache_02"`); hidden-gig unlocks live in `gigs`, never here |
| `crew` | int[] | Recruited actor IDs |
| `implants` | {slotId: tier} | Minimal shape; full spec owned by Chrome Implants GDD |
| `faction` | {wardens: int, chrome: int, ghosts: int} | Cumulative standing inputs |
| `flags` | {key: bool\|int\|string} | Generic narrative storage: beat history, choice records, relationship flags, NPC page flags |
| `credits_home` | none (absent by design) | Credits live in `$gameParty` gold (explicit). No parallel currency field exists; assert its absence |
| `meta` | {hints_fired: string[]} | Onboarding hint triggers (intro-done lives OUTSIDE `ch` — see global below) |
| `ConfigManager.ch_introDone` | bool (global, survives New Game) | Single source of truth for intro completion. Written by the intro-complete event AND the skip path; read at the title/new-game branch before `setupNewGame`. Never duplicated into `ch` |

2. Aliased `Game_System.initialize` builds `ch` defaults at new game; loads preserve `ch`, missing fields migrate forward. Alert/suspicion states are never serialized (transient by contract — the Stealth mercy rule falls out of the schema).
3. `ch.version` is stamped on every write; load runs the tri-state check (see Formulas); stale turf falls back to the last persisted valid state (only keyless saves default to Contested) — per District Maps, the map must not forget.
4. Plain data only in `ch` — numbers, strings, arrays, objects. No functions or class instances (`JsonEx` constraint). Non-plain data at write time: dev builds assert/throw naming the key path; release builds refuse the write with an Alert Red message — never silently persist a mutilated `ch`.
5. Autosave writes on gig completion + district transfer via a deferred flag flushed at frame end (deduped — one write per frame max); manual save always available; autosave uses the MZ default autosave slot (exact savefileId pinned in prototype). Autosave requests during battle are queued to gig completion, never executed inline.
6. Switches/variables are transient-only by convention; anything persistent belongs in a named `ch` field. enforced by AC: event-command audit must show no persistent-switch use.
7. The system CANNOT: store functions · read plugin params from saves · write saves mid-battle (battle rewards flush at gig completion).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| New Game | `ch` built from defaults | → Playing on game start |
| Playing | Live session, checkpoints arm | ⇄ checkpoint states on gig completion / transfer / manual save |
| Save: Current | `version == CURRENT` | Loads directly |
| Save: Stale-migratable | `version < CURRENT` | → Current via upgrade functions on load |
| Save: Incompatible-future | `version > CURRENT` | Refused with a message; never partially loaded |

### Interactions with Other Systems

- **District Maps** — reads unlock/turf/secret flags; writes secret discovery.
- **Gig Board** — reads/writes gig status values.
- **Heat/Rep/Turf** — reads/writes heat and rep values.
- **Faction & Endings** — reads/writes faction standing.
- **Breach Combat** — flushes battle rewards via gig completion (no direct save writes).
- **Deck-OS UI** — reads everything for display; writes nothing.

## Formulas

The `load_action` tri-state is defined as:

`load_action = save.version < CURRENT ? "migrate" : save.version == CURRENT ? "load" : "refuse"`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Save version | save.version | int | 0+ | Stamped version in `Game_System.ch` |
| Current version | CURRENT_VERSION | int | 1+ | Bumped on every schema change |

**Output Range:** "migrate" → run upgrade functions `v(save.version+1)` through `v(CURRENT)` in order, then load; "load" → load directly; "refuse" → message + title screen, no `$gameSystem.ch` mutation, manual slots untouched.
**Example:** save v2, CURRENT v4 → run `upgrade_3`, then `upgrade_4`, then load. Save v5, CURRENT v4 → refused.

MZ hook points (aliased, never overwritten): `Game_System.initialize` (defaults) · `DataManager.makeSaveContents` (stamp version + serialize `ch`) · `DataManager.extractSaveContents` (tri-state check + upgrades inside try/catch → corrupt saves route to title) · `DataManager.saveGame` / `loadGame` (entry points) · autosave via deferred flag (rule 5).

No other mathematical formulas exist in this system — heat/rep/turf math belongs to their respective GDDs and is referenced, not defined, here.

## Edge Cases

- **If a future-version save is loaded**: refused with a message, never partially loaded.
- **If a save file is corrupt**: caught on load, message shown, return to title — no crash, no console error in release.
- **If an autosave write fails**: the session continues with a warning; manual save remains available.
- **If `ch` is missing entirely** (pre-schema save): build sparse defaults → set version to 0 (never stamp CURRENT first) → run `upgrade_1..CURRENT` → stamp. Upgrades expect sparse old data; stamping first would double-apply.
- **If a persisted value fails its type/enum** (unknown turf string, unknown district id): coerce to the keyless default for that field + dev console warning (content bug, caught in QA) — never refuse a loadable save over one bad value. (Distinct from stale-missing-key, which keeps last-valid: a corrupt value has no valid prior to keep.)
- **If a save is attempted mid-battle**: menu save is disabled in battle; battle rewards flush only at gig completion.
- **If gig completion and district transfer fire on the same frame**: the deferred autosave flag dedupes — one write carrying both, flushed at frame end after transfer resolves.
- **If non-plain data is found in `ch` at write time**: dev builds assert/throw naming the key path; release builds refuse the write with an Alert Red message (developer error — must be caught in playtest before release).

## Dependencies

**Upstream:** none hard — the MZ `DataManager`/`JsonEx` substrate is assumed. Respects District Maps' stale-turf rule (preserve last persisted valid state; Contested only when keyless) as the migration fallback.

**Downstream:**
- **District Maps & Exploration** (hard) — consumes unlock, turf, and secret flags.
- **Gig Board & Missions** (hard) — gig status schema owned here.
- **Heat/Rep/Turf Reactivity** (hard) — heat and rep value fields owned here.
- **Faction & Endings** (hard) — faction standing field owned here.
- **Breach Combat + Hack** (soft) — writes via gig completion only, never directly.
- **Deck-OS UI/HUD** (soft) — read-only display of all persisted state.
- **Onboarding** (hard) — hint triggers in `ch.meta`; intro-done in the global `ConfigManager.ch_introDone` (single source, never in `ch`).
- **Dialogue & Narrative Events** (hard) — beat history, choice records, relationship flags in `ch.flags`.

## Tuning Knobs

- **autosave_triggers** (default: gig completion + district transfer): adding every-map-transfer = safer but noisier and slower; removing all = manual-only hardcore, not this game.
- **manual_slots** (default 3 + 1 autosave): fewer = player anxiety; more = clutter.
- **CURRENT_VERSION**: bump on ANY schema change, no exceptions — a forgotten bump strands saves silently.
- **migration_window**: how many versions back upgrades support. Default: all the way to v0 — short game, cheap to support forever.

## Visual/Audio Requirements

- Autosave toast in deck-OS terminal style (120–180ms decrypt-in, per art bible UI motion).
- Save slots follow district-accent rules; active district accent marks the latest slot.
- Corrupt/future-refusal messages in Alert Red panels with icon + text (never hue alone, per colorblind rule).
- Audio: soft terminal-blip on autosave (mutable in settings, never played during combat).

## UI Requirements

- Deck-OS save screen: 3 manual slots + 1 autosave slot.
- Autosave indicator (city-eye dot) visible during writes.
- Migration notice line on stale load (e.g., "Migrated v2 → v4").
- Refusal dialog for future-version saves.

> **📌 UX Flag — Save-State**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover the save screen, autosave indicator, and migration/refusal dialogs **before** writing epics.

## Acceptance Criteria

- **GIVEN** a new game, **WHEN** started with dev console open, **THEN** zero errors/warnings through first map entry, `ch.version === CURRENT_VERSION`, every schema-table key present.
- **GIVEN** a gig completed, **WHEN** resolved, **THEN** autosave slot contains `gigs[gigId] = {status: resolved, approach: <approach>, retries: <n>}`, heat/rep equal pre-resolution values plus the gig payload, manual slots unchanged.
- **GIVEN** a district transfer, **WHEN** taken, **THEN** autosave contains `districts[id].unlocked === true` and player position in the new district.
- **GIVEN** a stale save, **WHEN** loaded, **THEN** upgrades run in order, every schema-table field round-trips (diff empty), migration notice reads `Migrated vX → vY`.
- **GIVEN** a future-version save, **WHEN** loaded, **THEN** named refusal message, no `ch` mutation, title scene active, manual slots untouched.
- **GIVEN** any save/load cycle, **WHEN** completed, **THEN** all schema-table fields (including crew, implants, faction, flags, meta) round-trip exactly.
- **GIVEN** the event-command audit, **WHEN** run, **THEN** no persistent state lives in switches/variables (transient-only rule enforced).

## Open Questions

- `JsonEx` payload size ceiling for a full endgame `ch` — owner: prototype measurement.
- Autosave slot index convention vs. MZ default — owner: implementation.
- Steam Cloud / itch persistence — explicitly out of MVP; owner: release phase.
