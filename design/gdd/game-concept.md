# Game Concept: CHROME HEART CITY

*Created: 2026-09-18*
*Status: Draft*
*Engine: RPG Maker MZ 1.1.1 · Platform: PC (Steam / itch) · Review mode: lean*

---

## Elevator Pitch

> It's a cyberpunk street-crew JRPG where you take gigs across three neon districts in a rain-slick megacity to decide which faction owns it.
>
> Test: a new player understands in 10 seconds — take jobs, ghost or fight, city reacts, pick its future. Pass.

---

## Core Identity

| Aspect | Detail |
| ---- | ---- |
| **Genre** | Story-driven JRPG (top-down 2D) with stealth-infiltration gigs and turn-based breach combat |
| **Platform** | PC (Steam / itch) — MZ native deploy; Web export possible later |
| **Target Audience** | See Target Player Profile below (Storytellers + Explorers, 18–35) |
| **Player Count** | Single-player |
| **Session Length** | 45–90 min sessions; full game 3–4 hours |
| **Monetization** | Premium (small paid indie) or free portfolio release — undecided |
| **Estimated Scope** | Medium (4–6 months, solo, AI-assisted) |
| **Comparable Titles** | Jack Move! (cyberpunk JRPG lane) · Persona 5 palaces (infiltration missions) · GTA (heat/rep open-zone freedom) |

---

## Core Fantasy

You are the ghost the megacorps can't catch and the streets can't stop whispering about — a fixer who starts with nothing but a cracked deck and ends up holding the city's future in their hands.

The emotional promise: **street legend**. Every gig taken quietly or loudly, every camera turned, every faction played against the other — the city visibly bends around your reputation until three endings ask who you really built all that chrome for.

---

## Unique Hook

Like a classic JRPG party adventure, AND ALSO every battlefield is hackable terrain — cameras, turrets, and doors become your weapons through a Hack command, and every gig visibly moves heat, rep, and turf on a living district map.

One sentence: **steal the gig, hack the room, watch the whole district react.**

---

## Player Experience Analysis (MDA Framework)

### Target Aesthetics (What the player FEELS)

| Aesthetic | Priority | How We Deliver It |
| ---- | ---- | ---- |
| **Narrative** (drama, story arc) | 1 | Fixer-crew storyline, 3 faction endings, district story beats |
| **Discovery** (exploration, secrets) | 2 | 3 dense districts, hidden gigs, secrets, recontextualizing faction reveals |
| **Fantasy** (make-believe, role-playing) | 3 | Netrunner power trip, chrome upgrades, street-legend identity |
| **Challenge** (obstacle course, mastery) | 4 | Ghost ratings, heat management, ghost-or-loud mastery curve |
| **Expression** (self-expression) | 5 | Approach choice (ghost/loud), faction allegiance, implant picks |
| **Sensation** (sensory pleasure) | 6 | Rain-Slick Neon Noir visuals, synth audio, hack feedback juice |
| **Fellowship** (social connection) | 7 | Crew banter and fixer relationships (NPC, not multiplayer) |
| **Submission** (relaxation) | N/A | Not a comfort game — tension is the point |

### Key Dynamics (Emergent player behaviors)

- Players scout patrol routes before committing, then improvise when spotted — "planned ghost, loud exit" becomes a personal signature.
- Players chain "one more gig" to hit the next visible rep threshold or cool down heat.
- Players replay districts after faction shifts to see what changed, then argue with themselves over which ending to earn.

### Core Mechanics (Systems we build)

1. **Gig board + district maps** — pick jobs freely across 3 neon districts; event-driven stealth sightlines, patrols, hackable map objects.
2. **Breach combat** — turn-based battles with a Hack command that turns cameras, turrets, and doors against enemies.
3. **Heat / rep / turf meters** — every gig moves visible meters; heat triggers pursuit pressure, rep unlocks fixer tiers and gigs.
4. **Chrome implants (light)** — 3 tiers of implants that unlock new options (new hack targets, new approaches), never just +numbers.
5. **Faction standing → 3 endings** — gigs and story beats shift allegiance; the finale crowns the city's owner.

