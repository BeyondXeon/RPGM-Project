# Save-State & Persistence

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: The City Remembers · Short, Dense, Finished

## Overview

Save-State & Persistence is the data substrate of CHROME HEART CITY: a versioned `Game_System` schema holding every cross-session fact the reactive city needs — district unlock flags, turf states, heat/rep values, gig completion and secret discovery flags, crew roster, implant tiers, faction standing. All custom fields initialize through aliased defaults (old saves migrate forward, never break), serialize through `JsonEx`-safe plain data only, and stamp a save version on every write. If this system vanished, the city would forget everything between sessions — no memory, no legend.

## Player Fantasy

None — pure infrastructure. The player never meets this system; they meet what it enables: a city that remembers their gigs, their heat, their allegiances across every session. Its success metric is invisibility — no lost progress, no broken saves, no "where was I?" Every design choice below serves that invisibility.

## Detailed Design

### Core Rules

1. All custom state lives in a single `Game_System.ch` namespace: `{version, districts:{id:{unlocked, turf}}, heat, rep, gigs:{id:status}, secrets:[], crew:[], implants:{}, faction:{}}`.
2. Aliased `Game_System.initialize` builds `ch` defaults at new game; loads preserve `ch`, missing fields migrate forward.
3. `ch.version` is stamped on every write; on load, `version < CURRENT` runs sequential per-version upgrade functions; stale turf defaults to Contested.
4. Plain data only in `ch` — numbers, strings, arrays, objects. No functions or class instances (`JsonEx` constraint).
5. Autosave writes on gig completion + district transfer; manual save is always available; the autosave slot is separate from manual slots.
6. Switches/variables are transient-only; persistent facts live in named `ch` fields (grepable, documented).
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

The `needs_migration` decision is defined as:

`needs_migration = (save.version < CURRENT_VERSION)`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Save version | save.version | int | 0–CURRENT | Stamped version in `Game_System.ch` |
| Current version | CURRENT_VERSION | int | 1+ | Bumped on every schema change |

**Output Range:** boolean. `true` → run upgrade functions `v(save.version+1)` through `v(CURRENT)` in order, then load; `false` → load directly.
**Example:** save v2, CURRENT v4 → run `upgrade_3`, then `upgrade_4`, then load.

No other mathematical formulas exist in this system — heat/rep/turf math belongs to their respective GDDs and is referenced, not defined, here.

## Edge Cases

- **If a future-version save is loaded**: refused with a message, never partially loaded.
- **If a save file is corrupt**: caught on load, message shown, return to title — no crash, no console error in release.
- **If an autosave write fails**: the session continues with a warning; manual save remains available.
- **If `ch` is missing entirely** (pre-schema save): build full defaults, stamp the version, run migration as v0.
- **If a save is attempted mid-battle**: menu save is disabled in battle; battle rewards flush only at gig completion.
- **If gig completion and district transfer fire on the same frame**: gig completion resolves first, then transfer — one autosave write carrying both.
- **If non-plain data is found in `ch` at write time**: stripped with a console warning (developer error — must be caught in playtest before release).

## Dependencies

**Upstream:** none hard — the MZ `DataManager`/`JsonEx` substrate is assumed. Respects District Maps' stale-turf rule (Contested default) as the migration fallback.

**Downstream:**
- **District Maps & Exploration** (hard) — consumes unlock, turf, and secret flags.
- **Gig Board & Missions** (hard) — gig status schema owned here.
- **Heat/Rep/Turf Reactivity** (hard) — heat and rep value fields owned here.
- **Faction & Endings** (hard) — faction standing field owned here.
- **Breach Combat + Hack** (soft) — writes via gig completion only, never directly.
- **Deck-OS UI/HUD** (soft) — read-only display of all persisted state.

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

- **GIVEN** a new game, **WHEN** started, **THEN** `ch` exists with defaults, version stamped, no console errors.
- **GIVEN** a gig completed, **WHEN** resolved, **THEN** autosave writes with updated gig status + heat/rep.
- **GIVEN** a district transfer, **WHEN** taken, **THEN** autosave carries the new unlock flag.
- **GIVEN** a stale save, **WHEN** loaded, **THEN** upgrade functions run in order and play continues with all state intact.
- **GIVEN** a future-version save, **WHEN** loaded, **THEN** refused with a message, title screen intact.
- **GIVEN** any save/load cycle, **WHEN** completed, **THEN** turf, secrets, crew, implants, and faction standing round-trip exactly.

## Open Questions

- `JsonEx` payload size ceiling for a full endgame `ch` — owner: prototype measurement.
- Autosave slot index convention vs. MZ default — owner: implementation.
- Steam Cloud / itch persistence — explicitly out of MVP; owner: release phase.
