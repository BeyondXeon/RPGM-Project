# District Maps & Exploration

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: Short, Dense, Finished · The City Remembers · Ghost or Loud — Your Call

## Overview

District Maps & Exploration is the spatial substrate of CHROME HEART CITY: three dense hand-built districts (Sump → Market → Spire), each an MZ tilemap web of neon streets, an amber hub, and gig interiors stitched by gated transfers. Maps expose zone logic through region IDs and map metadata (district ID, turf state, stealth zones) that downstream systems — gigs, patrols, lighting — read as contracts. Player-facing, every district is a legible hunting ground: you learn its sightlines, its secrets, and its faction skin — and then turf shifts repaint it, so mastery never goes stale. No map exists without a gig, a secret, or a story beat.

## Player Fantasy

You own these streets. The fantasy is hard-won local mastery: the Sump's patrol rhythms in your bones, the Market's rooftop shortcuts under your fingers, the Spire's camera blind spots memorized — until a turf shift repaints the district and the hunter becomes a stranger again. Districts feel like territory gained, lost, and re-earned, never scenery. (Serves *The City Remembers* — the map itself is the scoreboard — and the street-legend core fantasy: legends know every alley.)

## Detailed Design

### Core Rules

1. Three districts unlock linearly (Sump → Market → Spire) via story-beat flags; transfers between districts are gated events, not open exits.
2. Each district contains: a street web (exploration + patrols), one amber hub (safe — no patrols, crew + fixers, shop/heal), and gig interiors (2–4 per district).
3. Zone logic is exposed through **region IDs** (provisional contract for implementation): R1–R9 patrol routes · R10–R19 restricted/alert zones · R20–R29 secret areas · R30+ reserved. Stealth reads these; exact numbering finalized in implementation.
4. Each map carries metadata: `districtId`, `turfState`, `lightingPreset` — read by gigs, patrols, and lighting as contracts.
5. Turf is shown by **overlay swaps on a single map**: poster variants, lighting preset shifts, patrol-density changes keyed to turf state. No duplicate maps.
6. Secrets: 3–5 per district — at least one hidden gig, one lore cache, one shortcut each.
7. The player CANNOT: leave a district except via its transfers · enter locked districts early · move turf by walking (only gigs and story beats move meters).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Locked | District unreachable, transfer sealed | → Active on story-beat flag |
| Active (Contested) | Playable, turf fluid | → Owned on faction threshold |
| Active (Owned:X) | Playable, overlays show faction X skin | → Owned:Y if turf flips |
| Resolved | Post-finale state, free roam, no new gigs | Terminal (endgame) |

### Interactions with Other Systems

- **Gig Board** (in: districtId + turfState to filter/frame gigs; out: completion flags that unlock transfers/secrets).
- **Stealth & Patrol** (in: region zones + patrol routes; out: alert states that can raise heat).
- **Lighting & Atmosphere** (in: lightingPreset + turfState for overlay selection).
- **Deck-OS UI** (in: district topology for the map screen; turf states for the city-eye display).
- **Save-State** (out: unlock flags, turf states, secret discovery — all persisted).

## Formulas

The `patrol_density` formula is defined as:

`patrol_density = base_patrols[district] + turf_modifier[turfState]`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| District base | base | int | 2–6 | Patrol units on the map at Contested baseline; Sump 3, Market 4, Spire 5 |
| Turf modifier | mod | int | −1 to +2 | Contested +0 · Owned (player-allied) −1 · Owned (hostile) +2 · Resolved −2 (min 1) |

**Output Range:** 1 to 8 patrol units per street map under normal play; clamped to [1, 8] at extremes (never zero — streets never feel dead; never above 8 — perf + readability cap).
**Example:** Market (base 4) under hostile ownership: 4 + 2 = 6 patrol units.

Turf overlay selection is a deterministic lookup, not a formula: `overlay_set = overlays[districtId][turfState]` — exactly one set per combination (3 districts × 4 states = 12 sets max, Pillar 4 cap).

## Edge Cases

