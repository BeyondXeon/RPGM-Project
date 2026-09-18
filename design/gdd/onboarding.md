# Onboarding

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: Short, Dense, Finished · Ghost or Loud — Your Call

## Overview

Onboarding is the first ten minutes of CHROME HEART CITY: meet your fixer, take one guided gig with a scripted ghost success, learn move / hack / fight basics through play — then the board opens and the training wheels come off. It choreographs guided variants of the Gig, Stealth, Dialogue, and Breach systems (scripted-calm patrols, a rigged first battle, beat-template tutorial dialogue) without inventing any new mechanics. Skippable on replay via a global ConfigManager key. If it vanished, new players would face an open board with no instincts — freedom without competence.

## Player Fantasy

You're a natural. The fantasy is instant competence — ninety seconds in you're slipping a patrol, four minutes in you've hacked your first camera, ten minutes in you've ghosted a real gig and the fixer knows your name. No classrooms, no pop-up lectures: the city teaches you by letting you win. (Serves the street-legend core fantasy at its smallest scale — every legend remembers the first job that felt easy.)

## Detailed Design

### Core Rules

1. Five-beat sequence (~10 min): fixer intro 1.5 (hub, amber safety) → movement + interaction basics 1.5 → guided gig 3, scripted ghost success (calm patrols, generous cones) → scripted battle demo 2 (rigged troop, Hack tutorialized: Camera first, then free choice) → board opens with 2 starter gigs + fading hints armed 2. Runtime overruns absorb into beat 5 (free play compresses); authoring cuts hit beat 2 first (competence beats 3–4 are sacred).
2. Guided variants only — no new mechanics: calm patrols (Stealth), rigged troop (Breach), beat-template dialogue (Dialogue), payload override (rule 3).
3. Guided-gig payload override: the guided gig suppresses ALL meter payloads (no heat/rep/tags regardless of approach) and pays street-base credits only — the first real gig is the first meter movement. Tutorial debriefs still record approach for the variant line.
4. Both paths taught: ghost first (the fantasy hook), loud second (the safety net) — the player learns failure has a fun exit before their first real gig.
5. Skippable: replay sense lives OUTSIDE `ch` — a global `ConfigManager` key (`ch_introDone`, survives New Game) drives a title/new-game branch offering skip ("Skip intro?"); skip jumps to board-open with starter gigs, all tutorial flags set.
6. Fading hints (~30 min post-intro): contextual one-liners on first-time triggers (first Alert, first Hack menu, first Warm band); each fires once, tracked in `ch.meta.hints_fired`; never during combat turns.
7. The system CANNOT: exceed 12 minutes unskipped · teach via text dumps (max 3 instruction lines per beat, rest is guided doing) · lock the guided gig's approach (scripted success bends toward ghost but a loud completion still resolves) · re-fire hints after their once-only trigger.

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Intro: Unplayed | Fresh run | → Playing on skip-decline · → Board-Open on skip-accept |
| Intro: Playing | Guided beats 1–5 | → Board-Open on gig resolution |
| Intro: Done | Flag in `ch` | Enables skip prompt on future runs |
| Hints: Armed | Post-intro, unfired triggers live | → Spent per trigger, one shot each |

### Interactions with Other Systems