---

## Player Motivation Profile

### Primary Psychological Needs Served

| Need | How This Game Satisfies It | Strength |
| ---- | ---- | ---- |
| **Autonomy** (freedom, meaningful choice) | Gig choice, ghost-or-loud approach, faction allegiance, 3 endings | Core |
| **Competence** (mastery, skill growth) | Ghost ratings, heat mastery, implant tiers, visible rep ranks | Core |
| **Relatedness** (connection, belonging) | Crew banter, fixer relationships, districts whose NPCs react to your legend | Supporting |

### Player Type Appeal (Bartle Taxonomy)

- [x] **Achievers** (goal completion, collection, progression) — How: ghost ratings per gig, rep ranks, 3 endings to collect, implant tiers.
- [x] **Explorers** (discovery, understanding systems, finding secrets) — How: dense districts with hidden gigs and secrets, hackable-system experimentation, faction-reveal recontextualization.
- [ ] **Socializers** (relationships, cooperation, community) — Single-player; crew relationships are narrative flavor, not systems.
- [ ] **Killers/Competitors** (domination, PvP, leaderboards) — No PvP or leaderboards; explicitly not served.

### Flow State Design

- **Onboarding curve**: first 10 minutes — meet your fixer, take a guided zero-heat gig (scripted ghost success), learn move/hack/fight basics before the board opens.
- **Difficulty scaling**: gig tiers (street → corporate → black-ice) raise patrol density and battle stats; heat adds dynamic pressure on top; ghost ratings grade mastery, never gate progress.
- **Feedback clarity**: ghost rating per gig, heat/rep/turf meters move on-screen after every job, implant unlocks announce new capabilities.
- **Recovery from failure**: spotted = loud path, not game over; defeat = retry from gig start with intel kept (patrol layouts learned); no harsh fail states.

---

## Core Loop

### Moment-to-Moment (30 seconds)

Read patrol sightlines → slip, hack, or engage → cameras flip, turrets turn, alerts spike or stay dark. Expressive problem-solving every few seconds, juiced by alert-state shifts and hack-takeover feedback. MZ delivers this with evented vision cones plus the battle Hack command.

### Short-Term (5-15 minutes)

The gig cycle: pick from the board → plan approach → execute (ghost or loud) → heat check → payout and meter movement. "One more gig" psychology from visible rep thresholds and board refreshes.

### Session-Level (30-120 minutes)

A district arc: arrive unknown → build rep → unlock fixer tiers → hit a faction-choice beat. Natural stops at payouts and district milestones; the next faction beat is always the reason to return.

### Long-Term Progression

Rep ranks, 2–3 crew recruits, 3 implant tiers, faction standing → one of 3 faction endings. Done = credits in 3–4 hours. Post-ending: replay for other endings (short game = replayable game).

### Retention Hooks

- **Curiosity**: unanswered faction questions, locked districts, hidden gigs, "what does the other ending change?"
- **Investment**: rep ranks earned, crew recruited, legend built — progress worth finishing.
- **Social**: none (single-player by design).
- **Mastery**: ghost ratings to perfect, heat to dance with, loud-vs-ghost self-challenges.

---

## Game Pillars

### Pillar 1: Ghost or Loud — Your Call

Every gig must be completable via stealth or combat; the player's approach signature is sacred.

*Design test*: If we're debating a map that forces one approach, this pillar says we redesign it.

### Pillar 2: The City Remembers

Every gig visibly moves heat, rep, or turf — no flavor-only rewards.

*Design test*: If we're debating a reward that moves no meter, this pillar says we tie it to one or cut it.

### Pillar 3: Chrome Is Character

Implants unlock new options (new hack targets, new approaches), never just numbers.

*Design test*: If we're debating an implant that's only +damage, this pillar says we rework it into a capability.

### Pillar 4: Short, Dense, Finished

3–4 hours, zero filler — a complete game that ships.

*Design test*: If we're debating a map that serves no gig, secret, or story beat, this pillar says we delete it.

### Anti-Pillars (What This Game Is NOT)

