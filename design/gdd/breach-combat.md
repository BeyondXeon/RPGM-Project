# Breach Combat + Hack

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: Ghost or Loud — Your Call · Chrome Is Character · The City Remembers

## Overview

Breach Combat + Hack is the loud path with brains: MZ turn-based battles where a dedicated Hack command turns cameras, turrets, and doors against the enemy — stunning patrols, overloading turrets to fire on their own side, sealing reinforcements out. Troops are built per gig interior from a small enemy roster; victories resolve to gig outcomes (any battle = loud approach) with rewards flushing at gig completion, never mid-battle. Player-facing, combat is controlled chaos with a hacker's signature — you don't just win fights, you turn the room itself.

## Player Fantasy

You're the breach. The fantasy is turning the enemy's house against them — the camera you hacked spots for you, the turret you overloaded mows down its own squad, the door you sealed trapping reinforcements in the hall while you finish the room. Brute force is for gangers; you fight like a netrunner, and every battle ends with the room owing you one. (Serves *Chrome Is Character* — Hack targets are capability expression — and *Ghost or Loud*: loud, but yours.)

## Detailed Design

### Core Rules

1. Side-view turn-based battles; party of 1–4 crew; troops built per gig interior from a roster of 6–8 enemy types.
2. Hack command (JS plugin skill): targets battlefield objects, not enemies — Camera (stuns a troop 1 turn + reveals hidden), Turret (overloads to strike enemies once), Door (seals, blocks one reinforcement wave). Each object hackable once per battle.
3. Standard Attack/Skills/Items alongside Hack; Hack costs TP (breach meter) that rebuilds per battle.
4. Victory resolves the gig objective; any battle marks approach loud; rewards flush at gig completion, never mid-battle. Defeat = gig Failed (retry per Gig Board).
5. Forced battles (stealth catch) start with an Alert penalty (enemy preemptive chance up).
6. The system CANNOT: Hack with no target object present · Hack the same object twice · save mid-battle · field troops above 4 enemies + 1 object set (perf + readability cap).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Battle: Intro | Preemptive/surprise roll, Alert penalty applied if forced | → Turns |
| Battle: Turns | Standard + Hack actions | → Victory / Defeat / Escape |
| Object: Unhacked → Hacked | Per-object, once per battle | Terminal per battle |
| Approach flag (per gig) | Set loud on any battle | Recorded at gig resolution |

### Interactions with Other Systems

- **Stealth & Patrol** (in: forced-battle trigger + Alert penalty).
- **Gig Board & Missions** (out: battle resolution + loud approach).
- **Save-State** (rewards flush at gig completion only).
- **Chrome Implants** (in: Hack potency/target unlocks, tier 1 in Vertical Slice).
- **Heat/Rep/Turf** (out: battle noise raises heat per payloads).
- **Dialogue & Events** (post-battle barks reuse the bark pools).

## Formulas

Standard attacks and skills use MZ defaults (`a.atk * 4 − b.def * 2`, ±20% variance). Custom Hack formulas below.

The `turret_strike` formula is defined as:

`turret_strike = 150 + 100 × district_index ± 20%`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| District index | idx | int | 0–2 | Sump 0 → 150 · Market 1 → 250 · Spire 2 → 350 (tracks enemy HP growth) |
| Variance | var | ±20% | — | MZ standard damage variance |

**Output Range:** 120 to 420 per overload, single enemy target, once per battle per turret.
**Example:** Market turret overloaded: 250 ± 20% = 200–300 to one enemy.

Hack TP costs (TP max 100, rebuilds per battle): Camera 20 · Door 30 · Turret 40. Preemptive chance: base 5% + 25% if forced battle (Alert penalty) − 10% with ghost-tier implant (future hook). Stun duration fixed at 1 turn (not a knob — keeps Hack tactical, not dominant).

## Edge Cases

