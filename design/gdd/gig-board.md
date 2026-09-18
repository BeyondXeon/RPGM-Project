# Gig Board & Missions

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-18
> **Implements Pillar**: Ghost or Loud — Your Call · The City Remembers · Short, Dense, Finished

## Overview

Gig Board & Missions is the job system of CHROME HEART CITY: a district-filtered bounty feed offering ~15 templated gigs (hook → approach → twist → payout) across Sump, Market, and Spire, each carrying meter payloads, allegiance tags, and a ghost-or-loud completion path. Gig states live in `ch.gigs` (available → active → resolved with approach recorded); completion fires autosave and moves heat, rep, and turf. Player-facing, the board is freedom with a pulse — you pick the job, the approach, and the faction it serves, and the feed refreshes with work that remembers what you did last.

## Player Fantasy

You're the hottest freelancer on the feed. The fantasy is chosen work, chosen way: scrolling bounties that fit your mood, picking the quiet infiltration or the loud door-kick, and watching the payout move numbers that matter. Every gig is a small story where you're the protagonist, the method, and the legend that follows. (Serves *Ghost or Loud* — the board never prescribes approach — and the street-legend core fantasy: freelancers with a signature get remembered.)

## Detailed Design

### Core Rules

1. The board shows only district + rep-eligible gigs; tier ladder runs street → corporate → black-ice.
2. Every gig follows hook → approach → twist → payout; both ghost and loud paths are valid; payout = credits + meter payloads + allegiance tag.
3. Statuses: available → active → resolved(ghost|loud|mixed) | failed → one retry → resolved(messy, reduced payout, heat+). Active gigs may be abandoned back to available with no penalty.
4. One-shots only: ~15 hand-built gigs (5/5/5 per district), MVP 4–5 in the Sump.
5. Resolution writes status + approach, fires autosave, and emits meter payloads.
6. The system CANNOT: ship gigs without both approaches · add filler/radiant gigs · allow board browsing mid-infiltration (hubs + streets only).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Available | On the board, eligible | → Active on accept |
| Active | Accepted, in progress | → Resolved on completion · → Failed on defeat · → Available on abandon |
| Resolved:ghost/loud/mixed | Completed with approach recorded | Terminal |
| Failed | Defeated, retry armed | → Active on retry (once) |
| Resolved:messy | Completed after failed retry, reduced payout, heat+ | Terminal |

### Interactions with Other Systems

- **District Maps** (in: districtId/turf filter; out: completion flags → transfers/secrets).
- **Save-State** (in: `gigs:{id:status}` schema; out: status writes + autosave trigger).
- **Dialogue & Events** (briefings/debriefings per the beat template, approach acknowledged).
- **Stealth & Patrol** (in: approach recorded at resolution).
- **Breach Combat + Hack** (in: battle resolution feeds gig outcome).
- **Heat/Rep/Turf** (out: meter payloads per gig).
- **Faction & Endings** (out: allegiance tags per gig).
- **Onboarding** (first gig is a guided variant with scripted ghost success).
- **Deck-OS UI** (out: feed data — title, fixer, payout, heat risk, district).

## Formulas

The `gig_payout` formula is defined as:

`gig_payout = base_credits[tier] × messy_modifier`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Tier base | base | int | 100–500 | street 100 · corporate 250 · black-ice 500 (credits, approach-neutral by pillar law) |
| Messy modifier | messy | float | 0.5 or 1.0 | 1.0 clean resolve (ghost/loud/mixed) · 0.5 messy resolve after failed retry |

**Output Range:** 50 to 500 credits per gig; lifetime gig income ≈ 15 gigs averaging ~250 = ~3,750 credits (no dedicated economy system — credits spend on implants/gear defined in their GDDs; sinks must match this faucet).
**Example:** corporate gig resolved loud: 250 × 1.0 = 250 credits + loud meter payload (heat+3, rep+1); same gig messy: 125 credits + heat+3.

