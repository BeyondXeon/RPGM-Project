# Heat/Rep/Turf Reactivity

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: The City Remembers · Short, Dense, Finished · Ghost or Loud — Your Call

## Overview

Heat/Rep/Turf Reactivity is the memory of CHROME HEART CITY: three meters — Heat (how badly the corps want you right now), Rep (how much the streets respect you, rank 0–4), Turf (who owns each district) — moved by every gig payload, Alert event, and battle, and read by boards, barks, patrols, prices, and endings. Meters are plain integers in `ch`; consequences are templated thresholds, never bespoke branches. Player-facing, the city visibly leans: patrols thicken, vendors warm up, fixers change their tune — every job tilts the streets you walk back through.

## Player Fantasy

You're the weather. The fantasy is visible consequence — walking back through a block you hit last night and finding it changed: new posters, heavier patrols, a vendor who suddenly remembers your name. Heat makes the city hunt you; rep makes it love you; turf decides whose colors fly over it all. Nothing you do evaporates. (Serves *The City Remembers* — this system IS the pillar — and the street-legend core fantasy: legends change the weather.)

## Detailed Design

### Core Rules

1. **Meters**: Heat 0–100 (decays) · Rep as lifetime points mapped to rank 0–4 (None 0 → Legend 4) · Turf per district (Contested / Owned:X).
2. **Approach payloads**: ghost → rep+2, heat+0 · loud → rep+1, heat+3 · mixed → rep+1, heat+1 · messy → rep+0, heat+3. Applied at gig resolution alongside credits — EXCEPT the guided tutorial gig, which suppresses all meter payloads per Onboarding (first real gig is the first meter movement). Ghost play is heat-immune by design — heat content is the loud tax, not a universal system. Payload-only loud (no battles) likewise stays cool: loud means fighting, and heat mastery is about managing combat exposure, not gig selection.
3. **Alert events** feed heat live: Suspicious +1, Alert +3 per incident (one incident = one unit's suspicion crossing 180; same unit cannot contribute again until suspicion fully returns to Calm or map re-entry). Both count as `alert_events` inside the +6 per-gig window cap. Battle noise: +2 per battle fought, +1 extra for escape-loud. All sources within a gig window (payloads + battle + Alerts) cap at +6 total.
4. **Heat bands**: 0–24 Calm (no consequences) · 25–69 Warm (extra patrol check +1 unit via `density_warm = min(8, patrol_density + 1)`, streets only, applied on map entry; vendors +10% flat at hub shops, shown at point of sale; fixers comment) · 70–100 Hot (pursuit squads on streets, interiors gain +1 scripted patrol authored by the gig script, high-heat-risk-flagged gigs lock until heat < 70 with the reason shown pre-accept).
5. **Heat decay**: −5 per district transfer, −20 per hub rest (free rest point per hub, usable once per district entry — spam guard built in); never below 0. Rep never decays.
6. **Turf flips** by gross count: 3 resolved gigs tagged to faction X flips the district to Owned:X regardless of other tags (3-2 majority wins; 2-2-1 stays Contested). Tag counters reset on flip, so districts can change hands repeatedly. Finale thresholds live in the Faction GDD.
7. **Allied/hostile mapping** (unblocks the patrol formula): for district Owned:X, X is allied iff X tops global `faction` standing, hostile otherwise; standing ties break toward the current owner. Contested and Resolved use the fixed +0/−2 mods.
8. The system CANNOT: move turf by walking · let heat rise without a telegraphed cause · gate main-story gigs behind rep (optional gigs only) · exceed the locked `turf_modifier` math (District Maps owns it).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Heat: Calm/Warm/Hot | Pressure band | Moves with heat value; decay on transfer/rest |
| Rep rank 0–4 | Lifetime standing | Rises on payloads; thresholds: 0 (0–2) · 1 (3–5) · 2 (6–9) · 3 (10–12) · 4 (13+) lifetime rep points — one messy gig forgiven for loud players |
| Turf: Contested | No faction at 3 tags | → Owned:X at 3 gross tags for X; counters reset on flip |
| Turf: Owned:X | Faction X skin + patrol mod (allied iff X tops faction standing, ties to owner) | → Owned:Y on fresh 3 tags for Y after reset · → Contested on plot beats |

### Interactions with Other Systems

- **Gig Board** (in: approach + allegiance tags at resolution; out: rep-gating for optional gigs).
- **Stealth & Patrol** (in: Alert events +3 heat; out: heat band feeds pursuit pressure).
- **Breach Combat** (in: battle noise +2, escape-loud +1 extra).
- **District Maps** (out: turf states drive overlay sets + `turf_modifier`; respects locked formula).
- **Dialogue & Events** (out: rep_rank tiers drive bark pools — 0–4 confirmed; turf flavors lines).
- **Faction & Endings** (out: allegiance counts + standing inputs).
- **Save-State** (heat/rep/turf fields in `ch`; all changes persist immediately).
- **Deck-OS UI** (out: meter values for the city-eye display).
- **Onboarding** (guided tutorial gig exempt from ALL payloads — no heat/rep/tags; first real gig is the first meter movement).

## Formulas

The `rep_rank` mapping is defined as:

`rep_rank = rank such that lifetime_rep ≥ threshold[rank]`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Lifetime rep | lifetime | int | 0–30+ | Sum of all rep payloads earned; never decays |
| Rank thresholds | thresholds | int[5] | fixed | rank 0: 0 · rank 1: 3 · rank 2: 6 · rank 3: 10 · rank 4: 13 |

**Output Range:** rank 0–4; ~30 lifetime rep available across 15 gigs; rank 4 lands mid-game for ghost players, late game for loud players (one messy forgiven).
**Example:** 7 lifetime rep → rank 2 (known); bark pools shift, new optional gigs unlock.

The `turf_flip` rule is defined as:

`flip_to(X) when tags[X] ≥ 3 since last reset; reset all tag counters on flip`

Heat band function: `band = heat < 25 ? Calm : heat < 70 ? Warm : Hot`. Decay: `heat = max(0, heat − 5)` per transfer, `max(0, heat − 20)` per hub rest (once per district entry). Per-gig window cap: `min(6, payloads + battle_noise + alert_events)`.

## Edge Cases

- **If heat would exceed 100**: clamped at 100 — Hot is the ceiling, never a game over.
- **If rep payloads arrive at rank 4**: points still accrue (harmless); rank stays 4.
- **If allegiance tags tie** (e.g., 2-2-1): no faction holds 3 — district stays Contested until one reaches gross 3.
- **If decay would drop heat below 0**: clamped at 0.
- **If the player farms Alerts for heat manipulation** (degenerate): each unit grants Alert heat once per map-entry — re-triggering the same patrol yields nothing.
- **If heat sources stack within one gig** (payload + battle + Alerts + escape): the full-window cap holds at +6 — one gig can ruin your evening, not your run.

## Dependencies

**Upstream:**
- **Gig Board & Missions** (hard) — approach + allegiance tags in; adopts finalized payload magnitudes.
- **Stealth & Patrol** (hard) — Alert/Suspicious events in.
- **Breach Combat + Hack** (hard) — battle noise in.
- **District Maps & Exploration** (hard) — turf states + locked `turf_modifier`; respects overlay-set cap.
- **Save-State & Persistence** (hard) — `ch.heat`/`ch.rep`/turf fields; all changes persist immediately.

**Downstream:**
- **Faction & Endings** (hard) — allegiance counts + standing inputs out.
- **Dialogue & Narrative Events** (hard) — rep_rank tiers out (0–4 adopted, no conflict); turf flavors out.
- **Deck-OS UI/HUD** (soft) — meter values out for the city-eye display.

## Tuning Knobs

- **approach payloads** (ghost +2/0, loud +1/+3, mixed +1/+1, messy +0/+3): higher rep = faster Legend, cheaper gig gating; higher heat = Hot becomes the default state — retune together, never alone.
- **rep thresholds** (0/3/6/10/13): lower = Legend by mid-game (bark variety front-loaded); higher = Legend unreachable for loud players (punishes approach choice — pillar violation, do not).
- **heat bands** (25/70): lower Warm threshold = constant pressure; higher = heat is decorative until late.
- **heat decay** (−5 transfer / −20 rest): weaker decay = Hot locks in; stronger = heat never matters. District-bounce wiping (−25/bounce) accepted — Hot still bites mid-district where it matters.
- **turf flip count** (3 gross tags, counters reset): lower = districts flip-flop meaninglessly; higher = turf never flips in a 5-gig district.
- **per-gig heat cap** (+6 full window): the safety valve — touch only if Hot becomes unavoidable.

## Visual/Audio Requirements

- Heat state reads at a glance: Calm (no tint) → Warm (amber edge-pulse on the city-eye) → Hot (Alert Red vignette + icon, never hue alone).
- Turf flips play a district-wide sting + poster-swap shimmer on next map entry.
- Rep rank-ups trigger a legend chord + amber flash; bark-tier changes are silent (discovered through dialogue, not fanfare).
- Audio all from the shared synth kit, mutable in settings.

## UI Requirements

- City-eye meters: circular heat/rep/turf display with numeric readouts (heat always numeric per colorblind rule).
- Band transitions toast in deck-OS terminal style ("HEAT: WARM — patrols thickening").
- Turf ownership shown per district on the map screen.

> **📌 UX Flag — Heat/Rep/Turf**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover the city-eye meters and band toasts **before** writing epics.

## Acceptance Criteria

- **GIVEN** a ghost resolution, **WHEN** payloads apply, **THEN** rep+2, heat+0, autosave carries both.
- **GIVEN** heat crossing 25/70, **WHEN** checked, **THEN** band consequences activate (Warm: density_warm patrol check, +10% hub prices; Hot: pursuit squads, gig locks).
- **GIVEN** a district transfer, **WHEN** taken, **THEN** heat drops 5 and patrol density recomputes.
- **GIVEN** 3 gross tags for one faction, **WHEN** counted, **THEN** district flips to Owned:X with overlay swap on next entry and counters reset.
- **GIVEN** lifetime rep 13, **WHEN** reached, **THEN** rank 4 unlocks legend bark pool + gated gigs.
- **GIVEN** any meter change, **WHEN** saved and reloaded, **THEN** values round-trip exactly.

## Open Questions

- Warm-band vendor pricing (+10%) — flat tax or per-item rounding? Owner: implementation.
- Hot-band gig locks — which gigs lock, and is the lock communicated before accept? Owner: Gig Board roster pass.
- Finale allegiance thresholds (district flips feed them, exact numbers live here or Faction?) — owner: Faction & Endings GDD.