- **NOT open-world sprawl**: 3 dense districts, not a county — protects Pillar 4 from GTA-scale imitation.
- **NOT grind**: no filler random encounters; every fight is a gig beat — protects the loop's density.
- **NOT spreadsheet builds**: Path-of-Exile depth stays out; implants are few meaningful picks — protects first-build shippability.
- **NOT cinematic set-pieces**: story via tight eventing, not voiced spectacles — protects solo scope.

---

## Visual Identity Anchor

*Seed of the art bible — direction: **Rain-Slick Neon Noir** (selected 2026-09-18).*

- **One-line visual rule**: every screen looks like it just rained on neon — dark streets, wet reflections, one electric accent color per district.
- **Supporting principles**:
  - *Light is information*: patrol sightlines, hackable objects, and objectives glow; everything else falls into shadow. Design test: if the player can't tell what's interactive at a glance, add light, not tutorial text.
  - *One accent per district*: Sump = acid green, Market = magenta, Spire = ice cyan. Design test: if a map needs two accents, it's two maps — split it.
  - *Grime grounds the glow*: industrial textures and clutter keep neon from feeling clean. Design test: if a screen looks like a showroom, add decay.
- **Color philosophy**: near-black blues for shadow, saturated single-hue neon for focus, warm amber reserved for safe zones (fixer hubs, crew scenes) so safety has a color.

---

## Inspiration and References

| Reference | What We Take From It | What We Do Differently | Why It Matters |
| ---- | ---- | ---- | ---- |
| GTA series | Open-zone gig freedom, heat/wanted pressure, living-city feel | 3 dense hand-built districts instead of sprawl; heat is a meter, not a simulation | Validates the "one more job" freedom loop and wanted-pressure fun |
| Path of Exile | Chrome/build power fantasy, loot-progression dopamine | Implants as few capability unlocks, not a passive tree | Validates build-expression hunger; scope kept shippable |
| Cyberpunk 2077 | Neon-street atmosphere, fixer/gig structure, chrome identity | 2D top-down JRPG systems instead of FPS open world | Validates setting hunger and the gig-board structure |
| Jack Move! | Proof that cyberpunk-JRPG on a small scope finds its audience | Our hook is hackable battlefields + reactive districts | Direct market validation for the lane |

**Non-game inspirations**: *Blade Runner* (rain, neon, moral grime) · *Neuromancer* (console cowboys, black ice) · synthwave (audio direction) · Night City photography (composition reference for map framing).

---

## Target Player Profile

| Attribute | Detail |
| ---- | ---- |
| **Age range** | 18–35 |
| **Gaming experience** | Mid-core — comfortable with JRPG combat and stealth basics, not seeking soulslike punishment |
| **Time availability** | 45–90 min sessions; finishes 3–4 hour games (a weekend or two) |
| **Platform preference** | PC (Steam / itch.io discovery) |
| **Current games they play** | Cyberpunk 2077, GTA Online/V, Path of Exile, story JRPGs (Omori, Jack Move!, Persona) |
| **What they're looking for** | A cyberpunk story they can *finish* — neon atmosphere and crew fantasy without a 100-hour commitment |
| **What would turn them away** | Grind, filler maps, build-homework, unfinished episodic structure |

---

## Technical Considerations

| Consideration | Assessment |
| ---- | ---- |
| **Recommended Engine** | RPG Maker MZ 1.1.1 — pinned by project; event system covers stealth/gigs/meters, JS plugins cover Hack command + lighting |
| **Key Technical Challenges** | Patrol vision-cone eventing at scale; lighting overlay performance for Neon Noir look; heat/rep state that's save-safe; Hack command battle integration |
| **Art Style** | 2D top-down, MZ tilesets + edits; Rain-Slick Neon Noir via palette discipline + lighting plugin + rain/weather overlays |
| **Art Pipeline Complexity** | Medium — RTP base with recolors/edits, AI-assisted facesets/sprites, lighting plugin for mood |
| **Audio Needs** | Moderate — synthwave BGM loop set, SFX for hacks/alerts/payouts; no voice |
| **Networking** | None (single-player) |
| **Content Volume** | ~3 districts, ~15 gigs, 2–3 recruits, 3 endings, 3–4 hours gameplay |
| **Procedural Systems** | None — all hand-crafted (density over size, per Pillar 4) |