Approach meter payloads (per gig, applied alongside credits): ghost → rep+2, heat+0 · loud → rep+1, heat+3 · mixed → rep+1, heat+1 · messy → rep+0, heat+3. Final per Heat/Rep GDD (2026-09-19 loud-tax rebalance).

## Edge Cases

- **If a gig is accepted and its district then locks** (sequence break): the active gig remains completable; board filters apply to new accepts only.
- **If a retry is used and the player then abandons**: abandon resets to available with the retry re-armed — no double-punish.
- **If two gigs target the same interior map**: interiors are instanced per gig accept — no shared-state collisions.
- **If payout is computed with a missing tier** (data error): default to street base 100 + console warning (content bug, caught in QA).
- **If approach is unrecorded at resolution** (script gap): default to mixed, never block payout.
- **If the player completes gigs out of tier order**: allowed when rep-eligible — tiers gate by rep, never by checklist.

## Dependencies

**Upstream:**
- **District Maps & Exploration** (hard) — districtId/turf filter; completion flags → transfers/secrets; respects the 2–4 interiors per district cap.
- **Save-State & Persistence** (hard) — `gigs:{id:status}` schema + autosave trigger; respects the plain-data rule.
- **Dialogue & Narrative Events** (hard) — briefing/debriefing beat template + approach acknowledgement.

**Downstream:**
- **Heat/Rep/Turf Reactivity** (hard) — meter payloads out; magnitudes provisional, finals owned there.
- **Faction & Endings** (hard) — allegiance tags out.
- **Onboarding** (hard) — guided first-gig variant.
- **Deck-OS UI/HUD** (soft) — feed data out (title, fixer, payout, heat risk, district).
- **Stealth & Patrol / Breach Combat** (soft) — approach + resolution in.

## Tuning Knobs

- **gig_count** (15 total, 5/5/5): more = writing + eventing debt against Pillar 4; fewer = thin 3–4h game.
- **tier_bases** (100/250/500): interacts with implant/gear sinks — sinks must match the ~3,750 lifetime faucet.
- **messy_modifier** (0.5): lower punishes experimentation; higher makes failure meaningless.
- **meter payload magnitudes** (provisional — owned by Heat/Rep; listed here as emitted values, not source of truth).
- **retry_policy** (one retry): unlimited retries erase failure stakes; zero retries strand players on hard gigs.

## Visual/Audio Requirements

- Board feed as bounty-terminal rows: fixer avatar glyph, neutral tier tabs (district accent reserved for the active district per the art bible — tiers never steal accent meaning), heat-risk pips in Alert Red with icon backup.
- Acceptance animation: row decrypts + stamps GHOST/LOUD/MESSY in accent.
- Audio: row-select blip, accept thunk, payout chime from the shared synth kit (mutable in settings).

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:gig-board` for feed glyph and stamp specs.

## UI Requirements

- Feed columns: title / fixer / payout / heat-risk / district; district filter auto-applied.
- Locked gigs show as redacted rows (tease without content).

> **📌 UX Flag — Gig Board**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover the gig board screen **before** writing epics.

## Acceptance Criteria

- **GIVEN** the board, **WHEN** opened in an unlocked district, **THEN** only eligible gigs list with title, fixer, payout, heat risk.
- **GIVEN** a gig accepted, **WHEN** checked in `ch.gigs`, **THEN** status reads active.
- **GIVEN** a gig resolved ghost, **WHEN** checked, **THEN** status ghost, autosave written, rep+2/heat+0 applied.
- **GIVEN** defeat, **WHEN** it happens, **THEN** status failed with retry armed.
- **GIVEN** retry failed, **WHEN** resolved, **THEN** messy status, half payout, heat+3.
- **GIVEN** 15 gigs, **WHEN** counted, **THEN** every one offers ghost and loud paths.
- **GIVEN** an active gig, **WHEN** abandoned, **THEN** status returns to available with retry re-armed.

## Open Questions

- Exact 15-gig roster titles — owner: writing pass.
- Implant/gear sink totals to match the ~3,750 lifetime faucet — owner: Chrome Implants GDD.
- Final meter magnitudes — owner: Heat/Rep GDD.