- **If turf flips while the player is on the map**: overlays and patrol counts apply on map re-entry, never hot-swapped under the player. Rationale: mid-map repainting breaks stealth readability and risks event collisions.
- **If a transfer flag is set while the player stands on the exit tile**: the exit resolves normally; the new district state applies on arrival.
- **If a discovered secret is re-entered**: discovery flags prevent double rewards; lore caches show "already recovered" text.
- **If the density formula meets a tiny map**: interiors and hubs use gig-scripted patrols only and never the formula; street maps clamp output to [1, 8].
- **If a save with stale turf data is loaded**: turf defaults to Contested. Rationale: safe neutral state, no free faction advantage.
- **If the player re-enters maps to reset patrols for a better ghost rating** (degenerate strategy): the rating locks at gig completion; re-entry cannot improve it.

## Dependencies

**Upstream** (all undesigned — contracts provisional): no hard design-level dependencies; the MZ tilemap/eventing substrate is assumed.

**Downstream:**
- **Gig Board & Missions** (hard) — reads districtId/turfState/completion flags to filter and frame gigs.
- **Stealth & Patrol** (hard) — reads region zones + patrol routes; writes alert states.
- **Lighting & Atmosphere** (hard) — reads lightingPreset + turfState for overlay selection.
- **Deck-OS UI/HUD** (soft) — reads district topology for the map screen and turf states for the city-eye display.
- **Save-State & Persistence** (hard) — persists unlock flags, turf states, secret discovery.

## Tuning Knobs

- **base_patrols per district** (2–6; Sump 3, Market 4, Spire 5): too high = unreadable streets + event perf hit; too low = dead city.
- **turf_modifier per state** (−2 to +2): interacts with base — high base + hostile mod hits the 8-cap, flattening district differences. Retune base down before raising mods.
- **secrets_per_district** (3–5): more = explorer joy, but each secret needs a gig, lore, or shortcut payload per Pillar 4 — count without payload is cut content wearing a trench coat.
- **overlay_sets_cap** (12 max): more sets = art debt at one set per district × state; raise only with art-bible sign-off.
- **hub_safe** (boolean, default true): hubs are patrol-free by contract. Turning off breaks the amber-safety color promise — do not touch without an art-bible revision.

## Visual/Audio Requirements

- **Streets**: district accent carried on signage + light bars over the Abyss Blue base; verticals oppress, neon horizontals mark routes.
- **Infiltration**: ambient dims to ~40%; sightlines and hackables glow (light-is-information).
- **Turf flips**: poster variants + lighting preset shift + patrol-density change (12 overlay sets max across 3 districts × 4 states).
- **Hubs**: amber practicals, low contrast, zero patrols.
- **Audio**: per-district ambient beds (Sump: low industrial hum · Market: crowd-murmur synth · Spire: sterile high tone) + turf-shift sting + secret-discovery chime, all from the shared synth kit.

📌 **Asset Spec** — Visual/Audio requirements are defined. After the art bible is approved, run `/asset-spec system:district-maps` to produce per-asset visual descriptions, dimensions, and generation prompts from this section.

## UI Requirements

- District topology feeds the deck-OS map screen (streets, hubs, transfers, discovered secrets).
- Turf states feed the city-eye heat/rep/turf meters.

> **📌 UX Flag — District Maps**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to create a UX spec for the district map screen and the city-eye meters **before** writing epics. Stories that reference UI should cite `design/ux/[screen].md`, not the GDD directly.

## Acceptance Criteria

- **GIVEN** a locked district, **WHEN** the player uses its transfer, **THEN** entry is refused with fixer dialogue.
- **GIVEN** an unlocked district, **WHEN** entered, **THEN** streets, hub, and 2+ gig interiors load with no console errors.
- **GIVEN** Contested turf, **WHEN** a faction threshold is crossed, **THEN** overlays, posters, and patrol density change on next map entry.
- **GIVEN** any street map, **WHEN** patrols spawn, **THEN** count equals base+mod clamped to [1, 8].
- **GIVEN** a secret found, **WHEN** re-entered, **THEN** no duplicate reward is granted.
- **GIVEN** the 60fps budget, **WHEN** a street map runs at full patrol count, **THEN** frame time stays under 16.6ms on target PC hardware.
- **GIVEN** a stale-turf save, **WHEN** loaded, **THEN** turf reads Contested.

## Open Questions

- Exact region-number assignments (R1–R9 / R10–R19 / R20–R29 provisional) — owner: implementation; resolve in prototype.
- Lighting plugin pick (overlay approach vs. static gradients) — owner: prototype.
- Spire verticality readability at 48px tile scale — owner: art pass after prototype map test.
