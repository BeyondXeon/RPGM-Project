# Audio Direction

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-19
> **Implements Pillar**: Short, Dense, Finished · The City Remembers

## Overview

Audio Direction is the sound of CHROME HEART CITY: a synthwave BGM set, three district ambient beds, and ONE shared synth SFX kit covering every system — hacks, alerts, payouts, UI, rain, turf shifts — all mutable in settings, all OGG + M4A pairs, no voice ever. Buses follow MZ standards (BGM/BGS/ME/SE) with locked ducking rules so dialogue stays intelligible and pursuit never drowns story. Player-facing, audio is the city's breath: the hum that tells you which district you're in, the sting that tells you you've been seen, the chord that tells you you're a legend now.

## Player Fantasy

You hear the city before you see it. The fantasy is total sonic orientation — the Sump's low hum telling you you're home in the grime, the Market's murmur thickening as you push deeper, the Spire's sterile tone putting your teeth on edge before a single patrol appears. Silence is a signal too: the beat that drops when cameras turn your way, the chord that lands when the board pays out. (Serves Neon Noir atmosphere at its most primal — rain looks wet, but the hum makes it cold.)

## Detailed Design

### Core Rules

1. BGM set (6 tracks, loop-point metadata each): title theme · hub-amber theme · Sump street · Market street · Spire street · battle theme + victory ME. District streets crossfade beds beneath them (beds never stop, BGM sits above).
2. District beds: Sump low industrial hum · Market crowd-murmur synth · Spire sterile high tone. 2s crossfade on district transfer; rain bed layers on streets (intensity-matched to visual rate).
3. ONE shared synth SFX kit — the complete roster, no additions without amending this list: Suspicious sting · Alert siren-blip · de-escalation exhale · pursuit drum loop · breach-entry riser · Hack decrypt loop · overload boom · victory payout chime · row-select blip · accept thunk · payout chime · choice-select blip · consequence sting · turf-shift sting · secret-discovery chime · autosave terminal-blip · legend chord · thunder sting (Hot entry).
4. Ducking stack (fixed priority): dialogue (top, never ducked) > alerts/stings > beds/BGM (beds −30% under dialogue; pursuit loop ducks under dialogue). Nothing ducks dialogue, ever.
5. All audio mutable in settings (master + BGM/BGS/SE sliders); autosave blip and combat-turn SFX excluded from mute-exceptions (no special cases — mute means mute, except the refusal buzz which always plays).
6. Formats: OGG + M4A pairs for every file; BGM <3MB per track (OGG); SFX <200KB each. No voice, ever.
7. The system CANNOT: add off-kit SFX without amending rule 3 · voice any line · exceed the file budgets · duck dialogue for any reason.

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Exploration mix | Beds + streets BGM + rain (streets) | → Combat mix on battle start · → Hub mix in hubs |
| Combat mix | Battle theme + pursuit/alert priority | → Exploration mix on victory/defeat |
| Hub mix | Hub theme + amber calm (no rain bed) | → Exploration mix on exit |
| Muted | Master 0 | All buses silent except refusal buzz |

### Interactions with Other Systems

- **District Maps** (beds per district + turf-shift sting on re-entry).
- **Stealth & Patrol** (Suspicious/Alert/de-escalation/pursuit set).
- **Breach Combat** (riser/decrypt/overload/victory set; battle theme).
- **Gig Board** (row blip/accept/payout set).
- **Dialogue & Events** (ducking beneficiary; choice/consequence set; legend chord).
- **Heat/Rep/Turf** (band-transition stings; Hot thunder).
- **Save-State** (autosave blip; never in combat).
- **Lighting** (rain bed intensity sync).
- **Onboarding** (first-ghost legend chord at low volume — the down payment).

## Formulas

No mathematical formulas exist in this system. The binding numbers are mix levels and file budgets:

`bed_level_dialogue = bed_level × 0.7 · pursuit_level_dialogue = pursuit_level × 0.5 · toast_sfx = fixed`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Bed duck ratio | duck_bed | float | 0.7 fixed | District beds under dialogue (−30%) |
| Pursuit duck ratio | duck_drum | float | 0.5 fixed | Pursuit loop under dialogue |
| BGM file cap | bgm_cap | MB | 3 (OGG) | Per track; M4A mirror required |
| SFX file cap | sfx_cap | KB | 200 | Per file; M4A mirror required |

**Output Range:** mix ratios fixed (not knobs — intelligibility is non-negotiable); file caps are hard build gates.
**Example:** Market bed at 80% volume ducks to 56% under fixer dialogue; returns on close.

## Edge Cases

- **If a BGM file is missing**: fall back to the district bed alone + dev console warning (never silence — beds always play).
- **If M4A mirror is missing**: desktop (OGG) unaffected; browser deploy blocked in QA — pair-check is a build gate.
- **If two stings fire same frame** (turf shift + payout): priority order — payout chime, then turf sting 300ms later; never overlapped.
- **If the player mutes mid-pursuit**: all buses cut instantly except the refusal buzz (safety-critical feedback).
- **If a loop point is misaligned**: audible click = content bug, caught in the audio QA pass (every track auditioned looped 3×).

## Dependencies

**Upstream:** none hard — the MZ WebAudio buses (BGM/BGS/ME/SE) are assumed; this system serves consumers, it needs nothing from them.

**Downstream (serves, owns nothing else's logic):** District Maps (beds + turf sting) · Stealth (alert set + pursuit loop) · Breach (battle set) · Gig Board (UI blips + payout) · Dialogue (ducking beneficiary + choice set + legend chord) · Heat/Rep (band stings + thunder) · Save-State (autosave blip) · Lighting (rain bed sync) · Onboarding (first-ghost chord) · Deck-OS UI (open/confirm/refusal set).

## Tuning Knobs

- **bgm_set** (6 tracks): more = cohesion risk + size; fewer = repetition fatigue in 3–4 hours (the battle theme repeats most — prioritize its variation).
- **duck ratios** (0.7 beds / 0.5 pursuit): fixed by intelligibility rule — adjustable only with a dialogue-clarity playtest proving the new values.
- **file caps** (3MB / 200KB): hard build gates, not knobs — overages recompress, never waive.
- **rain bed intensity curve**: follows the visual rain rate (40/90/150) — retune together, never alone.

## Visual/Audio Requirements

- This system IS the audio layer: all requirements live in Core Rules above. Sync contracts owned here: rain bed ↔ visual rain rate · alert glows ↔ siren timing (same frame budget, light and sound strike together) · turf sting ↔ poster-swap shimmer on map re-entry.

## UI Requirements

- Settings screen: master + BGM/BGS/SE sliders (mutable everything, refusal buzz exempt).

> **📌 UX Flag — Audio Direction**: covered under the Deck-OS settings toggle spec (`/ux-design`, Phase 4).

## Acceptance Criteria

- **GIVEN** any district entry, **WHEN** arrived, **THEN** the correct bed plays with 2s crossfade and the district street BGM loops seamlessly.
- **GIVEN** dialogue open, **WHEN** lines play, **THEN** beds measure −30% and pursuit −50% (meter-verified), dialogue fully intelligible.
- **GIVEN** an Alert, **WHEN** triggered, **THEN** siren-blip + pursuit loop start within 200ms; **WHEN** cleared, **THEN** exhale plays and beds restore.
- **GIVEN** every audio file, **WHEN** audited, **THEN** OGG + M4A pair exists, BGM ≤ 3MB, SFX ≤ 200KB, loops auditioned 3× with no clicks.
- **GIVEN** master mute, **WHEN** set, **THEN** all buses silent except the refusal buzz.

## Open Questions

- Music sourcing (licensed synthwave vs. commissioned vs. AI-assisted composition) — owner: production, budget-dependent.
- SFX kit authorship (single synth patch family for cohesion) — owner: sound pass.
- Battle theme variation needs (one theme for 3–4 hours of fights?) — owner: playtest fatigue check.
