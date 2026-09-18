# Art Bible: CHROME HEART CITY

*Created: 2026-09-18 · Status: Complete (Sections 9/9)*
*Foundation: Visual Identity Anchor "Rain-Slick Neon Noir" (game-concept.md) · Engine: RPG Maker MZ 1.1.1*

> **Art Director Sign-Off (AD-ART-BIBLE)**: SKIPPED — lean review mode (not a phase gate), 2026-09-18

---

## 1. Visual Identity Statement

**One-line rule:** *Every screen looks like it just rained on neon.*

### Supporting principles

1. **Light is information** (serves *Ghost or Loud* — Your Call)
   Patrol sightlines, hackable objects, and objectives glow; everything else falls into shadow.
   *Design test: when visual richness and readability conflict, cut richness.*

2. **One accent per district** (serves *The City Remembers*)
   Sump = acid green, Market = magenta, Spire = ice cyan — district identity readable in a single glance.
   *Design test: when a map wants two accents, split it — it's two maps.*

3. **Grime grounds the glow** (serves the street-legend fantasy and *Short, Dense, Finished*)
   Industrial decay in every frame keeps the neon hungry, never showroom.
   *Design test: when a screen looks clean, add decay before adding light.*

---

## 2. Mood & Atmosphere

| State | Emotion target | Lighting character | Adjectives | Energy |
|---|---|---|---|---|
| Street exploration | Curious but watched | Cool ambient wash + district accent, mid contrast | wet, humming, watchful, layered | measured |
| Gig infiltration | Tense focus | Overall dim; sightlines + hackables glow (light-is-information in action) | held-breath, sharp, narrow, electric | coiled |
| Breach combat | Controlled chaos | High contrast, accent flares on hack triggers | strobing, kinetic, decisive | frenetic-but-readable |
| Payout / victory | Earned exhale | Amber wash (safety color) over the scene | warm, earned, brief | release |
| Busted / defeat | Cold cost | Desaturated, thin red alert edge | stark, costly, cold | abrupt stillness |
| Fixer hub (safe) | Belonging | Warm amber practicals, low contrast | worn, warm, conspiratorial | contemplative |
| Title / menus | Invitation | Black rain + single cycling accent, minimal | sleek, promising, minimal | poised |

---

## 3. Shape Language

- **Character silhouettes:** readable at 48px sprite size — one exaggerated trait per archetype (fixer = long-coat triangle; corpsec = blocky visored mass; ganger = spikes and asymmetry). Sprites carry *role*; facesets carry *identity*. You know who's dangerous before a single line of dialogue.
- **Environment geometry:** verticals dominate (tower walls, rain streaks, hanging cables) = oppression pressing down; neon horizontals (signage, awnings, light bars) = opportunity and escape routes. Curves are rationed — reserved for safe zones (hub warmth) against the city's angles. The city looms; the glow offers a way through.
- **UI shape grammar:** sharp clipped corners + thin neon rules — menus feel like deck interfaces, part of the city rather than pasted on top. Circular motifs are reserved for heat/rep/turf meters: the all-seeing city eye. Even the menus are watching you.
- **Hero vs. supporting:** crew members and interactables get accent-color rims and unique silhouettes; crowds and props are desaturated repeats that recede. The eye goes to choices, never to clutter.

---

## 4. Color System

**Primary palette (role, not just hex):**

| Color | Role |
|---|---|
| Abyss Blue `#0A0E1A` | Shadow base — every map starts here |
| Wet Asphalt `#232733` | Ground, walls, the city's body |
| Sump Acid `#A6FF3F` | Sump district accent — toxic opportunity |
| Market Magenta `#FF2E88` | Market district accent — flesh-trade allure |
| Spire Cyan `#46E6FF` | Spire district accent — corporate ice |
| Hub Amber `#FFB347` | Safety, crew, payout — warmth is earned |
| Alert Red `#FF3131` | Busted/fail states ONLY — never decoration |

**Semantic vocabulary:** accent glow = interactive/hackable · amber = safe · red = failure/heat-critical (never decorative) · desaturation = the past, flashbacks, dead zones.

**Per-district temperature:** Sump runs sickly warm-green over cold blue shadow; Market runs hot magenta against deep blue; Spire runs ice-cyan, highest contrast, coldest shadows. Hubs in every district break to amber.

**UI palette:** near-black panels (`#0A0E1A` at 85% opacity), thin accent rules, white primary text, accent only for the active district + amber for confirm/safe actions. Diverges from the world palette by restraint: UI never uses more than one accent at a time.

**Colorblind safety:** red/green (Alert vs. Sump Acid) is the danger pair — every red state also carries an icon + shape change (circular meter spikes) + audio sting. Heat level is always numeric as well as colored. Never encode information in hue alone.

---

## 5. Character Design Direction