- **Gig Board** (guided first gig + 2 starter gigs; Sump roster = guided + 2 starters + 2 more = 5, inside the 15-count and faucet math).
- **Stealth & Patrol** (scripted-calm patrols with generous cones for the guided gig).
- **Breach Combat** (rigged first-battle troop; Hack Camera tutorialized).
- **Dialogue & Events** (tutorial beats reuse the beat template; max 3 instruction lines per beat).
- **Heat/Rep/Turf** (guided gig emits no heat; first real gig is the first meter movement — the teaching moment).
- **Save-State** (`ConfigManager` intro-done key + `ch.meta.hints_fired`).
- **Deck-OS UI** (board-open moment is the UI's first full reveal).

## Formulas

No mathematical formulas exist in this system. The binding calculation is the time budget:

`total_intro = beat_1 + beat_2 + beat_3 + beat_4 + beat_5 ≤ 12 min`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Beat durations | beat_N | min | fixed targets | 1 fixer intro · 2 basics · 3 guided gig · 4 battle demo · 5 board-open = 10 min target |
| Budget cap | cap | min | 12 | Hard ceiling; any beat running over steals from beat 5 (free play), never extends the total |

**Output Range:** 8–12 minutes unskipped; ~1 minute skipped.
**Example:** guided gig runs 4 min instead of 3 → board-open beat compresses to 1 min (hints cover the gap).

## Edge Cases

- **If the player goes loud in the guided gig**: resolves normally — the debrief acknowledges the noise via the loud variant. Ghost is scripted likely, never forced.
- **If the player loses the rigged battle**: mercy floor — Troop page, condition Actor HP ≤ 25%, span Once: Change HP sets the actor to 1 + tutorial line plays. A second lethal hit resolves normally (defeat → gig Failed → retry per Gig Board).
- **If the player skips**: all tutorial flags set, starter gigs issued, hints armed — skipping never strands state.
- **If a hint trigger fires mid-combat-turn**: queued to battle end — hints never interrupt turns.
- **If the intro exceeds 12 minutes in testing**: cut from beat 2 (basics), never from beats 3–4 (the competence fantasy lives there).

## Dependencies

**Upstream:**
- **Gig Board & Missions** (hard) — guided first gig + 2 starter gigs; zero-heat scripting.
- **Stealth & Patrol** (hard) — scripted-calm patrols with generous cones.
- **Dialogue & Narrative Events** (hard) — beat-template tutorial dialogue; 3-line instruction cap.
- **Breach Combat + Hack** (hard) — rigged first-battle troop + Camera Hack tutorial.
- **Heat/Rep/Turf** (soft) — first real gig as first meter movement (teaching moment).
- **Save-State** (hard) — global intro-done key + `ch.meta.hints_fired`.

**Downstream:** none — Onboarding is a leaf node (low risk, designed last for exactly this reason).

## Tuning Knobs

- **intro_budget** (10 min target, 12 hard cap): over = players meet the board already tired; under 8 = competence fantasy unproven.
- **instruction_lines_per_beat** (3 max): more = classroom; fewer = confusion — playtest with first-time players, not team members.
- **hint_window** (~30 min post-intro): longer = nagging; shorter = the Warm-band surprise feels unfair.
- **starter_gigs** (2): fewer = board looks empty at the big reveal; more = choice paralysis at minute ten.

## Visual/Audio Requirements

- Intro opens in the Sump hub at amber warmth (safety color = trust the teacher); guided gig shifts to the district accent as training wheels come off.
- Hint toasts in deck-OS terminal style, small, dismissible, never modal.
- First-ghost success gets the legend chord early (quiet) — the fantasy down payment.

## UI Requirements

- Replay skip prompt on new game (when the global intro-done key exists).
- Hint toasts queue outside combat turns.

> **📌 UX Flag — Onboarding**: In Phase 4 (Pre-Production), run `/ux-design` to cover the skip prompt and hint-toast behavior **before** writing epics.

## Acceptance Criteria

- **GIVEN** a fresh run, **WHEN** started, **THEN** fixer intro plays and the guided gig is offered within 3 minutes.
- **GIVEN** the guided gig, **WHEN** completed (ghost or loud), **THEN** approach recorded, zero heat emitted, debrief variant matches approach.
- **GIVEN** the battle demo, **WHEN** played, **THEN** Camera Hack is tutorialized before free choice.
- **GIVEN** intro completion, **WHEN** timed, **THEN** total ≤ 12 minutes unskipped.
- **GIVEN** a completed run on record, **WHEN** starting anew, **THEN** skip prompt appears; skipping sets all tutorial flags and issues starter gigs.
- **GIVEN** each hint trigger, **WHEN** fired, **THEN** it plays once and never repeats.

## Open Questions

- Rigged-battle mercy tuning (25% threshold vs. straight loss allowed?) — owner: prototype.
- Hint trigger list finalization (which firsts earn hints) — owner: playtest with first-timers.
- Skip-prompt wording that doesn't spoil ("Skip intro?" vs. in-fiction framing) — owner: writing pass.
