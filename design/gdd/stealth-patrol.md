# Stealth & Patrol

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: Ghost or Loud — Your Call · Short, Dense, Finished · The City Remembers

## Overview

Stealth & Patrol is the detection machinery of CHROME HEART CITY: one templated patrol rig — looped routes on R1–R9 regions, facing-based sightlines, three alert states — reused across every street map and gig interior, with unit counts set by the locked `patrol_density` formula. Detection resolves to approach records (ghost/loud/mixed) that gigs, debriefs, and meters consume; spotting never ends the game, it escalates it. Player-facing, infiltration is coiled problem-solving — reading rhythms, slipping cones, and choosing every few seconds whether to stay a ghost or go loud.

## Player Fantasy

You're smoke. The fantasy is untouchable rhythm — gliding through cones by half a tile, timing a patrol turn to the frame, vanishing behind a signboard as the balloon pops. And when it breaks, it's your choice how: melt away and stay a ghost, or kick the door in and make the noise mean something. Close calls are the currency; the city pays out in held breath. (Serves *Ghost or Loud* at its purest and the street-legend core fantasy: nobody saw you — everybody heard about it.)

## Detailed Design

### Core Rules

1. One templated rig reused everywhere: looped move-route patrols on R1–R9; a parallel detector checks facing + Manhattan distance within cone range, walls block via passability.
2. Cone = facing direction, range 4 tiles, ±1 lateral spread; cameras/turrets are stationary units with fixed cones.
3. Alert states run Calm → Suspicious → Alert. Suspicious telegraphs (balloon icon, SE sting, patrol pauses to investigate). Decay: Suspicious → Calm after 5s unseen; Alert → Suspicious after 8s broken sight.
4. Alert = pursuit at +1 speed; catch (adjacent tile) = forced battle = loud approach; leaving the map resets to Calm.
5. Approach grading: never Alert = ghost · Alert evaded without battle = mixed · any battle = loud (recorded at gig resolution).
6. The system CANNOT: instant-fail on spot · allow untelegraphed suspicion (every change gets balloon + SE + light cue) · place patrols in hubs · run detector parallelism above the 16.6ms budget (stagger checks across frames).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Calm (per unit) | Patrolling, unaware | → Suspicious on player in cone |
| Suspicious (per unit) | Investigating, telegraphed | → Alert on sustained visibility · → Calm after 5s unseen |
| Alert (per unit) | Pursuing | → Suspicious after 8s broken sight · → Calm on map exit |
| Approach: Ungraded → Ghost/Mixed/Loud (per gig) | Gig-level record | Graded at gig resolution per rule 5 |

### Interactions with Other Systems

- **District Maps** (in: region zones + `patrol_density`; out: alert events feed heat).
- **Gig Board** (out: approach recorded at gig resolution).
- **Heat/Rep/Turf** (out: Alert events raise heat).
- **Breach Combat** (out: forced-battle trigger on catch).
- **Dialogue & Events** (approach variants in debriefs).
- **Chrome Implants** (in: future cone/range modifiers — hooks reserved, values later).
- **Lighting & Atmosphere** (sightline glow sync with alert states).
- **Onboarding** (scripted-calm patrols for the guided first gig).

## Formulas

The `suspicion` accumulation is defined as:

`suspicion(t+1) = clamp(suspicion(t) + visible ? +1 : −decay_state, 0, 180)`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Build rate | rate | int | +1/frame | Fixed. Suspicious (60) reached in ~1s of visibility; Alert (180) in ~3s |
| Grace freeze | grace | int | frames | While unseen, suspicion holds for 300 frames (5s) before decaying — implements the Suspicious→Calm rule |
| Decay rate | decay_state | int | −2/frame | After grace, drains 60 points in 30 frames (0.5s) — de-escalation feels crisp, not leaky |
| Alert break | break_T | int | 480 frames | 8s broken sight drops suspicion to 59 (top of Suspicious) — implements Alert→Suspicious |

**Output Range:** suspicion ∈ [0, 180]; <60 Calm · 60–179 Suspicious · ≥180 Alert.
**Example:** visible 90 frames → suspicion 90 → Suspicious, investigating; break sight 5s → drains to 0 → Calm. Visible 180+ frames → Alert, pursuit; break sight 8s → suspicion 59 → Suspicious.