---

## Risks and Open Questions

### Design Risks

- Stealth devolves into patrol-avoidance tedium — mitigate with short sightlines, always-available loud path, ghost ratings as mastery not gates.
- Full reactivity explodes content — mitigate with meter-driven gating and templated reactions, not bespoke branches per gig.
- Thin mystery/faction writing sinks a story-first game — mitigate with early `/design-review` on narrative beats.

### Technical Risks

- Patrol/eventing complexity across 3 districts may strain MZ event limits — mitigate by templating one patrol system and reusing it.
- Lighting plugin performance on low-end PC — mitigate by testing early in `/prototype`, with a fallback to static overlays.
- Save-compat for heat/rep/turf state across versions — mitigate with `Game_System` aliased defaults per the rpgm-engine-coding checklist.

### Market Risks

- Cyberpunk-JRPG is a proven but small niche — mitigate with tight 3–4 hour scope and itch-first release.
- MZ RTP visual stigma — mitigate with the Neon Noir anchor: palette, lighting, and UI cohesion.

### Scope Risks

- Reactivity × 3 endings multiplies eventing — mitigate with Pillar 4 (delete filler) and meter-driven endings, not content-tripled finales.
- First-build optimism on timelines — mitigate with MVP-first ordering (District 1 proves the pipeline before Districts 2–3).

### Open Questions

- Which lighting plugin (VisuStella? OcRam? community)? — resolve in `/prototype`.
- Patrol vision implementation (event-based cones vs. plugin)? — resolve in `/prototype`.
- Hack command battle scope (which objects, how many per battle)? — resolve in `/design-system` (combat) after prototype.

---

## MVP Definition

**Core hypothesis**: Players find the ghost-or-loud gig loop (infiltrate → hack/fight → heat/rep movement) engaging for 30+ minute sessions.

**Required for MVP**:

1. One district with gig board (4–5 gigs), evented stealth sightlines, and ghost-or-loud completion paths.
2. Breach combat with working Hack command (cameras/turrets/doors as targets).
3. Heat/rep meters that move visibly per gig with at least one consequence (heat pursuit, rep-gated gig).

**Explicitly NOT in MVP** (defer to later):

- Districts 2–3 and faction endings (one district proves the loop).
- Crew recruits beyond the core duo.
- Implant tiers 2–3 (tier 1 proves "chrome as capability").
- Lighting-plugin final look (static overlays suffice for validation).

### Scope Tiers (if budget/time shrinks)

| Tier | Content | Features | Timeline |
| ---- | ---- | ---- | ---- |
| **MVP** | 1 district, 4–5 gigs | Core loop only (stealth, Hack combat, meters) | ~4–6 weeks |
| **Vertical Slice** | District 1 complete: intro + gigs + first faction beat, polished | Core + progression + Neon Noir look | +4 weeks |
| **Alpha** | All 3 districts, all gigs, endings rough | All features, rough edges | +2 months |
| **Full Vision** | 3 districts, ~15 gigs, 3 endings, 3–4 hours | All features, polished | 4–6 months total |

---

## Next Steps

- [x] Concept drafted (`/brainstorm`) — review mode: lean (director gates skipped)
- [ ] Fill in technology stack based on engine choice (`/setup-engine`) — MZ 1.1.1 already pinned; record it
- [ ] Create visual identity specification (`/art-bible`) — required before Technical Setup gate; grows the Visual Identity Anchor above
- [ ] Validate concept completeness (`/design-review design/gdd/game-concept.md`)
- [ ] **Prototype core idea** (`/prototype` stealth-gig loop) — before writing GDDs, validate the concept is worth designing
- [ ] If prototype PROCEEDS: decompose into systems (`/map-systems`), then per-system GDDs (`/design-system`)
- [ ] Build vertical slice in Pre-Production (`/vertical-slice`), validate with playtest (`/playtest-report`)
- [ ] Plan first milestone (`/sprint-plan new`)
