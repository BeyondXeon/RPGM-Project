# Chrome Implants

> **Status**: In Design
> **Author**: BeyondXeon + studio session
> **Last Updated**: 2026-09-19
> **Implements Pillar**: Chrome Is Character · Short, Dense, Finished

## Overview

Chrome Implants is the progression of CHROME HEART CITY: 3 slots (Optic / Frame / Deck) × 3 tiers of capability unlocks — new Hack targets, new infiltration options, new approach vectors — never +numbers. Each tier is a playstyle sentence, bought with gig credits at ripperdocs and stored in `ch.implants{slotId: tier}`; Stealth and Breach read the flags at runtime. Full completion is deliberately unattainable on one run's economy, so every install is a statement. Player-facing, chrome is character made visible — your build is your reputation with circuitry showing.

## Player Fantasy

You're under construction — by choice. The fantasy is self-authorship in chrome: the ghost who sees farther because she bought the eyes, the breacher whose deck turns turrets like party tricks, the frame-runner who ghosts routes nobody else can walk. Every install closes some doors and kicks others open, and by the finale your silhouette plays nothing like anyone else's. (Serves *Chrome Is Character* at its purest — and the PoE hunger for build identity, distilled to nine meaningful picks instead of a thousand passive nodes.)

## Detailed Design

### Core Rules

1. Three slots — Optic / Frame / Deck — each with 3 tiers. One install per slot per tier (replacement, never stacking); tiers gate sequentially (T2 requires T1 in that slot).
2. The roster (all capabilities, zero +numbers):
   - OPTIC T1 — Wall-sight: patrol cone overlays render through walls within 2 tiles (planning intel).
   - OPTIC T2 — Dead-zone nerves: R10–R19 restricted tiles no longer double suspicion build.
   - OPTIC T3 — Flatline: one free Alert-break per map (first Alert auto de-escalates to Suspicious, no pursuit).
   - FRAME T1 — Vent access: marked vent shortcuts traversable (map-side secret routes open).
   - FRAME T2 — Pursuit parity: pursuit speed bonus negated (outrun at equal speed, escapes become winnable).
   - FRAME T3 — Ghostwalk: restricted zones behave as normal tiles (immunity to the double-rate rule).
   - DECK T1 — Skim: Camera Hack costs 10 TP (20→10).
   - DECK T2 — Deep scan: Door targets show wave intel pre-battle (armed/empty + wave size).
   - DECK T3 — Black-ice suite: Turret overload splashes the adjacent enemy + ghost-tier preemptive −10% (reserved hook, now defined).
3. Pricing per slot: T1 300 · T2 800 · T3 1500. Full 9-install cost 7,800 vs ~3,750 lifetime faucet — roughly half affordable per run; every install is a statement. Ripperdoc vendors in each hub sell that district's tier range (Sump T1, Market T1–T2, Spire T2–T3).
4. Tier 1 available in Vertical Slice (Optic T1 + Deck T1); tiers 2–3 unlock with Market/Spire.
5. Implants write `ch.implants{slotId: tier}`; Stealth reads optic/frame flags at detection time; Breach reads deck flags at Hack resolution. No implant alters damage, HP, or credits directly — capability law enforced by schema (flags only, no stat fields exist to modify).
6. The system CANNOT: stack installs · skip tiers · sell +number implants · exceed one free Alert-break per map · grant Turret multi-target beyond adjacent splash.

### States and Transitions

| State | Meaning | Transitions |
|---|---|---|
| Slot: empty → T1 → T2 → T3 | Per-slot progression | Purchase at ripperdoc with credits; sequential only |
| Flatline: armed → spent | Per-map free break | Resets on map entry |
| Build: any 3–5 installs | Typical endgame state | Full 9 mathematically out of reach (by design) |

### Interactions with Other Systems

- **Stealth & Patrol** (out: optic/frame flags — wall-sight, dead-zone, flatline, ghostwalk, parity).
- **Breach Combat** (out: deck flags — skim discount, deep-scan intel, black-ice splash + preemptive hook).
- **Gig Board** (in: credits faucet ~3,750 funds ~half the roster).
- **District Maps** (in: vent shortcut routes + ripperdoc vendor placement per hub).
- **Save-State** (`ch.implants{slotId: tier}` round-trips; install purchases are save-safe events).
- **Deck-OS UI** (implant screen: 3 slots × 3 tiers, owned/unowned states, ripperdoc shop UI).

## Formulas

The `build_cost` totals are defined as:

`slot_full = 300 + 800 + 1500 = 2600 · roster_full = 3 × 2600 = 7800`

**Variables:**

