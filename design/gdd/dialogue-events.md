# Dialogue & Narrative Events

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: The City Remembers · Short, Dense, Finished · Ghost or Loud — Your Call

## Overview

Dialogue & Narrative Events is the storytelling contract of CHROME HEART CITY: every story beat, gig briefing, faction choice, crew banter line, and reactive street bark is delivered through templated MZ events (Show Text/Choices, switch-conditioned pages, common events) that read and write the `ch` save schema. Structure is rigid — beats follow fixed templates with variant lines, never sprawling trees — so writing stays shippable and every choice lands on a meter, a flag, or a relationship. Player-facing, conversations are where the city talks back: fixers remember your approach, crews bicker about your calls, and districts greet the legend you're becoming — or the problem you've become.

## Player Fantasy

You're the name on everyone's lips. The fantasy is being *known*: the fixer who greets ghosts differently than loudmouths, the crew who tease you about that Spire job, the street vendor whose prices shift with your rep. Every conversation proves the city was watching — and makes you hungry to give it something worth talking about. (Serves *The City Remembers* at its most literal and the street-legend core fantasy: legends are made of what people say about them.)

## Detailed Design

### Core Rules

1. Every story beat follows one template: hook (1–3 lines) → binary choice → consequence lines → meter/flag write.
2. Both options always write somewhere — no dead choices.
3. Gig briefings/debriefings use the same template; debriefs acknowledge ghost/loud/mixed approach with at least one variant line.
4. Bark pools are tiered by rep rank + turf state, max 3 tiers per pool; hub + street pools per district.
5. Crew auto-banter fires at triggers (first-entry, hub return post-gig, faction beats, recruits) — max 3 exchanges per banter, no chains.
6. The system CANNOT: offer choiceless choices · nest choices (depth is always choice→consequence) · run speaker turns over 6 lines without a break or choice.

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Beat: Unplayed | Not yet seen | → Played on completion, choice recorded in `ch` |
| Beat: Played | Seen; choice locked | Terminal (no replays change history) |
| Bark pool tier | Resolved from rep rank + turf at interaction time | Re-resolved every interaction |
| NPC pages | Event pages gate on quest/standing flags | Page up as flags advance |

### Interactions with Other Systems

- **Gig Board & Missions** — briefings/debriefings frame gigs; writes gig status.
- **Faction & Endings** — choice beats write faction standing.
- **Crew & Recruitment** — banter writes relationship flags.
- **Heat/Rep/Turf** — ranks select bark tiers + greeting variants.
- **Save-State** — all narrative writes land in `ch` fields.
- **Onboarding** — tutorial beats reuse the same template.

## Formulas

The `bark_tier` resolution is defined as:

`bark_tier = pool_index[rep_rank] with line_variant[turfState]`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Reputation rank | rep_rank | int | 0–4 | Player rep rank (None 0 → Legend 4) |
| Pool index | pool | int | 0–2 | Street pool: ranks 0–1 → 0 (unknown), ranks 2–3 → 1 (known), rank 4 → 2 (legend); hub pools shift +0 (hubs are friendly ground) |
| Turf flavor | turf | enum | Contested / Owned:X / Resolved | Selects the line variant within the pool, never a different pool |

**Output Range:** one pool (0–2) + one variant per interaction; exactly 3 pools per district per context (street/hub), 18 pools game-wide max (Pillar 4 cap).
**Example:** rep rank 3 in Market, turf Owned:Red → street pool 1, Red-flavored lines.

## Edge Cases

- **If a Played beat is re-triggered**: show a one-line recap, no re-choice — history is immutable.
- **If a beat fires while another beat is active**: queued — beats never interrupt beats; banter waits for the beat to end.
- **If an NPC is talked to while alerted in a restricted zone**: barks suppressed — stealth readability first.
- **If a turf-flavor line is missing**: fall back to Contested lines silently in release (content gap — caught in the QA content pass).
- **If a debrief has an unclassifiable approach**: default to the mixed variant.
- **If a line overflows the message window**: writing pass enforces per-turn caps; overflow is a content bug, not an engine problem.

## Dependencies

**Upstream:** none hard — the MZ eventing substrate is assumed. Reads `rep_rank`/turf from the Heat/Rep + District contracts (both undesigned — the 0–4 rank scale is defined here provisionally; the Heat/Rep GDD owns it and may revise with a conflict flag).

**Downstream:**
- **Gig Board & Missions** (hard) — briefing/debriefing template + gig status writes.
- **Faction & Endings** (hard) — choice beats write faction standing.
- **Crew & Recruitment** (hard) — banter writes relationship flags.
- **Onboarding** (hard) — tutorial beats reuse the beat template.
- **Save-State & Persistence** (hard) — all narrative writes land in `ch` fields; schema owned there.
- **Heat/Rep/Turf Reactivity** (soft) — consumes ranks for tiers, writes nothing directly.

## Tuning Knobs

- **pools_per_district** (3 street + 3 hub max): more = writing debt with no gameplay return.
- **banter_exchanges_cap** (3): longer = cutscene creep inside a gig loop.
- **speaker_turn_lines** (6 max): more breaks message-window rhythm.
- **rep_tier_thresholds** (provisional 0–4 mapping): owned by the Heat/Rep GDD — listed here as consumed values, not source of truth.
- **choices_per_beat** (LOCKED at 2): not a knob — raising it breaks the beat template and Pillar 4.

## Visual/Audio Requirements

- Facesets expressive-anime per the art bible (strong brows, sharp highlights); sprites stay functional.
- Message window deck-OS skinned: clipped corners, thin accent rule, district accent on the speaker name; speaker turns timed to the 120–180ms decrypt-in.
- Audio: district bed ducks 30% under dialogue; choice-select blip + consequence sting from the shared synth kit; no voice.

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:dialogue-events` for faceset/portrait specs and generation prompts.

## UI Requirements

- Show Choices skinned as terminal options (max 2, full-width rows, d-pad navigable).
- Bark indicators (icon + accent glow) mark tiered NPCs.

> **📌 UX Flag — Dialogue & Events**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover dialogue, choice, and bark-indicator UI **before** writing epics.

## Acceptance Criteria

- **GIVEN** any story beat, **WHEN** played, **THEN** it follows hook→choice→consequence→write with no console errors.
- **GIVEN** any choice, **WHEN** selected, **THEN** the corresponding meter/flag changes observably.
- **GIVEN** a gig debrief, **WHEN** the gig was ghosted, **THEN** at least one ghost-acknowledging line plays (and loud/mixed equivalents for those approaches).
- **GIVEN** rep rank 4, **WHEN** talking to a street NPC, **THEN** legend-pool lines play.
- **GIVEN** a Played beat, **WHEN** re-triggered, **THEN** recap line only, history unchanged.
- **GIVEN** 18 bark pools, **WHEN** counted, **THEN** none exceed 3 tiers and every pool resolves a tier without errors.

## Open Questions

- Font choice for bitmap text rendering at 26px+ — owner: art-bible/art pass.
- Name-box vs. inline speaker tags — owner: prototype.
- Total line-count budget for the 3–4h script — owner: writing pass (Pillar 4 cap TBD).