- **If Hack is used with no objects present**: the command is hidden (not disabled-gray — cleaner UI, no tease).
- **If two combatants Hack the same object on the same turn**: the first resolves; the second gets a free re-target (no wasted turn).
- **If a turret is overloaded and the battle ends the same turn**: the effect still resolves (reward for setup).
- **If the player escapes a forced battle**: allowed, but the gig records loud + heat+1 extra (you ran loud).
- **If a Door is sealed with no reinforcements queued**: the Door Hack is refunded (no wasted TP on empty value).
- **If the enemy roster is under 6 types at Alpha**: troops may repeat types with tier-scaled stats, but never identical troops back-to-back.

## Dependencies

**Upstream:**
- **Stealth & Patrol** (hard) — forced-battle trigger, Alert penalty, approach grading.
- **Save-State & Persistence** (hard) — no mid-battle saves; rewards flush at gig completion.
- **District Maps & Exploration** (hard) — gig interiors host battles.
- **Dialogue & Narrative Events** (soft) — post-battle barks reuse the bark pools.

**Downstream:**
- **Gig Board & Missions** (hard) — battle resolution + loud approach out.
- **Heat/Rep/Turf Reactivity** (hard) — battle noise payloads out.
- **Chrome Implants** (soft) — Hack potency/target hooks in.
- **Onboarding** (soft) — scripted first battle.

## Tuning Knobs

- **hack_tp_costs** (Camera 20 / Door 30 / Turret 40): cheaper = Hack dominates Attack; pricier = Hack never fires.
- **turret_base** (150 + 100/idx): tracks enemy HP growth — retune if troops outgrow Sump math.
- **stun_duration** (LOCKED at 1): longer stuns break encounter balance.
- **troop_cap** (4 enemies + 1 object set): more = unreadable battlefield + slow turns.
- **preemptive_rate** (5% base, +25% forced): higher = forced battles feel doomed; lower = the Alert penalty is theater.
- **enemy_roster_size** (6–8 types): fewer = repetitive troops; more = art debt.

## Visual/Audio Requirements

- Side-view battlers for 3–4 crew + 6–8 enemy types in the Neon Noir palette — the biggest art line-item in the MVP (flagged for the art pass).
- Battlebacks per district accent over the Abyss base.
- Hack visuals: camera swivel + scanline · turret muzzle-glow then friendly fire · door slam + seal glyph.
- Camera/turret cones visually distinct from patrol cones (thinner, dashed, cyan-white) — resolves Stealth's open question.
- Audio: breach-entry riser, Hack decrypt loop, overload boom, victory payout chime — all from the shared synth kit.

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:breach-combat` for battler, battleback, and Hack-effect specs.

## UI Requirements

- Hack command glows accent when objects are hackable (hidden otherwise).
- TP breach meter under each actor.
- Object target icons match map-side hackable glyphs.

> **📌 UX Flag — Breach Combat**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover the battle HUD and Hack menu **before** writing epics.

## Acceptance Criteria

- **GIVEN** a battle with objects, **WHEN** the Hack menu opens, **THEN** Camera/Turret/Door targets list with TP costs.
- **GIVEN** a Camera Hack, **WHEN** resolved, **THEN** one troop stunned 1 turn and hidden revealed.
- **GIVEN** a Turret Hack, **WHEN** resolved, **THEN** one enemy takes district-scaled damage.
- **GIVEN** a Door Hack, **WHEN** reinforcements queued, **THEN** the wave never arrives.
- **GIVEN** victory, **WHEN** tallied, **THEN** gig records loud and rewards flush at completion with no mid-battle save.
- **GIVEN** a forced battle, **WHEN** intro rolls, **THEN** enemy preemptive chance is elevated.
- **GIVEN** any troop, **WHEN** counted, **THEN** enemies ≤ 4 plus one object set.

## Open Questions

- Side-view battler art pipeline (AI-assisted vs. RTP edits) — owner: art pass, MVP-critical.
- Hack plugin vs. pure-evented final call — owner: prototype.
- Reinforcement wave design per troop — owner: troop design pass.
