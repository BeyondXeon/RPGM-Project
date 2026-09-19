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

1. The board is evented (hub NPC + Show Choices feed; redacted rows render as disabled choices) and shows only district + rep-eligible gigs. Tier ladder street → corporate → black-ice with rep gates: street rank 0+, corporate rank 2+, black-ice rank 3+. Main-story gigs are exempt from rep gating (always listed when their district is unlocked); optional gigs obey the tier gates — the roster pass tags every gig main/optional. Feed refreshes on accept, on district transfer, and on rep-rank change.
2. Every gig follows hook → approach → twist → payout; both ghost and loud paths are valid; payout = credits + meter payloads + allegiance tag.
3. Gig records use the canonical shape `ch.gigs[id] = {status, approach, retries}`: available → active → resolved (approach ∈ ghost|loud|mixed|messy) | failed → one retry → resolved messy (half payout, heat full). Abandon returns to available with retries PRESERVED (a consumed retry stays consumed — no infinite clean retries). Active gigs may be abandoned freely.
4. One-shots only: ~15 hand-built gigs (5/5/5 per district), MVP 4–5 in the Sump.
5. Resolution (owned by the resolution event, never the debrief — single-writer rule) writes status + approach, fires autosave, emits meter payloads + 1 allegiance tag to the gig's listed faction (wardens/chrome/ghosts) on ANY resolved approach including messy; failed/abandoned/active gigs emit zero tags. Debriefs read approach only; messy plays the mixed line per Dialogue.
6. The system CANNOT: ship gigs without both approaches · add filler/radiant gigs · allow board browsing mid-infiltration (hubs + streets only).

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Available | On the board, eligible | → Active on accept |
| Active | Accepted, in progress | → Resolved on completion · → Failed on defeat · → Available on abandon (retries preserved) |
| Resolved | Completed; approach ∈ ghost/loud/mixed/messy | Terminal |
| Failed | Defeated, retry armed (if retries left) | → Active on retry (once) · → Available on abandon (retry stays consumed) |

### Interactions with Other Systems

- **District Maps** (in: districtId/turf filter; out: `ch.flags["gig_<id>_done"]` per completion + `districts[target].unlocked=true` for transfer gigs; plot transfers stay owned by story-beat flags, which win on conflict).
- **Save-State** (in: canonical `gigs[id] = {status, approach, retries}` schema; out: status writes + autosave trigger).
- **Dialogue & Events** (briefings/debriefings per the beat template; debrief reads approach, messy plays mixed).
- **Stealth & Patrol** (in: approach recorded at resolution).
- **Breach Combat + Hack** (in: battle resolution feeds gig outcome).
- **Heat/Rep/Turf** (out: meter payloads per gig).
- **Faction & Endings** (out: allegiance tags per gig).
- **Onboarding** (first gig is a guided variant: scripted-likely ghost, never forced — loud resolves normally under the suppress-all payload override).
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
- **If a retry is used and the player then abandons**: abandon resets to available with the consumed retry STAYING consumed — the exploit is closed; the remaining path is a fresh accept at reduced margin, not a free reset.
- **If two gigs target the same interior map**: interiors are instanced per gig accept — no shared-state collisions.
- **If payout is computed with a missing tier** (data error): default to street base 100 + console warning (content bug, caught in QA).
- **If approach is unrecorded at resolution** (script gap): default to mixed, never block payout.
- **If the player completes gigs out of tier order**: allowed when rep-eligible — tiers gate by rep, never by checklist.

## Dependencies

**Upstream:**
- **District Maps & Exploration** (hard) — districtId/turf filter; writes `ch.flags["gig_<id>_done"]` + `districts[target].unlocked` for transfer gigs; respects the 2–4 interiors per district cap.
- **Save-State & Persistence** (hard) — canonical gig record schema + autosave trigger; respects the plain-data rule.
- **Dialogue & Narrative Events** (hard) — briefing/debriefing beat template; messy plays mixed.

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
- **meter payload magnitudes** (final per Heat/Rep loud-tax rebalance; mirrored here, owned there).
- **retry_policy** (one retry, preserved across abandon): unlimited retries erase failure stakes; zero retries strand players on hard gigs; abandon-resets-retry enables infinite clean retries — hence preservation.

## Visual/Audio Requirements

- Board feed as bounty-terminal rows: fixer avatar glyph, neutral tier tabs (district accent reserved for the active district per the art bible — tiers never steal accent meaning), heat-risk pips in Alert Red with icon backup.
- Acceptance animation: row decrypts + stamps GHOST/LOUD/MIXED/MESSY in accent (MESSY in Alert Red + icon per art-bible fail-state semantics).
- Audio: row-select blip, accept thunk, payout chime from the shared synth kit (mutable in settings).

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:gig-board` for feed glyph and stamp specs.

## UI Requirements

- Feed columns: title / fixer / payout / heat-risk / district; district filter auto-applied.
- Locked gigs show as redacted rows (tease without content).

> **📌 UX Flag — Gig Board**: This system has UI requirements. In Phase 4 (Pre-Production), run `/ux-design` to cover the gig board screen **before** writing epics.

## Acceptance Criteria

- **GIVEN** the board, **WHEN** opened in an unlocked district, **THEN** only eligible gigs list with title, fixer, payout, heat risk.
- **GIVEN** a gig accepted, **WHEN** checked in `ch.gigs`, **THEN** record reads `{status: 'active', approach: null, retries: <unchanged>}`.
- **GIVEN** a gig resolved ghost, **WHEN** checked, **THEN** record reads `{status: 'resolved', approach: 'ghost'}`, autosave written, Heat/Rep ghost payload applied.
- **GIVEN** defeat, **WHEN** it happens, **THEN** record reads `{status: 'failed'}` with retry armed iff retries consumed < 1.
- **GIVEN** retry failed, **WHEN** resolved, **THEN** record reads `{status: 'resolved', approach: 'messy'}`, half payout, Heat/Rep messy payload applied.
- **GIVEN** 15 gigs, **WHEN** counted, **THEN** every one offers ghost and loud paths.
- **GIVEN** an active gig, **WHEN** abandoned, **THEN** status returns to available with retries preserved (consumed stays consumed).

## Open Questions

- Exact 15-gig roster titles — owner: writing pass.
- Implant/gear sink totals to match the ~3,750 lifetime faucet — owner: Chrome Implants GDD.
- Final meter magnitudes — owner: Heat/Rep GDD.
