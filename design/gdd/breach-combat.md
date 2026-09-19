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
2. Hack command (JS plugin skill — plugin committed, prototype validates): battlefield objects are immortal hidden enemies carrying `<chObject:camera|turret|door>` note-tags, excluded from victory conditions, selected through an aliased `Window_BattleEnemy` filter (per CLAUDE.md code authority: alias-never-overwrite, IIFE, save-safe init). Effects — Camera (stuns ONE enemy 1 turn), Turret (overloads to strike one player-selected enemy once), Door (seals, blocks one reinforcement wave). Each object hackable once per battle.
3. Standard Attack/Skills/Items alongside Hack; Hack costs TP: start 30 per battle, +20 per turn (Camera T1, Door T1, Turret from T2 — costs bind from the opening round).
4. Victory resolves the gig objective; any battle marks approach loud (combat never produces mixed — mixed is stealth-only); rewards flush at gig completion, never mid-battle. Defeat = gig Failed (retry per Gig Board).
5. Forced battles (stealth catch) start with an Alert penalty (enemy preemptive chance up). Escape routes to the same map's entry point: the catching unit stays Alert, all others drop to Suspicious (configure per troop via `BattleManager.canEscape`).
6. Reinforcement waves: black-ice troops + corporate troops in Market/Spire interiors (the top half by roster order — roster pass tags each troop `waves:0|1`) queue exactly 1 wave of 2 units (trigger: turn 3 OR half the troop defeated, whichever first); Sump troops queue none. District sets the HP/turret numbers, tier sets the wave rule — a Market corporate troop uses Market HP with corporate waves. Queued waves are telegraphed (Door target shows armed icon) so sealing is an informed choice, never a gamble.
7. The system CANNOT: Hack with no target object present · Hack the same object twice · save mid-battle · field troops above 4 enemies + 1 object set (perf + readability cap).

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

**Output Range:** 120 to 420 per overload, player-selected single enemy target, once per battle per turret.
**Example:** Market turret overloaded: 250 ± 20% = 200–300 to one enemy.

Enemy HP ladder (turret tuned to a 40–60% chunk, never a kill):

| District | Grunt HP | Brute/Elite HP | Turret roll vs grunt |
|---|---|---|---|
| Sump | 280–340 | 420–520 (brute) | 120–180 ≈ 40–65% |
| Market | 450–550 | 650–800 (brute) | 200–300 ≈ 40–60% |
| Spire | 800–1000 (elite) | 1200+ (boss) | 280–420 ≈ 35–53% |

Hack TP costs (TP max 100, start 30, +20/turn): Camera 20 · Door 30 · Turret 40. Preemptive chance: base 5% + 25% if forced battle (Alert penalty) − 10% with ghost-tier implant (future hook). Stun duration fixed at 1 turn on ONE enemy (not a knob — keeps Hack tactical, not dominant).

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
- **Chrome Implants** (soft) — Hack potency/target hooks in; until that GDD lands, all implant hooks behave as defaults (no modifier, all targets available).
- **Onboarding** (soft) — scripted first battle.

## Tuning Knobs

- **hack_tp_costs** (Camera 20 / Door 30 / Turret 40; start 30, +20/turn): cheaper = Hack dominates Attack; pricier = Hack never fires; lower start delays the first Hack past turn 2.
- **turret_base** (150 + 100/idx): tracks enemy HP growth — retune if troops outgrow Sump math.
- **stun_duration** (LOCKED at 1, ONE enemy): longer stuns or wider targets break encounter balance.
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
- **GIVEN** a Camera Hack, **WHEN** resolved, **THEN** one enemy stunned 1 turn.
- **GIVEN** a Turret Hack, **WHEN** resolved, **THEN** one player-selected enemy takes district-scaled damage per the Formulas table (Sump 120–180 / Market 200–300 / Spire 280–420, ±20%).
- **GIVEN** a Door Hack, **WHEN** a wave is queued (armed icon visible), **THEN** the wave never arrives; **WHEN** no wave queued, **THEN** 30 TP refunded and the object stays hackable.
- **GIVEN** victory, **WHEN** tallied, **THEN** gig records loud (never mixed), credits/approach/heat apply exactly once at gig completion; battle-active save attempts are refused.
- **GIVEN** a forced battle, **WHEN** intro rolls over seeded rolls, **THEN** enemy preemptive chance is 30% (5% base + 25% forced).
- **GIVEN** any troop, **WHEN** counted, **THEN** enemies ≤ 4 plus one object set.
- **GIVEN** two actors targeting the same object, **WHEN** the first resolves, **THEN** the second returns to target selection with turn unconsumed.

## Open Questions

- Side-view battler art pipeline (AI-assisted vs. RTP edits) — owner: art pass, MVP-critical.
- MVP art floor lock (2 crew battlers + 3 Sump enemy types + 1 battleback to start; roster-fallback rule covers the rest) — owner: art pass, pre-implementation gate.
