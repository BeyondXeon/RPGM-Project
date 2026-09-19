# Deck-OS UI/HUD

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-19
> **Implements Pillar**: Chrome Is Character · Short, Dense, Finished

## Overview

Deck-OS UI/HUD is the face of CHROME HEART CITY: eight diegetic deck interfaces — city-eye meters, district map, gig board feed, dialogue/choice/bark layer, save screen, battle HUD + Hack menu, stealth pips + approach projection, skip/hint toasts — built as custom MZ scenes skinned full Neon Noir (clipped corners, thin neon rules, decrypt-in motion, tabular numerals). Every screen obeys the same nav contract (keyboard + d-pad, cancel/confirm rhythm preserved) and the Pillar-4 fallback: if custom-scene velocity drops, reskinned MZ defaults ship with the surface intact. Player-facing, the deck is home — menus feel like the machine you live in, and every readout proves the city is watching back.

## Player Fantasy

You don't open menus — you jack in. The fantasy is total fluency in your own machine: the board flickers alive under your fingers, the city-eye dilates as heat rises, hack targets bloom in accent light the instant they're vulnerable. Every interaction snaps at terminal speed, and the deck's voice — clipped, dry, loyal — makes the chrome feel like a crew member. (Serves *Chrome Is Character*: the deck is the most present party member, and it never leaves your side.)

## Detailed Design

### Core Rules

1. Eight deck surfaces, phased: PHASE 1 (MVP loop) — city-eye meters, gig board feed, dialogue/choice/bark layer, battle HUD + Hack menu. PHASE 2 — district map screen, save screen + dialogs, stealth pips + approach projection, skip prompt + hint toasts, settings toggle. Phase 2 surfaces may ship as reskinned MZ defaults without blocking Phase 1.
2. Every surface obeys the nav contract: full keyboard + d-pad operability (mandatory), mouse supported (optional); cancel/confirm rhythm preserved even in custom layouts; focus state always visible (accent underline + icon, never color alone).
3. Pillar-4 fallback is per-screen: any surface falling behind reverts to its reskinned MZ default (Neon Noir surface: near-black panels, thin accent rules, tabular numerals) with zero behavior change. Fallback is declared per screen in the sprint plan, never mid-sprint silently.
4. Motion standard: 120–180ms decrypt-in snaps on open/select/confirm; no slow fades in combat or infiltration; toasts auto-dismiss in 3s (except refusal dialogs, which require confirm).
5. Type standard: condensed grotesque headers, clean sans body, tabular numerals on ALL meters (heat always numeric per colorblind rule), monospace for terminal readouts only.
6. The system CANNOT: ship a surface without d-pad navigation · encode state in hue alone (icon + numeric + shape backups per art bible) · block gameplay behind a toast (toasts never modal except refusals) · mix custom and default styling on one screen.

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Surface: Custom | Full diegetic scene | → Fallback on velocity call (sprint planning only) |
| Surface: Fallback | Reskinned MZ default | → Custom if scope re-opens (post-launch only, never mid-sprint) |
| Toast: Live → Dismissed | 3s auto or confirm | Terminal |
| Focus: roving highlight | Keyboard/d-pad cursor | Moves per input; never hidden, never hue-only |

### Interactions with Other Systems

- **District Maps** (in: topology + turf states for the map screen).
- **Heat/Rep/Turf** (in: meter values + band transitions for city-eye + toasts).
- **Gig Board** (in: feed rows + filter states + redacted-row rules).
- **Dialogue & Events** (in: message styling, 2-option terminal choices, bark indicators).
- **Save-State** (in: slots + autosave dot + migration/refusal dialog content).
- **Breach Combat** (in: battle HUD layout + Hack menu object list + TP meter).
- **Stealth & Patrol** (in: suspicion pips + approach projection state).
- **Onboarding** (in: skip prompt + hint toast triggers + queue rules).
- **Lighting** (out: none — UI draws above the overlay stack by contract).

## Formulas

No mathematical formulas exist in this system. The binding numbers are motion and timing constants:

`toast_lifetime = 3s (non-modal) · decrypt_snap = 120–180ms · focus_move = instant per input`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Toast lifetime | toast | seconds | 3 fixed | Auto-dismiss; refusal dialogs exempt (require confirm) |
| Decrypt snap | snap | ms | 120–180 | Open/select/confirm animation window; combat + infiltration use the fast end (120ms) |
| Pip display cap | pips | int | 3 | Nearest-first suspicion pips (Stealth owns the data) |

**Output Range:** constants — tunable only via the art pass, never per-screen.
**Example:** band-transition toast appears (decrypt 120ms), lives 3s, dismisses; refusal dialog stays until confirm.

