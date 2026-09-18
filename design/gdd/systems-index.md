# Systems Index: CHROME HEART CITY

> **Status**: Draft
> **Created**: 2026-09-18
> **Last Updated**: 2026-09-18
> **Source Concept**: design/gdd/game-concept.md

---

## Overview

CHROME HEART CITY is a 3–4 hour cyberpunk street-crew JRPG (RPG Maker MZ): an open-zone gig loop (take jobs, ghost-or-loud infiltration, hackable turn-based battles) wrapped in city-wide reactivity (heat/rep/turf meters, 3 faction endings), skinned in a full-diegetic deck-OS UI under the Rain-Slick Neon Noir art bible. 14 systems across gameplay, world, persistence, and presentation serve the pillars — Ghost or Loud, The City Remembers, Chrome Is Character, Short/Dense/Finished — with the MVP proving the gig loop in a single district before volume and identity land in the Vertical Slice.

---

## Systems Enumeration

| # | System Name | Category | Priority | Status | Design Doc | Depends On |
|---|-------------|----------|----------|--------|------------|------------|
| 1 | Gig Board & Missions | Gameplay | MVP | In Review | design/gdd/gig-board.md | District Maps, Save-State |
| 2 | Stealth & Patrol | Gameplay | MVP | In Review | design/gdd/stealth-patrol.md | District Maps |
| 3 | Breach Combat + Hack | Gameplay | MVP | In Review | design/gdd/breach-combat.md | Save-State |
| 4 | Heat/Rep/Turf Reactivity | Gameplay | MVP | Approved | design/gdd/heat-rep-turf.md | Gig Board, Save-State |
| 5 | Chrome Implants | Progression | Vertical Slice | Not Started | — | Breach Combat, Stealth |
| 6 | Faction & Endings | Narrative | Vertical Slice | Not Started | — | Heat/Rep/Turf, Dialogue |
| 7 | Crew & Recruitment | Narrative | Alpha | Not Started | — | Breach Combat, Dialogue |
| 8 | District Maps & Exploration | World | MVP | Approved | design/gdd/district-maps.md | — |
| 9 | Save-State & Persistence (inferred) | Persistence | MVP | Approved | design/gdd/save-state.md | — |
| 10 | Onboarding (inferred) | Meta | MVP | Designed | design/gdd/onboarding.md | Gig Board, Stealth |
| 11 | Deck-OS UI/HUD | UI | Vertical Slice | Not Started | — | Gig Board, Heat/Rep/Turf |
| 12 | Lighting & Atmosphere (inferred) | Presentation | Vertical Slice | Not Started | — | District Maps |
| 13 | Audio Direction (inferred) | Audio | Vertical Slice | Not Started | — | — |
| 14 | Dialogue & Narrative Events (inferred) | Narrative | MVP | In Review | design/gdd/dialogue-events.md | — |

---

## Categories

| Category | Description | Systems |
|----------|-------------|---------|
| **Gameplay** | The gig loop: jobs, infiltration, combat, reactivity | Gig Board, Stealth & Patrol, Breach Combat + Hack, Heat/Rep/Turf |
| **Progression** | How the player grows | Chrome Implants |
| **World** | Where the game happens | District Maps & Exploration |
| **Persistence** | Save state and continuity | Save-State & Persistence |
| **Narrative** | Story and relationship delivery | Faction & Endings, Crew & Recruitment, Dialogue & Events |
| **UI** | Deck-OS information displays | Deck-OS UI/HUD |
| **Presentation** | Look, light, and sound | Lighting & Atmosphere, Audio Direction |
| **Meta** | Outside the core loop | Onboarding |

---

## Priority Tiers

| Tier | Definition | Target Milestone | Design Urgency |
|------|------------|------------------|----------------|
| **MVP** | Required for the core loop to function. Without these, you can't test "is this fun?" | First playable prototype | Design FIRST |
| **Vertical Slice** | Required for one complete, polished area. Demonstrates the full experience. | Vertical slice / demo | Design SECOND |
| **Alpha** | All features present in rough form. Complete mechanical scope, placeholder content OK. | Alpha milestone | Design THIRD |
| **Full Vision** | Polish, edge cases, nice-to-haves, and content-complete features. | Beta / Release | Design as needed |

