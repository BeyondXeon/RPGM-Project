# Faction & Endings

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-19
> **Implements Pillar**: The City Remembers · Short, Dense, Finished · Ghost or Loud — Your Call

## Overview

Faction & Endings is the verdict of CHROME HEART CITY: cumulative standing across Wardens / Chrome / Ghosts — fed by gig allegiance tags (+1) and story choice beats (+2) — resolves at the Spire finale into one of 3 meter-driven endings staged on a SINGLE finale map with outcome variants (never tripled content). Post-credits, districts flip to Resolved free roam; replayability comes from short runtime, not branches. Player-facing, every late-game gig feels weighted: you're not just working, you're voting — and the city will announce the winner.

## Player Fantasy

You're the kingmaker. The fantasy is watching the whole city add up your choices and hand you the crown to place — the fixer who calls with three offers, the crew who argues about who you've become, the skyline that changes color with your answer. Endings aren't rewards for finishing; they're verdicts on how you played. And the best one is the one you argue about afterward. (Serves *The City Remembers* at its absolute peak and the street-legend core fantasy: legends don't retire, they get remembered by whoever writes history.)

## Detailed Design

### Core Rules

1. Standing inputs: gig allegiance tags +1 each (resolved gigs only) · story choice beats +2 each (both options write — choices shift weight, never abstain). Totals live in `ch.faction{wardens, chrome, ghosts}`.
2. Finale unlocks when Spire main gigs resolve (optional gigs may remain — the finale never waits for completionists).
3. Ending selection: argmax of `faction` standing wins; exact ties resolve via a final binary choice beat between the tied factions (drama, not coin-flip). The choice itself writes the deciding +2.
4. ONE finale map (Spire apex), 3 outcome variants: shared infiltration spine (ghost-or-loud entry per Stealth/Breach contracts) diverging at the crown sequence — variant scenes, faction-skin overlays, distinct final choice + epilogue slides. Never tripled maps.
5. Post-credits: all districts flip to Resolved (patrols persist at −2, no new gigs); free roam + replay prompt (short game = replayable game).
6. The system CANNOT: lock the finale behind optional gigs · resolve ties randomly · triple finale maps · let heat/rep gate ending eligibility (meters flavor endings, never lock them).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Standing: open | Accumulating through gigs + beats | → Locked at finale entry |
| Finale: available | Spire mains resolved | → Played on entry (one shot per run) |
| Ending: Wardens/Chrome/Ghosts | Variant played, credits | → Resolved free roam |
| Resolved | Post-finale city | Terminal (replay via new run) |

### Interactions with Other Systems

- **Gig Board** (in: allegiance tags +1 per resolved gig).
- **Dialogue & Events** (in: choice beats +2; finale tiebreak uses the beat template).
- **Heat/Rep/Turf** (in: turf states flavor the finale staging; meters never gate eligibility).
- **District Maps** (out: all districts → Resolved on credits).
- **Breach/Stealth** (finale entry honors ghost-or-loud contracts).
- **Save-State** (`faction{}` fields + finale flags in `ch`; ending recorded for replay prompt).
- **Deck-OS UI** (ending screens + epilogue slides per variant).

## Formulas

The `ending_select` rule is defined as:

`ending = unique_max(faction) ? argmax : tiebreak_choice(tied)`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Faction standing | faction{X} | int | 0+ | Cumulative: gig tags +1, choice beats +2 |
| Unique max | unique_max | bool | — | Exactly one faction strictly tops the other two |
| Tiebreak | tiebreak | choice beat | binary | Final binary choice between tied factions; writes the deciding +2 |

**Output Range:** one of Wardens / Chrome / Ghosts, always.
**Example:** standings {wardens: 9, chrome: 9, ghosts: 4} → tiebreak beat Wardens-vs-Chrome → player picks Chrome (+2) → ending: Chrome, final {9, 11, 4}.

## Edge Cases

- **If all three standings tie**: binary choice cannot cover three — final choice offers the top TWO by (standing, then most-recent-tag recency); the third is narrated as withdrawing. Ties beyond pairwise are resolved by recency, never randomly.
- **If the player enters the finale with near-zero standings** (speedrun, skipped optionals): argmax still resolves (lowest totals still crown someone); epilogue acknowledges the thin mandate ("The city barely knows your name — yet it obeys").
- **If a standing field is missing/corrupt**: coerce to 0 + dev warning (save-state coercion rule); finale proceeds with remaining totals.
- **If the player abandons the finale map mid-run**: finale state resets to available (one-shot flag only set on credits); standings untouched.
- **If turf states contradict the ending** (district Owned:Y, ending X wins): credits sequence re-skins all districts to the winner before Resolved free roam — the ending outranks turf, always.

## Dependencies

**Upstream:**
- **Gig Board & Missions** (hard) — allegiance tags +1 per resolved gig.
- **Dialogue & Narrative Events** (hard) — choice beats +2; tiebreak uses the beat template.
- **Heat/Rep/Turf** (hard) — turf states for finale staging; meters never gate eligibility.
- **District Maps** (hard) — Spire completion trigger; Resolved flip target.
- **Save-State** (hard) — `faction{}` + finale flags in `ch`.

**Downstream:**
- **Deck-OS UI/HUD** (soft) — ending screens + epilogue slides.
- **Breach/Stealth** (soft) — finale entry honors their contracts (consumers, not owners).

## Tuning Knobs

- **choice_weight** (+2 vs gig +1): higher = story beats dominate gigs (railroady); lower/equal = finale decided by grind volume, not choices.
- **finale_trigger** (Spire mains): earlier trigger = shorter game, thinner standings; later (full completion) = finale waits on completionists.
- **tiebreak design** (binary + recency): three-way ties need the recency rule — removing it reintroduces randomness.
- **epilogue slide count** (per ending): more slides = richer verdicts; each slide is writing + art cost against Pillar 4.

## Visual/Audio Requirements

- Finale map (Spire apex): panoramic city vista, winner-faction accent wash over the vista (single accent per victor — no soup), crown-sequence lighting shift per variant.
- Ending screens: faction sigil + epilogue slides in deck-OS terminal style; each variant gets a distinct sting (Wardens: cold brass · Chrome: distorted guitar · Ghosts: lone synthwave lead) from an extended — not new-kit — palette.
- Post-credits Resolved state: calm graded lighting, pursuit audio retired permanently.

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:faction-endings` for vista, sigil, and sting specs.

## UI Requirements

- Tiebreak choice beat uses the terminal 2-option UI (d-pad navigable).
- Epilogue slides advance on confirm; skippable after first viewing.

> **📌 UX Flag — Faction & Endings**: covered under the Deck-OS choice/slide specs (`/ux-design`, Phase 4).

## Acceptance Criteria

- **GIVEN** Spire main gigs resolved, **WHEN** checked, **THEN** the finale unlocks regardless of optional completion.
- **GIVEN** a unique standing max, **WHEN** the finale resolves, **THEN** the argmax ending plays with no player choice required.
- **GIVEN** an exact two-way tie, **WHEN** the finale resolves, **THEN** a binary tiebreak beat plays and the pick decides.
- **GIVEN** credits, **WHEN** finished, **THEN** all districts read Resolved with winner skin and free roam begins.
- **GIVEN** any ending, **WHEN** counted, **THEN** variant content shares the single finale map (no tripled maps).

## Open Questions

- Epilogue slide count per ending (richness vs Pillar-4 cost) — owner: writing pass.
- New Game+ vs fresh-run replay (carry nothing vs carry codex?) — owner: release scope decision.
- Winner-skin overlay technique for all districts post-credits — owner: art pass (reuse turf overlay pipeline).