| Variable | Symbol | Type | Range | Description |
|---|---|---|---|---|
| Tier prices | prices | int | 300/800/1500 | Per slot per tier, sequential purchase |
| Lifetime faucet | faucet | int | ~3,750 | 15 gigs averaging ~250 (Gig Board) |
| Affordable share | share | float | ~0.48 | 3750 ÷ 7800 — roughly half the roster per run |

**Output Range:** a full playthrough funds 4–5 installs (e.g., one slot to T3 + one to T2, or three slots to T1 + one T2).
**Example:** ghost-leaning player: Optic T1+T2 (1100) + Deck T1+T2 (1100) + Frame T1 (300) = 2500 of ~3750 — Frame T3 vs Deck T3 is the endgame dilemma, never both.

## Edge Cases

- **If the player can afford nothing they want**: T1 at 300 is reachable after 1–2 street gigs — the first install always lands in Sump (progression starts immediately, never gated).
- **If the player buys cross-district** (Sump credits for Spire chrome): allowed — credits are global; tiers gate by slot sequence, never by district (vendor ranges are convenience, not locks).
- **If Flatline triggers while already Calm**: no-op, stays armed (only consumes on an actual Alert-break).
- **If a save predates an install** (bought, then old save loaded): `ch.implants` round-trips exactly — installs are save data, never session state.
- **If vent shortcut routes are used without the Frame T1 install**: blocked with a deck message ("sealed — Frame T1 required") — capabilities gate content, visibly.

## Dependencies

**Upstream:**
- **Breach Combat + Hack** (hard) — deck flags consumed at Hack resolution (skim, deep-scan, black-ice).
- **Stealth & Patrol** (hard) — optic/frame flags consumed at detection time.
- **Gig Board** (hard) — credit faucet funds installs.
- **District Maps** (hard) — vent routes + ripperdoc vendor placement.
- **Save-State** (hard) — `ch.implants{slotId: tier}` schema.

**Downstream:** none — Implants is a leaf consumer of credits and a flag-source for Stealth/Breach (flags flow out, but no system depends on Implants existing).

## Tuning Knobs

- **tier_prices** (300/800/1500): cheaper = completionism (kills meaningful picks); pricier = T3 unreachable (capstone content wasted).
- **faucet coverage** (~48% of full roster): higher coverage = build homogeneity across runs; lower = most players see 3 installs (thin progression fantasy).
- **skim_discount** (Camera 20→10): deeper = Camera always hacked first (degenerate default); shallower = Deck T1 a dead pick.
- **flatline_scope** (once per map): per-gig refill = ghost insurance (too safe); per-run = hoarded never used (too precious).

## Visual/Audio Requirements

- Installed chrome visible on crew sprites (optic glow, frame plating seams, deck rig) — silhouette changes per slot owned, tier shown by glow intensity.
- Install sequence: ripperdoc chair scene, accent-wash flash, capability unlock sting (distinct per slot family).
- Ripperdoc shops in hub-amber clinical light (safe + surgical, not back-alley).

📌 **Asset Spec** — Visual/Audio requirements are defined. Run `/asset-spec system:chrome-implants` for chrome-variant sprite and shop specs.

## UI Requirements

- Implant screen: 3 slots × 3 tiers grid, owned/unowned/locked states, capability text per install (never stat text).
- Ripperdoc shop UI reuses the grid with prices + district tier range.

> **📌 UX Flag — Chrome Implants**: covered under the Deck-OS grid specs (`/ux-design`, Phase 4).

## Acceptance Criteria

- **GIVEN** 300 credits, **WHEN** buying Optic T1, **THEN** cone overlays render through walls within 2 tiles and `ch.implants.optic === 1`.
- **GIVEN** Deck T1 owned, **WHEN** the Hack menu opens on a Camera, **THEN** cost reads 10 TP.
- **GIVEN** a full serial playthrough, **WHEN** totaled, **THEN** affordable installs ≤ 6 of 9 (half-build economics hold).
- **GIVEN** Frame T1 unowned, **WHEN** entering a vent route, **THEN** blocked with the sealed message.
- **GIVEN** Flatline armed, **WHEN** first Alert fires, **THEN** auto de-escalation to Suspicious with no pursuit; second Alert pursues normally.
- **GIVEN** any install, **WHEN** inspected, **THEN** zero stat modifications exist (capability law — flags only).

## Open Questions

- Vent shortcut route placement per district (map-side content, needs mapper pass) — owner: District Maps content pass.
- Ripperdoc vendor personalities per hub (one character or three?) — owner: writing pass.
- Respec option (refund/replace installs?) — default NO (choices matter); revisit if playtests show regret spirals.