---

## Dependency Map

### Foundation Layer (no dependencies)

1. District Maps & Exploration — the spatial substrate every system plays out on
2. Save-State & Persistence — the data substrate reactivity is recorded in
3. Dialogue & Narrative Events — the eventing contract gigs and story are built on
4. Audio Direction — standalone; RTP placeholders decouple it from everything

### Core Layer (depends on foundation)

1. Gig Board & Missions — depends on: District Maps, Save-State ◆ bottleneck
2. Stealth & Patrol — depends on: District Maps
3. Breach Combat + Hack — depends on: Save-State (rewards/persistence)

### Feature Layer (depends on core)

1. Heat/Rep/Turf Reactivity — depends on: Gig Board, Save-State
2. Chrome Implants — depends on: Breach Combat, Stealth
3. Crew & Recruitment — depends on: Breach Combat, Dialogue
4. Faction & Endings — depends on: Heat/Rep/Turf, Dialogue
5. Onboarding — depends on: Gig Board, Stealth (teaches the Core loop from above)

### Presentation Layer (depends on features)

1. Deck-OS UI/HUD — depends on: Gig Board, Heat/Rep/Turf (wraps gigs + meters)
2. Lighting & Atmosphere — depends on: District Maps (wraps maps in Neon Noir)

### Polish Layer (depends on everything)

- None yet — polish pass scheduled at Full Vision without dedicated systems.

---

## Recommended Design Order

| Order | System | Priority | Layer | Est. Effort |
|-------|--------|----------|-------|-------------|
| 1 | District Maps & Exploration | MVP | Foundation | M |
| 2 | Save-State & Persistence | MVP | Foundation | S |
| 3 | Dialogue & Narrative Events | MVP | Foundation | M |
| 4 | Gig Board & Missions | MVP | Core | M |
| 5 | Stealth & Patrol | MVP | Core | L |
| 6 | Breach Combat + Hack | MVP | Core | L |
| 7 | Heat/Rep/Turf Reactivity | MVP | Feature | M |
| 8 | Onboarding | MVP | Feature | S |
| 9 | Lighting & Atmosphere | Vertical Slice | Presentation | M |
| 10 | Deck-OS UI/HUD | Vertical Slice | Presentation | L |
| 11 | Audio Direction | Vertical Slice | Presentation | S |
| 12 | Faction & Endings | Vertical Slice | Feature | M |
| 13 | Chrome Implants | Vertical Slice | Feature | S |
| 14 | Crew & Recruitment | Alpha | Feature | M |

---

## Circular Dependencies

None found. Heat/Rep/Turf and Faction & Endings flow one direction (meters → standing → endings); no system reads back upstream.

---

## High-Risk Systems

| System | Risk Type | Risk Description | Mitigation |
|--------|-----------|-----------------|------------|
| Stealth & Patrol | Technical | Event-system stealth (sightlines, alert states) unproven in MZ at this density | Prototype first, regardless of tier; template one patrol rig and reuse |
| Deck-OS UI/HUD | Scope | Full-diegetic custom UI multiplies plugin/scene work | UI shell in MVP; Pillar 4 fallback to reskinned MZ defaults |
| Heat/Rep/Turf Reactivity | Scope | "Full reactivity" can explode into bespoke branches per gig | Meter-driven gating + templated reactions, never bespoke branches |
| Gig Board & Missions | Design | Bottleneck system — weak gig structure sinks Heat, Faction, Onboarding, UI | Design early (order #4); keep gig template rigid (hook → approach → twist → payout) |

---

## Progress Tracker

| Metric | Count |
|--------|-------|
| Total systems identified | 14 |
| Design docs started | 1 |
| Design docs reviewed | 0 |
| Design docs approved | 0 |
| MVP systems designed | 8/8 |
| Vertical Slice systems designed | 0/5 |

---

## Next Steps

- [ ] Review and approve this systems enumeration
- [ ] Design MVP-tier systems first (use `/design-system [system-name]`)
- [ ] Run `/design-review` on each completed GDD
- [ ] Run `/gate-check pre-production` when MVP systems are designed
- [ ] Validate the highest-risk systems with `/vertical-slice` before committing to Production
