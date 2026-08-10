# state — Seraphina's Secret

*The project state doc: status, decision log, open questions, plan. Rewritten in full at each checkpoint distillation. At session boot, claude.ai reads this doc first.*

## What this project is

"Seraphina's Secret" — a cozy, no-fail 2D exploration game for Julia (Matt's 4-year-old): Seraphina, a young girl, secretly raises baby dragons, and her dad doesn't know. Learning goals in priority order: interactive engagement, menu/UI navigation, goal-holding, walking/spatial navigation, ambient reading exposure (letter *sounds*, not names). Full design in seraphinas-design.md; how the project runs in seraphinas-workflow.md.

## Status (2026-08-10, checkpoint 2)

All hardware-verified by Matt/Julia on the real 360 pad: title screen (one press wakes the pad AND starts the game, resumes audio, voiced "Hi! Let's play!" with per-word highlight, flourish into the world), the animated player, and the tile world. Matt's verdict 2026-08-10: "all the design feels are fine"; pathing/map/aesthetics need improvement but base infrastructure is good. Suite: 23 Playwright tests, ~6.2 min single worker, typecheck green.

Main repo `C:\Code\seraphinas-secret`: Phaser 3 + Vite + TypeScript; voice pipeline at `tools/voice/` (edge-tts proxy voices → `public/voice/` audio + `manifest.json`). No NPC or quest content yet; all voices are placeholders.

## The world (world_rebuild, 2026-08-10)

Two zones. **Outside**: 64×44 tiles (~3.2×3.9 screens at 1280×720) — Seraphina's house (enterable), Dad's shed, Joey's house, Scar's house, Secret Cave mouth (facades, not enterable; interacting gives a wiggle + burst + two knocks, no text), the Mystic Woods down the left 16 columns, a fenced vegetable patch, pond, joined dirt roads, 367 sprites. **House**: 40×26, four rooms on ONE scrolling floor plan, no internal transitions, placeholder static furniture. Maps are generated data: `content/world/layout.ts` → `tools/world/build.ts` → JSON; future map changes are data edits. Exterior↔interior via the flourish doorway system (walk-through, no button).

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
- Player is the pack's paper doll (2026-08-09): base+shoes+pants+shirt+hair layered sheets — an outfit is a layer list, so wardrobe/dress-up/coin-sink is data. Seraphina's blessed look: Hair_4_Brown, Shirt_1_Pink, Pants_1_Blue, Shoes_1_Brown (Matt, 2026-08-09).
- One world scale: `WORLD_SCALE = 4` in src/config.ts is the only pack-pixels→screen-pixels number (CC, 2026-08-10).
- Rendering: per-texture NEAREST + camera roundPixels, NOT global pixelArt — global would jaggy the vector-drawn UI (bubble, dots, glows) (CC, 2026-08-09, reaffirmed 2026-08-10).
- Walk-up interaction juice carried to tileset props (campfire smoke, well voice); vector star retired (CC, 2026-08-10).
- Walk speed 300 px/s (CC, 2026-08-10; Matt's hardware walk accepted the feel).

## Facts & constraints learned

- Bare `npm install phaser` resolves Phaser 4 — stay pinned `^3` (3.90.0).
- Voice/edge-tts quirk #2: a mid-line "!" yields ~0.93 s of silence — use the say/text split ("Hi! Let's play!" displays, "Hi, let's play!" is spoken). Retires with ElevenLabs, as does quirk #1 (unvoiced letter sounds).
- edge-tts (`edge-tts-universal@1.4.0`): free, no API key, word boundaries in 100-ns ticks, ~1 s/line, zero failures in ~40 calls; unofficial API, could throttle or break someday — fine for placeholder volumes.
- Phonics on edge-tts: any run of s's is read as the letter NAME, once per letter; SSML is unusable (the package escapes it). Voiced continuants (m, n, l, r, v, z, vowels) sustain fine as "Mmm"-style spellings; unvoiced consonants (s, f, th, h) cannot — author them "Suh"-style and treat as placeholders. "Shhh" works for the *sh* digraph. Full experiment: main repo `scratch/voice_pipeline_spike_phonics.json`.
- Timing: non-initial word boundaries are frame-accurate; first and post-pause words highlight ~0.05–0.14 s early (edge-tts ignores leading silence) — deliberately left, since early highlight is the right direction for a child following along. Clips carry ~1 s trailing silence; the bubble holds on manifest duration instead. `npm run voice:inspect` measures alignment.
- Player sheets: all 9 cols × 56 rows of 64×64 (char art ~16×18 centered); rows 0–2 idle, 3–5 walk, in down/RIGHT/up order; LEFT is a code flip (pack convention). Aseprite files carry NO frame tags but define layer order (horse, tool_under, base, shoes, pants, shirt, hair, accesory, hands, tool_top). Pack readmes are license-only — always measure grids from PNGs.
- NPC sheets are NOT on the player grid (Knights 48×48; Farmer_Bob not 48-divisible) — the NPC prompt must measure each sheet; animation-table reuse is unproven.
- Interior furniture sheets are not on a safely indexable grid (why house furniture is static placeholder).
- Movement is substepped ≤16 px per collision step (headless frames reach 250 ms; one-jump-per-frame tunnels or crawls).
- Tall sprites fade to 42% when the player is behind them (she must never vanish behind a canopy).
- Harness (standing constraints): headless runs ~15–29 FPS; NEVER assume a timed key hold moves a predictable distance — `walk()` waits for the idle animation; harness route-plans breadth-first over the exposed collision grid; screenshot tour teleports (`teleport(x,y)` hook); per-test timeout 150 s; `artLoaded` hook guards the silent green-square fallback when a sheet 404s; Phaser does not cull sprites (the 367 exterior sprites are the perf cost).

## Verification ledger

Matt tracks CC-verified vs Matt-verified; Matt-verified = Julia-verified for now.

- **Matt/Julia:** pad single-press wake+start, greeting audio+highlight, transitions, walking, interactables, animated character + blessed outfit, the tile world's feel, walk speed.
- **CC-only:** everything Playwright covers per above.
- The pad path is hand-test-only forever (headless Playwright cannot drive the Gamepad API).

## Open questions

- Quest/world state layer — nothing persists across zone changes today (a poked prop resets); quests need state that survives zone restarts and later sleep — build it with the first quest prompt.
- NPC speech bubbles need speaker anchors. The bed currently speaks a dad-voiced line (props reuse existing manifest audio) — speaker attribution to fix when NPCs land.
- Map/pathing/aesthetic improvement vs the new guideposts (see design doc).
- Line length for a 4-year-old: wrap at 620 px, two-line bubbles untested with Julia — observe in play.
- Plus the design doc's existing opens (Secret mechanics, dragon care loop, etc.).

## Plan — near-term candidates

- First NPC + quest grammar skeleton + persistence layer (the next big one).
- "World life" polish pass — animate the pack's strips (campfire/water/chest currently draw frame 0), grass variants (pack has 4 + edge sets), furniture, aesthetics per guideposts.

## Parked

- ElevenLabs + unvoiced phonics.
- Secret Cave interior (with dragons); facades becoming enterable.
