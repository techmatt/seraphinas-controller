# state — Seraphina's Secret

*The project state doc: status, decision log, open questions, plan. Rewritten in full at each checkpoint distillation. At session boot, claude.ai reads this doc first.*

## What this project is

"Seraphina's Secret" — a cozy, no-fail 2D exploration game for Julia (Matt's 4-year-old): Seraphina, a young girl, secretly raises baby dragons, and her dad doesn't know. Learning goals in priority order: interactive engagement, menu/UI navigation, goal-holding, walking/spatial navigation, ambient reading exposure (letter *sounds*, not names). Full design in seraphinas-design.md; how the project runs in seraphinas-workflow.md.

## Status (2026-08-11, checkpoint 3)

One working day, five prompts, all landed: `exterior_rework_v2`, `grass_and_interiors`, `hitbox_alignment`, `world_boundary`, `interior_wall_edging`. Matt approved every era change from screenshots on 2026-08-11 — sole exception, some interior edging is incomplete (future cleanup pass, not blocking).

**Hardware verification is PENDING for everything this era** — A-press doors, the new world, the boundary, the interiors, the golden hair are screenshot-verified only. Matt/Julia have not driven any of it on the pad.

Suite: 26 Playwright tests, 26/26, ~7.5 min single worker; typecheck green. Main repo `C:\Code\seraphinas-secret`: Phaser 3 + Vite + TypeScript; voice pipeline at `tools/voice/` (edge-tts proxy voices → `public/voice/` audio + `manifest.json`). No NPC or quest content yet; all voices are placeholders.

## The world

Two zones, both generated data: `content/world/**` → `tools/world/build.ts` → JSON; map changes are data edits.

**Outside**: 72×50 tiles (3600), 941 sprites, 1360 solid cells. Seraphina's house (enterable), Dad's shed, Joey's house, Scar's house, Secret Cave mouth, plus a village hall, 3 market stalls, a silo, a market square and a green (facades, not enterable; interacting gives a wiggle + burst + two knocks, no text). The Mystic Woods hold the west; a fenced vegetable patch, pond, well and joined dirt roads fill the village. Boundary ring 210/216 solid — the 6 open cells are the named woods gap.

**House**: 40×29, four rooms on ONE scrolling floor plan, no internal transitions, real walls and measured furniture.

**mountainPath** leaves the top road's west end and runs along the cliff foot to the cave: 16 tiles, none below row 14, wood unbroken. Trade-off accepted (Matt, 2026-08-11): the cave is not visible from the bottom of the wood. `cave_chest` keeps its name in the (kept) old clearing — the id is in the voice manifest.

## Infrastructure now in place

- **Modular layout source**: `content/world/outside/*` + `house/*` modules; siblings import only `plan.ts` (keeps the graph acyclic); prefabs are named clusters; `RoadSpec` polylines carry width + `alongRoad`; a perimeter band spec with an explicit gap rect; per-region seeded scatter, so a region edit doesn't churn its neighbors; one composition file; `layout.ts` is 25 lines.
- **Build gates**: nothing solid on roads; `smooth()` bites one-tile peninsulas; `assertWalledIn` runs on zones with `sealed` (a `ZoneLayout` field — Outside yes, house no): two flood-fills from spawn, pass 2 strips invisible collision so only visible obstacles count; `sealed.soft` is the named exemption (the woods gap); every spawn, doorway, prop and landmark must be reachable. Two keep-clear sets, `KEEP_CLEAR` and `KEEP_CLEAR_INLAND`, plus the `BOUNDARY` band.
- **Alignment convention** (hitbox_alignment era): catalog `blocks` are measured rects that may name half tiles; `footing.ts` nudges sprites at placement so solid cells land whole-tile, centered under the visual base; half tiles round UP uniformly; scatter no longer jitters solids. Tools: `world:footings` (image base vs declared blocks) and `world:measure` (+ a minimal `png.ts`). Fixed en route: signposts blocked air not post; plant pots; the trough rect was double-width; the fridge rect was double-width (the old kitchen monolith).
- **Overlay tile layer** (`BuiltMap.overlay`): born for grass variants — now removed, Outside carries 0 overlays and the machinery is dormant pending matching-base variants — and used by the house for the cap beam (74 cells, 8 mitred ends).
- **Interior wall system**: `RoomLayout` derives wall faces from the floor rect; `Interior_Walls.png` measured (14×6: plaster/wood/stone/brick faces + left/right seamed pairs, 5 px seam); a run-position pass runs after openings are cut (a neighbor-same-material test covers run ends, jambs and room joins; 36/204 cells seamed); the cap is the sheet's 3×3 nine-slice beam drawn on the overlay layer over the opaque filler (ground-layer placement fails on shared wall lines). Furniture catalog is ~60 measured images; rugs carry a `flat` flag (depth-sort fix); `rugRound` works over any band; `cushion` was renamed `bookShut` (it is a shut book).
- **Debug**: hold-keyboard-B hitbox overlay (solid cells, prop footprints, origins, player box; `debugHitboxes` hook); `overview(fit)` camera hook. Keyboard stays dev-only; pad B remains the player's cancel.
- `walkToProp` arrival now means "nearest interactable" (flake fix).

## Decision log

- Title & premise: "Seraphina's Secret" (Matt, 2026-08-08).
- Three-part workflow adopted; repos: `seraphinas-secret` (main/code), `seraphinas-controller` (docs), `seraphinas-drive-sync` (Drive-synced exchange, not in git) (Matt, 2026-08-08).
- Stack: web — TypeScript + Phaser 3 + Vite, npm; local browser page, F11 fullscreen, "dad opens it" launch. Chosen for CC iteration speed, including Playwright screenshot audits (Matt, 2026-08-08). Phaser 3 confirmed over Phaser 4 for v1 (Matt, 2026-08-08).
- Voice: all audio pre-generated at content time, never runtime; provider must return word timestamps (Matt, 2026-08-08). Start free with edge-tts "proxy voices"; swap to ElevenLabs later by re-running generation — the game reads only the provider-neutral `public/voice/manifest.json` (Matt, 2026-08-08).
- Proxy voices ear-checked, accepted as placeholders (Matt, 2026-08-08): Seraphina `en-US-AnaNeural` (rate −8%), Dad `en-US-GuyNeural` (rate −10%, pitch −15 Hz), Little Sister `en-US-AnaNeural` (rate +6%, pitch +40 Hz). Letter-sound (phonics) lines: do NOT try to solve sustained unvoiced consonants on the proxy TTS; "Suh"-style approximations are acceptable until ElevenLabs (Matt, 2026-08-08).
- Voice content schema: a line has display `text`, optional spoken `say`, and per-line prosody overrides; the manifest carries every display token (punctuation-only tokens get `start === end` and never highlight); alignment mismatches fail the build (CC, 2026-08-08).
- Audio architecture: one shared AudioContext (`src/audio/context.ts`) for voice and sfx, one unlock; the highlight clock is `ctx.currentTime`, so voice playback bypasses Phaser's sound manager (CC, 2026-08-08).
- Test-shaped code lives only in `src/testHooks.ts`; tests freeze the scene or read high-water marks rather than racing transient effects (CC, 2026-08-08).
- Standing design rules live in the main repo's CLAUDE.md (Matt, 2026-08-08): colored dots never letter labels (ButtonDot.ts renders, CLAUDE.md is authority), all text speaks with per-word highlight, no fail states. Prompts no longer restate them.
- "No fail states" is a GAME-DESIGN mechanic — no timers/death/wrong-answer buzzers (Matt, 2026-08-08). Graceful handling of missing assets etc. is ordinary bug-avoidance; never attribute it to the design principle.
- World is big and scrollable, Stardew-style; "static one-screen room" retired (Matt, 2026-08-08). Full world scope laid 2026-08-08 (see design doc).
- One-NPC-per-screen rule dropped entirely (Matt, 2026-08-09).
- Art: full Cute Fantasy RPG pack (Kenmi) purchased (Matt, 2026-08-09). Side-loaded at `C:\Code\seraphinas-assets`; license forbids redistribution → pack files NEVER enter the repo. Gitignored `public/assets/` populated by `assets:sync` (predev/prebuild) from `tools/assets/config.ts` (side-load path + category list in one place); missing side-load fails loudly; attribution in README.
- Player is the pack's paper doll (2026-08-09): base+shoes+pants+shirt+hair layered sheets — an outfit is a layer list, so wardrobe/dress-up/coin-sink is data. Seraphina's blessed look: `Hair_4_Blonde` (golden — Matt, 2026-08-10, replacing `Hair_4_Brown`), `Shirt_1_Pink`, `Pants_1_Blue`, `Shoes_1_Brown` (Matt, 2026-08-09).
- One world scale: `WORLD_SCALE = 4` in src/config.ts is the only pack-pixels→screen-pixels number (CC, 2026-08-10).
- Rendering: per-texture NEAREST + camera roundPixels, NOT global pixelArt — global would jaggy the vector-drawn UI (bubble, dots, glows) (CC, 2026-08-09, reaffirmed 2026-08-10).
- Walk-up interaction juice carried to tileset props (campfire smoke, well voice); vector star retired (CC, 2026-08-10).
- Walk speed 300 px/s (CC, 2026-08-10; Matt's hardware walk accepted the feel).
- Doors: entering is an A-press on the door, exiting is an automatic walk-through portal (Matt, 2026-08-10, supersedes walk-through entry — see design doc).
- Obstacle density targets ~75% of Stardew Valley and occasional 1-tile gaps are fine; the hard ≥2-tile gap rule is retired (Matt, 2026-08-10 — see design doc).
- The map boundary is obstacle-filled and build-gated; grass correctness, the ratified interior aesthetic, and "a small village with a garden+farm" all land in the design doc (Matt, 2026-08-10/11).

## Facts & constraints learned

- Bare `npm install phaser` resolves Phaser 4 — stay pinned `^3` (3.90.0).
- Voice/edge-tts quirk #2: a mid-line "!" yields ~0.93 s of silence — use the say/text split ("Hi! Let's play!" displays, "Hi, let's play!" is spoken). Retires with ElevenLabs, as does quirk #1 (unvoiced letter sounds).
- edge-tts (`edge-tts-universal@1.4.0`): free, no API key, word boundaries in 100-ns ticks, ~1 s/line, zero failures in ~40 calls; unofficial API, could throttle or break someday — fine for placeholder volumes.
- Phonics on edge-tts: any run of s's is read as the letter NAME, once per letter; SSML is unusable (the package escapes it). Voiced continuants (m, n, l, r, v, z, vowels) sustain fine as "Mmm"-style spellings; unvoiced consonants (s, f, th, h) cannot — author them "Suh"-style and treat as placeholders. "Shhh" works for the *sh* digraph. Full experiment: main repo `scratch/voice_pipeline_spike_phonics.json`.
- Timing: non-initial word boundaries are frame-accurate; first and post-pause words highlight ~0.05–0.14 s early (edge-tts ignores leading silence) — deliberately left, since early highlight is the right direction for a child following along. Clips carry ~1 s trailing silence; the bubble holds on manifest duration instead. `npm run voice:inspect` measures alignment.
- Player sheets: all 9 cols × 56 rows of 64×64 (char art ~16×18 centered); rows 0–2 idle, 3–5 walk, in down/RIGHT/up order; LEFT is a code flip (pack convention). Aseprite files carry NO frame tags but define layer order (horse, tool_under, base, shoes, pants, shirt, hair, accesory, hands, tool_top). Pack readmes are license-only — always measure grids from PNGs.
- NPC sheets are NOT on the player grid (Knights 48×48; Farmer_Bob not 48-divisible) — the NPC prompt must measure each sheet; animation-table reuse is unproven.
- Interior furniture and wall sheets are not on a safely indexable grid — every image is measured individually (why the catalog is ~60 measured images, not an index).
- Movement is substepped ≤16 px per collision step (headless frames reach 250 ms; one-jump-per-frame tunnels or crawls).
- Tall sprites fade to 42% when the player is behind them (she must never vanish behind a canopy).
- Harness (standing constraints): headless runs ~15–29 FPS; NEVER assume a timed key hold moves a predictable distance — `walk()` waits for the idle animation; harness route-plans breadth-first over the exposed collision grid; screenshot tour teleports (`teleport(x,y)` hook); per-test timeout 150 s; `artLoaded` hook guards the silent green-square fallback when a sheet 404s; Phaser does not cull sprites (the 941 exterior sprites are the perf cost).

## Verification ledger

Matt tracks CC-verified vs Matt-verified; Matt-verified = Julia-verified for now.

- **Matt/Julia (hardware):** pad single-press wake+start, greeting audio+highlight, transitions, walking, interactables, animated character + blessed outfit, the tile world's feel, walk speed. All from before this era.
- **Screenshot-verified only (hardware PENDING):** everything from the 2026-08-10/11 era — A-press doors, the 72×50 world and its landmarks, the sealed boundary, the interiors, golden hair.
- **CC-only:** everything else Playwright covers per above.
- The pad path is hand-test-only forever (headless Playwright cannot drive the Gamepad API).

## Open questions

- Quest/world state layer — nothing persists across zone changes today (a poked prop resets); quests need state that survives zone restarts and later sleep — build it with the first quest prompt.
- NPC speech bubbles need speaker anchors. The bed currently speaks a dad-voiced line (props reuse existing manifest audio) — speaker attribution to fix when NPCs land.
- Line length for a 4-year-old: wrap at 620 px, two-line bubbles untested with Julia — observe in play.
- Interior edging is incomplete in places — cleanup pass, not blocking (Matt, 2026-08-11).
- The wood's eastern half is thin now the road is gone — planting-pass candidate.
- Tabletop dressing needs a per-placement depth lift (a candle on a table draws behind the table).
- Accepted as-is: 14 furniture images block 5–12 px below their drawn base; `oakBig2`/`spruceBig2` are shadow-less art variants (palette call open); the overlay doesn't draw doorway trigger rects.
- Measured-but-unused inventory: bathroom kit, interior doors, planters, greenhouse-interior tiles, stairs, kitchen wall-cupboards; sofa/armchair side facings won't cut. Uncataloged exterior sheets: windmill, barn, greenhouse, cobble road.
- Plus the design doc's existing opens (Secret mechanics, dragon care loop, etc.).

## Plan

**Next era: NPC + quest grammar + persistence.** Start with a design conversation — the state store's shape, the first quest template, which family member fronts it — before any prompt.

Then, or interleaved: interior edging cleanup, the woods planting pass, the tabletop depth lift, and world-life animation strips (still undone beyond campfire/water/chest).

## Parked

- ElevenLabs + unvoiced phonics.
- Secret Cave interior (with dragons); facades becoming enterable.
