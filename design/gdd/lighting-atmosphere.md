# Lighting & Atmosphere

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-19
> **Implements Pillar**: The City Remembers · Short, Dense, Finished

## Overview

Lighting & Atmosphere is the Neon Noir renderer of CHROME HEART CITY: a capped overlay stack (ambient base + district accent + turf-state preset + dynamic sightline/alert glows) driven by the `chLight` 5-preset enum and turf states that District Maps publishes. It implements the art bible's core laws in light — every screen looks rained-on, light is information, one accent per district — while holding the 60fps budget with a static-gradient fallback. Player-facing, lighting is how the city speaks: you read danger, safety, and ownership in the glow before a single line of dialogue.

## Player Fantasy

You're walking through a film that knows you're there. The fantasy is a city that lights your story — rain hissing on neon that brightens as you earn the streets, shadows that deepen when the corps are hunting, amber warmth spilling from the one door that's always safe. Light never decorates here; it narrates. (Serves the art bible's founding law — every screen looks like it just rained on neon — and *The City Remembers*: turf you earned glows differently than turf you merely visit.)

## Detailed Design

### Core Rules

1. Overlay stack, max 4 layers per map (bottom → top): ambient base (district color temp) · district accent (signage/light-bar glow) · turf-state preset (poster/palette shift per `chLight` preset) · dynamic glow (sightlines, alert states, hackable highlights). Exceeding 4 is a content bug, not a tuning choice.
2. Preset selection is a deterministic lookup: `preset = chLight[districtId]` modified by turf (Owned flavors shift hue within the district accent, never introduce a new hue — art bible one-accent law). The 5 `chLight` values: sump_night, market_neon, spire_cold, hub_amber, alert_red.
3. Rain renders on street maps only; interiors and hubs stay dry (calm contrast — the absence of rain is the safety signal). Rain intensity steps with heat band: Calm light, Warm steady, Hot driving.
4. Fallback: auto-degrade to static gradients when FPS < 55 sustained 5s, plus a manual settings toggle (Full / Static). Degrade order: dynamic glow → rain → turf preset (ambient + accent never drop — identity survives).
5. Sightline/alert glow syncs with Stealth states: Calm translucent accent · Suspicious white pulse + icon · Alert red flash + icon. Lighting consumes Stealth state, never computes detection.
6. The system CANNOT: exceed 4 layers · introduce a second accent hue per district · run rain in hubs/interiors · drop ambient + accent under any fallback.

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Full | All 4 layers live | → Degraded on FPS trigger or manual toggle |
| Degraded | Static gradients, ambient + accent only | → Full on toggle or sustained 58+ FPS for 30s |
| Preset: 5 chLight values | Per-map look | Switched on map entry + turf flip (re-entry rule respected) |

### Interactions with Other Systems

- **District Maps** (in: `chLight` preset + turfState per map; out: nothing — lighting reads, never writes map state).
- **Stealth & Patrol** (in: unit alert states for glow sync + sightline rendering).
- **Heat/Rep/Turf** (in: heat band for rain intensity + Hot vignette).
- **Breach Combat** (in: battle state for battleback accent wash — combat requests, lighting renders).
- **Deck-OS UI** (out: none — UI draws above the overlay stack, unaffected).
- **Save-State** (manual Full/Static toggle persists in settings, not in `ch`).

## Formulas

The `degrade_check` predicate is defined as:

`degrade = fps_avg_5s < 55 ? true : fps_avg_30s ≥ 58 ? false : hold_current`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| 5s FPS average | fps_avg_5s | float | 0–60 | Rolling median frame rate |
| 30s FPS average | fps_avg_30s | float | 0–60 | Recovery gate — sustained smooth play before restoring layers |
| Manual toggle | toggle | enum | Full / Static | Player setting; overrides the predicate in both directions |