- **Player crew:** 3–4 members max (Pillar 4). Each reads at 48px by silhouette first (coat triangle / visor block / spike asymmetry); identity is carried by facesets. One accent-colored signature item each (scarf, optic, jacket stripe) in their home district's hue.
- **Tell-apart rules:** allies = amber rim + visible face; corpsec = visored (no face) + Spire cyan lights; gangers = asymmetry + district accent; civilians = desaturated, faceless at sprite scale. Threat assessment without dialogue.
- **Expression/pose:** facesets expressive-anime (strong brows, sharp highlights); sprites functional — 4-frame readability over fluidity. Exaggerate hack/tech poses (deck raised, optic flare) as the game's signature gesture.
- **LOD philosophy:** the MZ camera is fixed — design FOR 48px, not down to it. Detail lives in facesets, busts, and menu portraits; sprites stay bold and simple.

---

## 6. Environment Design Language

- **Architecture:** vertical arcology sprawl — towers crowd every exterior; interiors are converted industrial (server rooms in bathhouses, clinics in parking structures). Culture reads through reuse: the poor live inside the rich's discarded infrastructure.
- **Texture philosophy:** stylized-painted over PBR — RTP-compatible tileset edits + recolors, mood carried by lighting overlays rather than texture fidelity. Solo + AI-assisted pipeline can't sustain custom PBR; cohesion beats realism.
- **Prop density:** dense-but-legible streets (clutter frames paths, never blocks sightline readability); infiltration interiors sparse by design (negative space = tension). Density rule: if a prop doesn't frame, hide, or pay off — delete it (Pillar 4).
- **Environmental storytelling:** faction control shown, not told — posters change, accent lighting shifts, patrol density visibly rises as turf moves. Every district carries one "before/after" set-piece per faction state.

---

## 7. UI/HUD Visual Direction

**Decision:** full diegetic deck-OS interface (art wins over convention). Every menu is the player character's deck: boot sequence on title, terminal panes for items/skills, circular city-eye meters for heat/rep/turf, gig board as a bounty-feed UI. The Hack battle command is a spatial terminal interaction, not another menu row.

- **Typography:** condensed grotesque headers (eurostile-adjacent), clean sans body, tabular numerals on all meters. Monospace accents for terminal readouts only.
- **Iconography:** outlined neon-glyph style, one accent color at a time; alert states add icon + shape change (never hue alone).
- **Animation:** fast terminal snaps (120–180ms); data "decrypts" into place on open. No slow fades during combat.
- **Constraints (non-negotiable):** full keyboard navigation and d-pad support (per tech prefs); cancel/confirm rhythm preserved even in custom layouts.
- **Risk + mitigation:** custom UI scenes multiply plugin work — the UI shell is prototyped in the MVP. Pillar 4 escape hatch: if velocity drops, fall back to reskinned MZ default layouts with the Neon Noir surface intact.

---

## 8. Asset Standards

- **Formats:** PNG for all 2D; OGG + M4A pairs for every audio file (MZ requires both for desktop/browser coverage); BGM loop metadata documented per track.
- **Resolution tiers:** tiles 48×48 native (no upscaling — crispness is the style); characters 48×72 cells, 3-frame/4-dir sheets; faces 144×144; menu busts/portraits max 2× faceset scale; parallax overlays at map resolution. Ideal (4K source paintings) vs. standard (author at 2×, ship at 1×) — ship-tier wins: MZ renders 1×, oversized sources only bloat deploy size.
- **Naming:** `CH_<System>_<Name>` for customs (e.g., `CH_Sump_TowerA.png`); MZ prefixes honored (`$` single-sheet, `!` no-shift); no spaces, ASCII only (importer constraint).
- **Lighting/overlays:** one overlay approach (plugin TBD in prototype); overlays capped per map for the 60fps budget; static-gradient fallback if perf fails.
- **Audio budget:** BGM loops <3MB per track (OGG); SFX <200KB; hack/alert SFX share one synth kit for cohesion.
- **Tradeoff log:** 4K sources rejected (deploy bloat, zero on-screen gain) · animated parallaxes capped at 2 per map (perf) · voice acting excluded (scope, Pillar 4).

---

## 9. Reference Direction

1. **Blade Runner 2049** — TAKE: district color-blocking (orange Las Vegas vs. teal LA) as the model for one-accent-per-district, plus volumetric gloom for infiltration lighting. AVOID: cinematic vastness and slowness — our maps stay dense and readable.
2. **Neuromancer (Gibson)** — TAKE: cyberspace-as-place language (ice, constructs, consensual hallucination) for breach-battle framing. AVOID: 80s pastiche — our chrome is 2070s street, not retro.
3. **CP2077 Night City photography** — TAKE: signage layering and vertical framing rules for map composition. AVOID: photoreal density — suggestion over simulation, tiles over geometry.
4. **Synthwave cover art** — TAKE: limited-palette graphic language (grid, chrome type) for deck-OS UI surfaces. AVOID: nostalgia-kitsch purple overload — our accents are district-coded, never decorative.
5. **Jack Move!** — TAKE: proof that tiles + bold UI carry a cyberpunk JRPG at indie scope. AVOID: its cleaner anime look — ours is grimier (grime grounds the glow).
