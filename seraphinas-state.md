# state — Seraphina's Secret

*The project state doc: status, plan, verification ledger, watch flags, parked and open questions. Rewritten in full at each checkpoint distillation. At session boot, claude.ai reads this doc first.*

## What this project is

"Seraphina's Secret" — a cozy, no-fail 2D exploration game for Julia (Matt's 4-year-old): Seraphina, a young girl, secretly raises baby dragons, and her dad doesn't know. Learning goals in priority order: interactive engagement, menu/UI navigation, goal-holding, walking/spatial navigation, ambient reading exposure (letter *sounds*, not names).

## The doc set

Five controller docs, flat at the root, written by claude.ai for claude.ai: **seraphinas-state.md** (this one — status, plan, ledger, watch/parked/open; rewritten every checkpoint), **seraphinas-design.md** (design canon: premise, characters, quests, reading, economy, menus), **seraphinas-world.md** (zones, layout, boundary, aesthetics, measured world facts), **seraphinas-workflow.md** (the three-part workflow), **seraphinas-systems.md** (a compact machinery index for prompt-writing). Engineering reference now lives in the MAIN repo and is CC-maintained there: `docs/systems.md` (how the machinery works) and `docs/engineering.md` (facts, constraints, gotchas) — cite them by path in prompts. The old "Infrastructure now in place" and "Facts & constraints learned" sections of this doc were retired into that pair at checkpoint 6; read them there, not here.

## Status (2026-08-13, checkpoint 6)

Checkpoint 6, 2026-08-13 (same-day as checkpoint 5 — a fast era). Shipped and Matt-verified on hardware: quest #2 "the bunny rescue" (Hazel gives; pen/chop/carrots/lure loop; full design now in the design doc) and coins v0 (persist over sleep — the first `resetForSleep`/`reset` divergence; 3-slot HUD; every quest pays 1 at completion). Two-quest days work: all un-completed-today quests offer, one active, completing restores the others (`run.completed`; `inPlay()` keeps a finished quest's furniture alive) — CC-verified end-to-end, Matt's casual verify pending. Bunny art is a stand-in (white frog, "bunny" everywhere player-facing); Matt redraws later — the swap contract is the constants atop `src/world/Bunny.ts`. Repo-side `docs/` now owns engineering reference. Suite: 19 tests = 14 fast (~5.2 min) + 5 @slow; `test:all` ~6.7 min.

## Plan

**Next: the reading flagship** — Hazel + a book (design template 5). A design conversation is owed before any prompt is written.

Behind it, unscheduled: the woods planting / aesthetics pass; a third quest; the wardrobe coin sink; the quest-furniture overlap gate.

## Verification ledger

Everything shipped is Matt-verified: he hardware-verified the checkpoint-3 era on 2026-08-11, played quest #1 end to end on 2026-08-12, and hardware-verified the whole day-cycle era on 2026-08-13. Matt-verified = Julia-verified for now. The pad path remains hand-test-only forever (headless Playwright cannot drive the Gamepad API); everything else Playwright covers is CC-verified on top. Added this era: Matt played the bunny quest's full loop on 2026-08-13; the two-quest day is CC-verified end-to-end only, with Matt's casual verify still pending.

## Watch flags

- The first swing of a session runs long (still unexplained).
- The two-event night at ≈11 s may be too long a hold and is unverified with Julia.
- Green-mashing at the bed mid-quest could accidentally sleep away a quest (mitigation if it is ever observed: arm the confirm only after the line finishes).
- The merged two-quest e2e already runs 1.6 min against the 150 s per-test cap — there is no room for a third quest in it, so a third means splitting the test or moving the cap.
- The woods-walk @slow test flaked once (29 s timeout) and passed on retry.

## Open questions

- The 2 hand-placed pond trees lack the choppable flag (the only hand-authored interior placement) — flip at the next world rebuild if anyone cares.
- NPC seated/holding poses are an art gap: Sneak's book and Hazel's pebble lie at their feet as scenery (accepted v0).
- Interior edging is incomplete in places — cleanup pass, not blocking (Matt, 2026-08-11). Tabletop dressing needs a per-placement depth lift (a candle on a table draws behind the table).
- Line length for a 4-year-old: wrap at 620 px, two-line bubbles untested with Julia — observe in play.
- Accepted as-is: 14 furniture images block 5–12 px below their drawn base; `oakBig2`/`spruceBig2` are shadow-less art variants (palette call open); the overlay doesn't draw doorway trigger rects.
- Measured-but-unused inventory: bathroom kit, interior doors, planters, greenhouse-interior tiles, stairs, kitchen wall-cupboards; sofa/armchair side facings won't cut. Uncataloged exterior sheets: windmill, barn, greenhouse, cobble road.
- Plus the design doc's existing opens (Secret mechanics, dragon care loop, etc.).

## Parked

- **The quest-furniture overlap gate.** A comment above the stones in `quests.ts` is the only guard: `world:build` and the static grid checks never see runtime quest furniture, so a quest prop landing on a world prop is a class of collision nothing catches (it already happened once — the malachite stone vs the bunny pen).
- **The wardrobe / coin sink.** Coins exist; the spend side does not.
- Disk persistence (the store is session-scoped; a reload is a fresh day).
- The woods planting / density pass — the Mystic Woods interior measures very open.
- Matt's bunny art redraw (the stand-in swaps at the constants atop `src/world/Bunny.ts`).
- A visible dad NPC, and family schedules / activity loops.
- Weather.
- The interior-edging cleanup pass.
- ElevenLabs + unvoiced phonics (and with it, letter-sound name intros).
- Dragons — the cave is built, but nothing dragon-shaped exists yet.
- Facades becoming enterable.

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
- World is big and scrollable, Stardew-style; "static one-screen room" retired (Matt, 2026-08-08). Full world scope laid 2026-08-08 (see the world doc).
- One-NPC-per-screen rule dropped entirely (Matt, 2026-08-09).
- Art: full Cute Fantasy RPG pack (Kenmi) purchased (Matt, 2026-08-09). Side-loaded at `C:\Code\seraphinas-assets`; license forbids redistribution → pack files NEVER enter the repo. Gitignored `public/assets/` populated by `assets:sync` (predev/prebuild) from `tools/assets/config.ts`; missing side-load fails loudly; attribution in README.
- Player is the pack's paper doll (2026-08-09): base+shoes+pants+shirt+hair layered sheets — an outfit is a layer list, so wardrobe/dress-up/coin-sink is data. Seraphina's blessed look: `Hair_4_Blonde` (golden — Matt, 2026-08-10), `Shirt_1_Pink`, `Pants_1_Blue`, `Shoes_1_Brown` (Matt, 2026-08-09).
- One world scale: `WORLD_SCALE = 4` in src/config.ts is the only pack-pixels→screen-pixels number (CC, 2026-08-10).
- Rendering: per-texture NEAREST + camera roundPixels, NOT global pixelArt — global would jaggy the vector-drawn UI (bubble, dots, glows) (CC, 2026-08-09, reaffirmed 2026-08-10).
- Walk-up interaction juice carried to tileset props (campfire smoke, well voice); vector star retired (CC, 2026-08-10). Walk speed 300 px/s (CC, 2026-08-10).
- Doors: entering is an A-press on the door, exiting is an automatic walk-through portal (Matt, 2026-08-10).
- Obstacle density targets ~75% of Stardew Valley and occasional 1-tile gaps are fine; the hard ≥2-tile gap rule is retired (Matt, 2026-08-10 — see design doc).
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