**Output Range:** boolean per evaluation tick (1s cadence, not per frame).
**Example:** 52 FPS for 6s → degrade true (dynamic glow drops first); recovers to 59 for 40s → Full restored.

Rain intensity mapping: Calm 40 particles/s · Warm 90/s · Hot 150/s (street maps only; exact counts tuned in prototype against the 60fps budget).

## Edge Cases

- **If FPS oscillates around 55**: hysteresis via the split thresholds (drop at <55/5s, restore at ≥58/30s) — no flicker between states.
- **If turf flips mid-map**: preset swaps apply on map re-entry per District Maps (never hot-swapped under the player).
- **If a map lacks a chLight tag**: load is refused with the missing tag named (District Maps audit rule).
- **If rain + 8 patrols + overlays exceed budget on min-spec**: degrade order is fixed (dynamic → rain → turf preset); if Full still misses 55, the map is overbuilt — cut props, not layers 1–2.
- **If the player forces Static in settings**: choice persists across sessions (settings, not `ch`); no auto-restore prompts.
- **If battle starts during Degraded**: battles render with ambient + accent wash only — no mid-battle layer changes (stability first).

## Dependencies

**Upstream:**
- **District Maps & Exploration** (hard) — `chLight` preset + turfState per map; re-entry rule; 15 overlay sets.
- **Stealth & Patrol** (hard) — unit alert states for glow sync.
- **Heat/Rep/Turf** (hard) — heat band for rain intensity + Hot vignette.

**Downstream:**
- **Breach Combat + Hack** (soft) — renders battleback accent wash on request.
- **Deck-OS UI/HUD** (soft) — UI draws above the stack; no interaction.
- **Save-State** (soft) — Full/Static toggle persists in settings.

## Tuning Knobs

- **layer_cap** (4): more = perf death on min-spec; fewer = turf nuance lost (preset merges into accent).
- **degrade_threshold** (55fps/5s, restore 58/30s): tighter = flickering states; looser = prolonged sub-60 play.
- **rain_rates** (40/90/150 particles/s): higher = atmosphere at perf cost — first dial turned in perf triage.
- **preset palette values** (per chLight preset): owned by the art pass; this system consumes them, never defines hues.

## Visual/Audio Requirements

- Overlay assets: ambient gradient washes per district temp + accent glow tiles for signage + 15 turf preset variants + white/red alert glows + rain sprite sheet. All in Neon Noir palette discipline (one accent per district).
- Rain audio: looped rain bed on streets (intensity-matched to visual rate), silent interiors/hubs; thunder sting on Hot-band entry.

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:lighting-atmosphere` for overlay, glow, and rain specs.

## UI Requirements

- Settings screen: Full / Static lighting toggle with one-line perf note.
- No HUD elements — lighting is purely environmental (Deck-OS UI draws above the stack).

## Acceptance Criteria

- **GIVEN** any street map, **WHEN** entered, **THEN** layers render ambient + accent + turf preset + rain with no console errors.
- **GIVEN** a turf flip, **WHEN** re-entering the map, **THEN** the new preset is active with the district accent preserved (no second hue).
- **GIVEN** FPS below 55 for 5s, **WHEN** measured, **THEN** dynamic glow drops first, then rain, then turf preset — ambient + accent never drop.
- **GIVEN** a hub or interior map, **WHEN** entered, **THEN** no rain renders and amber practicals dominate.
- **GIVEN** a Suspicious unit, **WHEN** visible, **THEN** sightline pulses white + icon (never amber).
- **GIVEN** the settings toggle on Static, **WHEN** saved and reloaded, **THEN** Static persists and no auto-restore occurs.

## Open Questions

- Lighting plugin pick (overlay approach vs. static gradients vs. evented) — owner: prototype (THE decision for this system; GDD specifies requirements only).
- Rain particle counts vs. min-spec budget — owner: prototype measurement.
- Battleback accent wash technique (shared overlay vs. per-battleback art) — owner: art pass.