## Edge Cases

- **If toasts stack** (band change + autosave + hint in one frame): queue FIFO, max 3 visible; refusal dialogs jump the queue and pause it.
- **If a custom scene fails to load**: fall back to its reskinned default with a dev console warning (content bug, caught in QA) — never a black screen.
- **If focus is lost** (mouse leaves window, controller disconnects): focus freezes on last element; first input re-anchors visibly — never an invisible cursor.
- **If a meter value is missing** (save gap): display "—" with icon, never 0 (0 is a valid heat reading; absence must look different).
- **If the player opens the map during Alert**: allowed — the map is information, not pause; patrols keep moving (stealth readability preserved).

## Dependencies

**Upstream** (all Approved — read contracts, display data, write nothing):
- **District Maps** (hard) — topology + turf for the map screen.
- **Heat/Rep/Turf** (hard) — meter values + band transitions.
- **Gig Board** (hard) — feed rows + filter/redaction rules.
- **Dialogue & Events** (hard) — message styling + terminal choices + bark indicators.
- **Save-State** (hard) — slots + autosave dot + dialog content.
- **Breach Combat** (hard) — battle HUD + Hack menu + TP meter.
- **Stealth & Patrol** (hard) — pips + approach projection.
- **Onboarding** (hard) — skip prompt + hint toast triggers.
- **Lighting** (soft) — draws above the stack; no interaction.

**Downstream:** none — Deck-OS UI is a pure consumer (leaf node; re-verify after any upstream visual change).

## Tuning Knobs

- **decrypt_snap** (120–180ms): faster = snappier but cheaper-feeling; slower = cinematic but sluggish in combat — combat surfaces lock to 120ms, menus may use 180ms.
- **toast_lifetime** (3s): longer = readable but cluttering; shorter = missed band transitions (the city changes without telling you).
- **fallback declarations** (per screen, sprint-planned): flipping a screen to fallback mid-sprint is forbidden — scope discipline, not quality judgment.
- **Type scale** (headers/body/numerals): owned by the art pass; this system consumes the scale, never sets sizes.

## Visual/Audio Requirements

- Full Neon Noir surface per art bible §7: near-black panels (85% opacity), thin accent rules (one accent at a time), clipped corners, city-eye circular meters, deck-OS decrypt motion.
- Per-surface treatments: city-eye (heat-state tints per Heat/Rep) · gig feed (bounty-terminal rows, redacted-row styling) · message layer (faceset-forward, district-accent speaker names, hub-amber exception) · battle HUD (accent-glow Hack command, TP breach meter) · pips/projection (white pulse language from Stealth).
- Audio: open/select/confirm blips, refusal buzz, payout chime — shared synth kit, mutable.

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:deck-os-ui` for panel, glyph, and meter specs.

## UI Requirements

- This system IS the UI layer: all requirements live in Core Rules + Visual/Audio above. Consolidated screen list for `/ux-design`: city-eye meters · district map · gig board feed · dialogue/choice/bark layer · save screen + dialogs · battle HUD + Hack menu · stealth pips + approach projection · skip prompt + hint toasts · settings toggle.

> **📌 UX Flag — Deck-OS UI**: In Phase 4 (Pre-Production), run `/ux-design` for each screen above **before** writing epics. Stories that reference UI cite `design/ux/[screen].md`, not this GDD.

## Acceptance Criteria

- **GIVEN** any deck surface, **WHEN** navigated by d-pad alone, **THEN** every action is reachable with visible focus and working cancel/confirm.
- **GIVEN** a band transition, **WHEN** it fires, **THEN** a terminal toast appears (120ms decrypt), lives 3s, and shows numeric heat.
- **GIVEN** the gig board, **WHEN** opened, **THEN** rows show title/fixer/payout/heat-risk/district with locked rows redacted-disabled.
- **GIVEN** a battle with hackable objects, **WHEN** the Hack menu opens, **THEN** targets list with TP costs and the command hides when none exist.
- **GIVEN** a custom scene failure, **WHEN** loaded, **THEN** its reskinned default renders with a dev console warning (never black screen).
- **GIVEN** a missing meter value, **WHEN** displayed, **THEN** "—" with icon (never 0).

## Open Questions

- Per-screen custom-vs-fallback assignments — owner: sprint planning (Phase 1 surfaces default custom).
- MZ custom-scene implementation pattern (aliased Scene_Menu vs bespoke scenes) — owner: architecture (ADR).
- Toast queue depth tuning (3 visible cap) — owner: playtest.
