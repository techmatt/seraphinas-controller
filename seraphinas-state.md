# state — Seraphina's Secret

*The project state doc: status, decision log, open questions, plan. Rewritten in full at each checkpoint distillation. At session boot, claude.ai reads this doc first.*

## What this project is

"Seraphina's Secret" — a cozy, no-fail 2D exploration game for Julia (Matt's 4-year-old): Seraphina, a young girl, secretly raises baby dragons, and her dad doesn't know. Learning goals in priority order: interactive engagement, menu/UI navigation, goal-holding, walking/spatial navigation, ambient reading exposure (letter *sounds*, not names). Full design in seraphinas-design.md; how the project runs in seraphinas-workflow.md.

## Status (2026-08-13, checkpoint 5)

The day-cycle era, COMPLETE and hardware-verified (Matt, 2026-08-13). Three prompts, all landed: `housekeeping_dot_glow_voice`, `sleep_and_reset`, `dusk_and_recap`. Shipped: a day clock that eases into a permanent dusk, a bed that ends the day on the ordinary two-press grammar, a night sequence carrying a spoken bedtime recap, and one function that knows what a night clears. The session store is still session-scoped — disk persistence remains future, so a page reload is still a fresh day.

Suite: 18 Playwright tests, `test:all` 6.7 min single worker, fast subset 13 tests / 3.3 min; typecheck green. Main repo `C:\Code\seraphinas-secret`: Phaser 3 + Vite + TypeScript; voice pipeline at `tools/voice/` (edge-tts proxy voices → `public/voice/` audio + `manifest.json`), 42 lines. All voices are placeholders.

## The world

Three zones. Outside and House are generated data: `content/world/**` → `tools/world/build.ts` → JSON; map changes are data edits.

**Outside**: 72×50 tiles (3600), 941 sprites, 1197 solid cells (down from 1360 — buildings went base-only; the +1 over 1196 is campfire logs). Seraphina's house (enterable), Dad's shed, Joey's house, Scar's house, the Secret Cave mouth, plus a village hall, 3 market stalls, a silo, a market square and a green (facades, not enterable; interacting gives a wiggle + burst + two knocks, no text). The Mystic Woods hold the west; a fenced vegetable patch, pond, well and joined dirt roads fill the village. Boundary ring 210/216 solid — the 6 open cells are the named woods gap. Of 256 trees, 46 are choppable; perimeter/boundary trees never are. 29 sprites now carry `glow` (wall torches, campfire, and the exterior lamp posts) and light at dusk.

**House**: 40×29, four rooms on ONE scrolling floor plan, no internal transitions, real walls and measured furniture.

**Secret Cave**: 20×11, exactly one screen, deliberate. Grey cobble walls + the pack's cavern floor, four wall torches, a campfire and the spell circle — which is a permanent landmark, not quest furniture (Matt, 2026-08-13). The cave mouth is a real A-press doorway with a violet light; exit is a walk-through portal. Nothing dragon-shaped exists in it yet.

**mountainPath** leaves the top road's west end and runs along the cliff foot to the cave: 16 tiles, none below row 14, wood unbroken. Trade-off accepted (Matt, 2026-08-11): the cave is not visible from the bottom of the wood. `cave_chest` keeps its name in the (kept) old clearing — the id is in the voice manifest.

## Infrastructure now in place

- **Modular layout source**: `content/world/outside/*` + `house/*` modules; siblings import only `plan.ts` (keeps the graph acyclic); prefabs are named clusters; `RoadSpec` polylines carry width + `alongRoad`; a perimeter band spec with an explicit gap rect; per-region seeded scatter, so a region edit doesn't churn its neighbors; one composition file; `layout.ts` is 25 lines.
- **Build gates**: nothing solid on roads; `smooth()` bites one-tile peninsulas; `assertWalledIn` runs on zones with `sealed` (a `ZoneLayout` field — Outside yes, house no): two flood-fills from spawn, pass 2 strips invisible collision so only visible obstacles count; `sealed.soft` is the named exemption (the woods gap); every spawn, doorway, prop and landmark must be reachable. The gate **re-runs with every choppable tree's cells removed** — "she can never chop her way out" is build-gated (marking the west wall choppable fails with 138 leaking cells). Two keep-clear sets, `KEEP_CLEAR` and `KEEP_CLEAR_INLAND`, plus the `BOUNDARY` band.
- **Alignment convention** (hitbox_alignment era): catalog `blocks` are measured rects that may name half tiles; `footing.ts` nudges sprites at placement so solid cells land whole-tile, centered under the visual base; half tiles round UP uniformly; scatter no longer jitters solids. Tools: `world:footings` (image base vs declared blocks) and `world:measure` (+ a minimal `png.ts`). Building collision is base-only (~2-tile bands measured off pixels), and depth/`revealBehind` key off foot-of-footprint, not picture bottom.
- **Overlay tile layer** (`BuiltMap.overlay`): born for grass variants — now removed, Outside carries 0 overlays and the machinery is dormant pending matching-base variants — and used by the house for the cap beam (74 cells, 8 mitred ends).
- **Interior room system**: `RoomLayout` derives wall faces from the floor rect; `Interior_Walls.png` measured (14×6: plaster/wood/stone/brick faces + left/right seamed pairs, 5 px seam); a run-position pass runs after openings are cut (36/204 cells seamed); the cap is the sheet's 3×3 nine-slice beam drawn on the overlay layer over the opaque filler. `FLOOR_PATTERNS` grew a `size` for the cavern floor. Furniture catalog is ~60 measured images; rugs carry a `flat` flag; `rugRound` works over any band; `cushion` was renamed `bookShut`.
- **Session state store**: run/world halves; owns quest phase + progress, quest inventory, granted tools, world deltas (broken gems, chopped trees, poked props). Session-scoped (page lifetime), no disk.
- **Sleep** (`src/state/sleep.ts`): `nightPasses()` is the single place that knows what a night clears — it calls `resetForSleep()` and resets the day clock. `resetForSleep()` had three gaps, now fixed (world deltas, so wood regrows; belt slots 2–4; offer counters); it is byte-identical to `reset()` today but kept separate deliberately — coins will be the first sleep-survivor to diverge them. Wake position is `wakeAt` in world pixels via `nearestStanding`. Test hook `sleeps`.
- **Nightfall** (`src/world/nightfall.ts`): ~3.6 s, drawn on its own curtain ABOVE the HUD rather than as a camera fade; the scene restart is hidden at the darkest instant; moon, stars and motes. No Zzz — a Z on screen would be unspoken text, against the all-text-speaks rule. The beat stretches to fit the recap lines (`NightBeat.onSky`): quiet ≈4.4 s, two-event ≈10.9 s.
- **Day clock** (`src/state/dayClock.ts`): ms since wake — ~8 min daylight, ~2 min ramp, then holds at dusk forever (never night, never forced), reset by `nightPasses()`. Test hook `warpDay(ms)` ADDS time.
- **Dusk** (`src/world/dusk.ts`): screen-fixed tint down to a 0.44 cozy floor, exterior only (`OUTDOOR_ZONES=['outside']`); a new `DEPTH.dusk` band sits UNDER every interactive element, so nothing she is asked to press is ever dimmed. `glow` sprites light via a second additive halo (2.1×) that is invisible in daylight; 16 firefly motes outdoors at dusk. `dad_bedtime` is one dad-voiced call anchored at the front door at dusk onset, waiting for a quiet balloon; its once-per-day latch lives on the day clock, not the scene, so dusk arriving in the cave doesn't spend it. SpeechBubble now clamps vertically (his words could land off-canvas).
- **Recap** (`src/state/recap.ts`): the store is snapshotted BEFORE `nightPasses()`; predicates run faeries → on-an-errand → stones → trees, max 2, always closing on `seraphina_goodnight`. A quiet day IS just the goodnight — no separate default line. A "quest completed" predicate is deliberately absent: completion and faeries are the same instant. `QuestEngine.finished` treats any parked phase as finished, so recap is not quest-hardcoded.
- **Quest engine v0**: one active quest max, ordered phases, per-phase voiced instruction; goal kinds `gather` (free-order slot row), `travel` (arriving completes), `ritual` (ordered button presses; a ritual goal now names a ZONE, and the ring's position/radius live in zone data — `RitualSite` is deleted). Offer = thought-bubble marker visible from a distance, then A-press (the second press IS acceptance, no yes/no). Y replays the current phase's line over Seraphina in the giver's voice, from anywhere. Passive sparkle on current objectives; no arrows, no minimap. Quest props live in `src/quest/quests.ts`, not world data — moving one is a quest edit with no rebuild; an e2e test covers their reachability, the build gate does not see them.
- **Interaction reach**: `INTERACT_RADIUS` is 98 px (1.5 tiles, down from 140), and the same constant is the tool-swing reach — unified, and it feels right on hardware. Fallout was fixed properly rather than papered over: title spawn moved 0.7 tile so the dot shows at spawn; bed and well gained `at` standing spots.
- **Tool belt**: 4 slots, empty slots render, axe permanent in slot 1 (`take('axe')` refuses), X cycles skipping empties. Quest-granted tools occupy 2–4 and are revoked at quest completion. The pack draws hands on a separate paper-doll layer the game never loaded; it is loaded now (the axe needed it) and visibly fixes idle and walk everywhere.
- **Choppable trees**: 3 whacks → fall (escalating juice) → stump; 2 more → gone, and the cells become walkable — the first runtime collision-grid mutation (the harness route-planner reads the live grid).
- **Faeries**: drawn vector glow-motes (the pack has nothing), one per gem color, twinkling on three clocks; off-grid leash followers, no collision or pathfinding, cross zones, live in `session.run.faeries` and clear at sleep.
- **Debug**: hold-keyboard-B hitbox overlay (solid cells, prop footprints, origins, player box; `debugHitboxes` hook); `overview(fit)` camera hook. Keyboard stays dev-only; pad B remains the player's cancel.
- `walkToProp` arrival means "nearest interactable" (flake fix).

## Decision log

- Title & premise: "Seraphina's Secret" (Matt, 2026-08-08).
- Three-part workflow adopted; repos: `seraphinas-secret` (main/code), `seraphinas-controller` (docs), `seraphinas-drive-sync` (Drive-synced exchange, not in git) (Matt, 2026-08-08).
- Stack: web — TypeScript + Phaser 3 + Vite, npm; local browser page, F11 fullscreen, "dad opens it" launch. Chosen for CC iteration speed, including Playwright screenshot audits (Matt, 2026-08-08). Phaser 3 confirmed over Phaser 4 for v1 (Matt, 2026-08-08).
- Voice: all audio pre-generated at content time, never runtime; provider must return word timestamps (Matt, 2026-08-08). Start free with edge-tts "proxy voices"; swap to ElevenLabs later by re-running generation — the game reads only the provider-neutral `public/voice/manifest.json` (Matt, 2026-08-08).
- Proxy voices ear-checked, accepted as placeholders: Seraphina `en-US-AnaNeural` (rate −8%), Dad `en-US-GuyNeural` (rate −10%, pitch −15 Hz), Hazel `en-US-AnaNeural` (rate +6%, pitch +40 Hz) (Matt, 2026-08-08), Sneak `en-US-AnaNeural` (rate +4%, pitch −35 Hz) (Matt, 2026-08-12). Letter-sound (phonics) lines: do NOT try to solve sustained unvoiced consonants on the proxy TTS; "Suh"-style approximations are acceptable until ElevenLabs (Matt, 2026-08-08).
- Voice content schema: a line has display `text`, optional spoken `say`, and per-line prosody overrides; the manifest carries every display token (punctuation-only tokens get `start === end` and never highlight); alignment mismatches fail the build (CC, 2026-08-08).
- **Narrative voice** (Matt, 2026-08-13): narrative and prop lines default to Seraphina's voice; the real characters (dad, Sneak, Hazel) keep their own. Four prop lines were re-cut accordingly (well, apple tree, bookshelf, toybox). The rule also lives in the main repo's CLAUDE.md.
- Audio architecture: one shared AudioContext (`src/audio/context.ts`) for voice and sfx, one unlock; the highlight clock is `ctx.currentTime`, so voice playback bypasses Phaser's sound manager (CC, 2026-08-08).
- Test-shaped code lives only in `src/testHooks.ts`; tests freeze the scene or read high-water marks rather than racing transient effects (CC, 2026-08-08).
- Standing design rules live in the main repo's CLAUDE.md (Matt, 2026-08-08): colored dots never letter labels (ButtonDot.ts renders, CLAUDE.md is authority), all text speaks with per-word highlight, no fail states. Prompts no longer restate them.
- "No fail states" is a GAME-DESIGN mechanic — no timers/death/wrong-answer buzzers (Matt, 2026-08-08). Graceful handling of missing assets etc. is ordinary bug-avoidance; never attribute it to the design principle.
- World is big and scrollable, Stardew-style; "static one-screen room" retired (Matt, 2026-08-08). Full world scope laid 2026-08-08 (see design doc).
- One-NPC-per-screen rule dropped entirely (Matt, 2026-08-09).
- Art: full Cute Fantasy RPG pack (Kenmi) purchased (Matt, 2026-08-09). Side-loaded at `C:\Code\seraphinas-assets`; license forbids redistribution → pack files NEVER enter the repo. Gitignored `public/assets/` populated by `assets:sync` (predev/prebuild) from `tools/assets/config.ts`; missing side-load fails loudly; attribution in README.
- Player is the pack's paper doll (2026-08-09): base+shoes+pants+shirt+hair layered sheets — an outfit is a layer list, so wardrobe/dress-up/coin-sink is data. Seraphina's blessed look: `Hair_4_Blonde` (golden — Matt, 2026-08-10), `Shirt_1_Pink`, `Pants_1_Blue`, `Shoes_1_Brown` (Matt, 2026-08-09).
- One world scale: `WORLD_SCALE = 4` in src/config.ts is the only pack-pixels→screen-pixels number (CC, 2026-08-10).
- Rendering: per-texture NEAREST + camera roundPixels, NOT global pixelArt — global would jaggy the vector-drawn UI (bubble, dots, glows) (CC, 2026-08-09, reaffirmed 2026-08-10).
- Walk-up interaction juice carried to tileset props (campfire smoke, well voice); vector star retired (CC, 2026-08-10). Walk speed 300 px/s (CC, 2026-08-10).
- Doors: entering is an A-press on the door, exiting is an automatic walk-through portal (Matt, 2026-08-10).
- Obstacle density targets ~75% of Stardew Valley and occasional 1-tile gaps are fine; the hard ≥2-tile gap rule is retired (Matt, 2026-08-10 — see design doc).
- The map boundary is obstacle-filled and build-gated; grass correctness, the ratified interior aesthetic, and "a small village with a garden+farm" all land in the design doc (Matt, 2026-08-10/11).
- **Cast** (Matt, 2026-08-11): **Sneak** — Seraphina's close friend, around her age, lives next door (a neighbor facade, flavor only); a sneaky/hiding type kept subtle and never overemphasized; obsessed with his spell book. He is NOT a sibling — never call him a brother. **Hazel** — Seraphina's little sister; the speaker key was renamed sister→hazel (no audio rebuild; the fingerprint excludes the key). The three kids are one Ana timbre at three pitches — Sneak lowest, Seraphina middle, Hazel highest — because edge-tts has exactly one child voice.
- Both kids are paper dolls (Hazel at scale 0.75, keeping the pixel multiple whole). NPCs are pure data (`NpcLayout` → `map.npcs`), walk-through with no collision (Matt), and face Seraphina on interact.
- Tool belt is 4 slots (Matt, 2026-08-11). The hammer's belt icon is a round mallet, deliberately distinct from the axe at a glance, while the in-hand swing uses the pack's pickaxe rows — the pack has no hammer, and Matt approved the mismatch (2026-08-12).
- Chop feel, from hardware (Matt, 2026-08-11/12): A-priority — the nearest in-range interactable wins and the tool swing is the fallback (whiffs anywhere, fine); trees are NOT interactables and get no green dot (the dot is reserved for quest-style interactables) — a chop connects if a tree is in range at swing start; the movement lock is removed, so she moves while swinging, the hit is decided at swing start, and the chop anim is uninterruptible for its window; the stump spawns as the fall lands, with no bare-ground beat.
- Interact and chop reach share one 1.5-tile constant and stay unified — verified fine on hardware (Matt, 2026-08-13).
- **Sleep clears everything** (Matt, 2026-08-12): ALL quests reset on sleep, no exceptions unless a need appears — she re-summons the faeries the next day. This SUPERSEDES the old "quest progress persists through sleep" rule and the tool-guarantee-covers-waking-mid-quest rationale; both are dead. Coins, when they exist, are the intended first exception.
- The bed is the sleep trigger on the ordinary two-press grammar (2026-08-13): first A speaks the repurposed `seraphina_bed` line, a second A within the ~4 s bubble window sleeps. Offer state is read off the balloon, so walk-away, timeout and red all mean no with nothing to unset; red cancels only at the bed.
- The bedtime recap is folded into the sleep sequence — not a separate feature or screen (Matt, 2026-08-13).
- Taste calls settled (Matt, 2026-08-12/13): cave walls stay grey cobble; the exterior lamp posts carry `glow`; the spell circle is a permanent cave landmark, not quest furniture.
- A quest ends on a phase with NO instruction — that absence removes the yellow dot, restores the giver's idle lines, and blocks re-offer; there is no completion flag.
- **Quest #1, the faerie summoning** (Matt's design, 2026-08-11): Sneak's spell book needs three magic stones. Find the hammer near the well (auto-equips on pickup) → break malachite/ruby/sapphire gem rocks, 2 hammer hits each, free order, the gem colors being the pad's button colors so the quest teaches the pad's color vocabulary → meet at the Secret Cave (`travel`) → ritual at the campfire spell circle, where the face buttons are claimed ONLY in circle proximity and ONLY during that phase, Sneak dictates red→green→blue one at a time reading from the book, each press sends its gem into the fire, a wrong press is a fizzle + giggle + per-color retry line, progress never regresses, and `atFire` in the done-list holds the sequence's place across exits → summoning (the biggest celebration; a ~10 s line chain over instant state completion, hammer poofing in sync) → three faeries follow her. Y is deliberately NOT claimed by the ritual: mid-sequence it answers with the awaited color.
- The cave chamber uses grey cobble walls + the pack's cavern floor. `Cave_Walls.png` is not a run kit, so a cave wall material would mean a second wall pipeline — declined for one room (CC, 2026-08-12). Wall torches carry the `glow` catalog flag ("this picture is a light").
- **Tool voice**, from hardware (Matt, 2026-08-12): Seraphina barks the tool's name on switch and an item's name on pickup (verbose is fine — it is learning material), plus a corrective line naming the RIGHT tool on a wrong-tool swing ("I need my axe!"). `bark()`/`say()` are a two-class priority: barks interrupt each other but never anything said with `say()` (NPC, quest, prop lines). Line ids are derived (`src/voice/barks.ts`), so a new tool is a `lines.json` entry and nothing else. Pickup-suppresses-switch-bark falls out structurally (pickups never route through X) and is commented against "fixing".
- Letter-sound name intros ("Mmm — malachite!") are DROPPED until the voice can do them, i.e. ElevenLabs (Matt, 2026-08-11).
- **Screenshot rule** (Matt, 2026-08-12): CC reports gather NO evidence screenshots by default — Matt verifies by playing and pastes anything relevant to claude.ai; a screenshot is included only when the prompt explicitly asks for one. Main-repo CLAUDE.md amended (`a51b33c`). Supersedes the 2026-08-10 evidence-screenshot rule.
- **Suite philosophy** (Matt, 2026-08-12): keep the suite lean and fast; aggressive cuts are fine and weaker coverage is an acceptable price, because Matt tests on hardware and exhaustive automated testing is a poor fit for a video game.

## Facts & constraints learned

- Bare `npm install phaser` resolves Phaser 4 — stay pinned `^3` (3.90.0).
- Voice/edge-tts quirk #2: a mid-line "!" yields ~0.93 s of silence — use the say/text split ("Hi! Let's play!" displays, "Hi, let's play!" is spoken). A *trailing* "!" is free. The trick works for **periods too, not just "!"**: re-cutting `seraphina_goodnight` with its mid-line "." as "," saved 0.94 s of dead air. Retires with ElevenLabs, as does quirk #1 (unvoiced letter sounds).
- edge-tts (`edge-tts-universal@1.4.0`): free, no API key, word boundaries in 100-ns ticks, ~1 s/line, zero failures in ~40 calls; unofficial API, could throttle or break someday — fine for placeholder volumes.
- The voice manifest's per-line `speaker` field can go stale if a speaker is renamed while its voice config is unchanged — the fingerprint excludes the speaker name, so nothing rebuilds. Latent, harmless so far, worth remembering at the next rename.
- Phonics on edge-tts: any run of s's is read as the letter NAME, once per letter; SSML is unusable (the package escapes it). Voiced continuants (m, n, l, r, v, z, vowels) sustain fine as "Mmm"-style spellings; unvoiced consonants (s, f, th, h) cannot — author them "Suh"-style and treat as placeholders. "Shhh" works for the *sh* digraph. Full experiment: main repo `scratch/voice_pipeline_spike_phonics.json`.
- Timing: non-initial word boundaries are frame-accurate; first and post-pause words highlight ~0.05–0.14 s early (edge-tts ignores leading silence) — deliberately left, since early highlight is the right direction for a child following along. Clips carry ~1 s trailing silence; the bubble holds on manifest duration instead. `npm run voice:inspect` measures alignment.
- `sayInstructionAfter` bug (fixed): it read the instruction when its timer FIRED, not when it was scheduled, so fast phase completion double-spoke instructions over gameplay — capture at schedule time and bail if the quest moved. The phase-end beat went 700→1100 ms so phase-ending barks are not clipped.
- Player sheets: all 9 cols × 56 rows of 64×64 (char art ~16×18 centered); rows 0–2 idle, 3–5 walk, in down/RIGHT/up order; LEFT is a code flip (pack convention). Rows 38–40 are byte-identical to 35–37 — one swing shipped twice. Aseprite files carry NO frame tags but define layer order (horse, tool_under, base, shoes, pants, shirt, hair, accesory, hands, tool_top). Pack readmes are license-only — always measure grids from PNGs.
- The pack has NO child NPC sheets and NO hammer tool art (the pickaxe rows are the only ground-striking swing), and no seated or holding poses. Gem art is `Ores.png` (world lumps + HUD icon frames in one sheet).
- NPC sheets are NOT on the player grid (Knights 48×48; Farmer_Bob not 48-divisible) — measure each sheet; animation-table reuse is unproven.
- Interior furniture and wall sheets are not on a safely indexable grid — every image is measured individually (why the catalog is ~60 measured images, not an index).
- Movement is substepped ≤16 px per collision step (headless frames reach 250 ms; one-jump-per-frame tunnels or crawls).
- Tall sprites fade to 42% when the player is behind them; buildings joined the fade.
- Keyboard B is the hitbox debug overlay, so the ritual's red button is C and L on keys (L = diamond right, where pad-B sits); the pad is unaffected.
- `world:footings` checks a rect's center/bottom alignment but never its SIZE — it cannot catch oversized collision rects (a future hardening candidate).
- Suite shape: the diet took it 44→17 tests, and it has been **18** since checkpoint 4 (`landmarks.spec` grew a third tour with the cave) — the "17" carried in the last record was stale, and the main CLAUDE.md is corrected. Page boot (~4.5 s each) dominates, so merge assertions into shared boots and teleport where the walk is not what's under test. Parallel workers were tried and abandoned with numbers (2 workers slower + a timeout; a GPU-less shared renderer plus wall-clock key holds); config stays 1 worker.
- Wall time crept 4.0→6.7 min across the day-cycle era, purely from assertions folded into existing tests. Acceptable, but the suite is an eventual diet-pass candidate.
- Watch flags: the first swing of a session runs long (still unexplained); the two-event night at ≈11 s may be too long a hold and is unverified with Julia; green-mashing at the bed mid-quest could accidentally sleep away a quest (mitigation if it is ever observed: arm the confirm only after the line finishes).

## Verification ledger

Everything shipped is Matt-verified: he hardware-verified the checkpoint-3 era on 2026-08-11, played quest #1 end to end on 2026-08-12, and hardware-verified the whole day-cycle era on 2026-08-13. Matt-verified = Julia-verified for now. The pad path remains hand-test-only forever (headless Playwright cannot drive the Gamepad API); everything else Playwright covers is CC-verified on top.

## Open questions

- The 2 hand-placed pond trees lack the choppable flag (the only hand-authored interior placement) — flip at the next world rebuild if anyone cares.
- NPC seated/holding poses are an art gap: Sneak's book and Hazel's pebble lie at their feet as scenery (accepted v0).
- Interior edging is incomplete in places — cleanup pass, not blocking (Matt, 2026-08-11). The wood's eastern half is thin now the road is gone — planting-pass candidate. Tabletop dressing needs a per-placement depth lift (a candle on a table draws behind the table).
- Line length for a 4-year-old: wrap at 620 px, two-line bubbles untested with Julia — observe in play.
- Accepted as-is: 14 furniture images block 5–12 px below their drawn base; `oakBig2`/`spruceBig2` are shadow-less art variants (palette call open); the overlay doesn't draw doorway trigger rects.
- Measured-but-unused inventory: bathroom kit, interior doors, planters, greenhouse-interior tiles, stairs, kitchen wall-cupboards; sofa/armchair side facings won't cut. Uncataloged exterior sheets: windmill, barn, greenhouse, cobble road.
- Plus the design doc's existing opens (Secret mechanics, dragon care loop, etc.).

## Plan

**Next tasks, decided (Matt, 2026-08-13), with a design conversation owed before each prompt:**

1. **The bunny quest** — the second quest template, exercising the grammar on new ground: free the bunny, using chopping-opens-a-path as the rescue verb.
2. **The reading flagship** — Hazel + a book.

## Parked

- Coins and the wardrobe. Note the seam: `reset()` and `resetForSleep()` are byte-identical today and kept separate precisely because coins are the first thing that will diverge them.
- Disk persistence (the store is session-scoped; a reload is a fresh day).
- A visible dad NPC, and family schedules / activity loops.
- Weather.
- The interior-edging cleanup pass.
- ElevenLabs + unvoiced phonics (and with it, letter-sound name intros).
- Dragons — the cave is built, but nothing dragon-shaped exists yet.
- Facades becoming enterable.