Cone geometry (exact tile set): facing direction D, range 4 — tiles D×1 through D×4, plus lateral ±1 at distances 1–3. Hiding spots and future implant modifiers subtract effective range (hooks reserved, values in the Implants GDD).

## Edge Cases

- **If two detectors see the player on the same frame**: suspicion accrues once per frame (single accumulator per map, not per unit) — no double-jeopardy.
- **If the player saves during Alert**: on load, all units reset to Calm with patrols at route starts (documented mercy rule).
- **If the player stands in a cone behind an event-blocking prop**: passability blocks — props are cover by construction.
- **If pursuit crosses a map transfer**: pursuit ends; the destination map starts Calm (District Maps transfer rule respected).
- **If a forced battle is defeated**: gig Failed state (Gig Board owns); stealth resets on retry.
- **If detector parallelism spikes frame time**: stagger unit checks across frames (even/odd units alternate); if still over budget, cut cone range before cutting unit count — density is a locked promise, range is a knob.

## Dependencies

**Upstream:**
- **District Maps & Exploration** (hard) — region zones, `patrol_density`, interior/scripted-patrol rule, transfer-calm rule.
- **Dialogue & Narrative Events** (soft) — Suspicious/Alert barks reuse the bark pools.

**Downstream:**
- **Gig Board & Missions** (hard) — approach grading out (ghost/mixed/loud).
- **Heat/Rep/Turf Reactivity** (hard) — Alert events out.
- **Breach Combat + Hack** (hard) — forced-battle trigger out.
- **Onboarding** (hard) — scripted-calm patrols for the guided gig.
- **Chrome Implants** (soft) — cone/range modifier hooks reserved.
- **Lighting & Atmosphere** (soft) — sightline glow sync with alert states.

## Tuning Knobs

- **cone_range** (4): longer = unfair, unreadable cones; shorter = trivial stealth.
- **sus_threshold** (60): lower = twitchy patrols; higher = free ghosts.
- **alert_threshold** (180): lower = constant pursuit; higher = Alert never fires.
- **grace_freeze** (300): shorter = suspicion feels leaky; longer = no consequence for peeking.
- **pursuit_speed** (+1): higher = unescapable; lower = pursuit is theater.
- **patrol_density formula** (LOCKED): owned by District Maps — tune range and thresholds instead, never unit counts.

## Visual/Audio Requirements

- Sightlines glow in district accent at Calm (translucent), pulse amber at Suspicious, flash Alert Red at Alert — always with icon/brightness change, never hue alone.
- Patrol units get alert-state balloon + rim-light shift; pursuit adds a screen-edge red vignette.
- Audio: Suspicious sting + Alert siren-blip + de-escalation exhale from the shared synth kit; pursuit drum loop while Alert (mutable, ducks under dialogue).

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:stealth-patrol` for cone/balloon/vignette specs.

## UI Requirements

- Overhead suspicion pips per visible unit (max 3 shown, nearest-first).
- Gig HUD shows live approach projection (GHOST / MIXED / LOUD-in-progress).

> **📌 UX Flag — Stealth & Patrol**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover stealth pips and approach projection **before** writing epics.

## Acceptance Criteria

- **GIVEN** a patrol facing away, **WHEN** the player crosses behind, **THEN** suspicion stays 0.
- **GIVEN** 1s in-cone, **WHEN** exposed, **THEN** the unit enters Suspicious with balloon + SE.
- **GIVEN** 3s in-cone, **WHEN** exposed, **THEN** Alert pursuit begins.
- **GIVEN** broken sight 5s, **WHEN** Suspicious, **THEN** the unit returns to Calm.
- **GIVEN** a catch, **WHEN** adjacent, **THEN** forced battle starts and the gig records loud.
- **GIVEN** a ghost run, **WHEN** resolved, **THEN** approach grades ghost with zero Alerts logged.
- **GIVEN** 8 patrol units, **WHEN** active, **THEN** frame time stays under 16.6ms.

## Open Questions

- Parallel-detector perf at 8 units on min-spec hardware — owner: prototype measurement (THE high-risk validation for this system).
- Balloon vs. rim-light readability at 48px — owner: art pass.
- Camera/turret cone visuals distinct from patrol cones — owner: Breach Combat GDD.
