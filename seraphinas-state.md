# state — Seraphina's Secret

*The project state doc: status, decision log, open questions, plan. Rewritten in full at each checkpoint distillation. At session boot, claude.ai reads this doc first.*

## What this project is

"Seraphina's Secret" — a cozy, no-fail 2D exploration game for Julia (Matt's 4-year-old): Seraphina, a young girl, secretly raises baby dragons, and her dad doesn't know. Learning goals in priority order: interactive engagement, menu/UI navigation, goal-holding, walking/spatial navigation, ambient reading exposure (letter *sounds*, not names). Full design in seraphinas-design.md; how the project runs in seraphinas-workflow.md.

## Status (2026-08-08, checkpoint 1)

- Design v1 complete (seraphinas-design.md; unchanged this era).
- Main repo `C:\Code\seraphinas-secret`: Phaser 3 + Vite + TypeScript skeleton — one room, stick/arrow-key walking, particle+sound interaction, and a working SpeechBubble (pre-generated voice + per-word highlighting) wired to it. Voice pipeline at `tools/voice/` (edge-tts proxy voices → `public/voice/` audio + `manifest.json`; 9 real lines, 3 voices). Playwright: 6 tests passing; screenshots in `tests/screenshots/` are the planner's visual audit channel (`04-speech-bubble.png` shows a highlight mid-line). Commits `12b8f81` → `ed2a1e0`.
- Controller repo initialized: flat `seraphinas-` prefixed docs, mechanical-applier CLAUDE.md.
- No world, NPC, or quest content yet. All voices are placeholders.

## Decision log

- Title & premise: "Seraphina's Secret" (Matt, 2026-08-08).
- Three-part workflow adopted; repos: `seraphinas-secret` (main/code), `seraphinas-controller` (docs), `seraphinas-drive-sync` (Drive-synced exchange, not in git) (Matt, 2026-08-08).
- Stack: web — TypeScript + Phaser 3 + Vite, npm; local browser page, F11 fullscreen, "dad opens it" launch. Chosen for CC iteration speed, including Playwright screenshot audits (Matt, 2026-08-08).
- Phaser 3 confirmed over Phaser 4 for v1 — CC fluency and API stability outweigh "latest"; revisit only if 4 offers something the game needs (Matt, 2026-08-08).
- Voice: all audio pre-generated at content time, never runtime; provider must return word timestamps (Matt, 2026-08-08). Start free with edge-tts "proxy voices"; swap to ElevenLabs later by re-running generation — the game reads only the provider-neutral `public/voice/manifest.json` (Matt, 2026-08-08).
- Proxy voices ear-checked, accepted as placeholders (Matt, 2026-08-08): Seraphina `en-US-AnaNeural` (rate −8%), Dad `en-US-GuyNeural` (rate −10%, pitch −15 Hz), Little Sister `en-US-AnaNeural` (rate +6%, pitch +40 Hz). All get replaced when ElevenLabs lands.
- Letter-sound (phonics) lines: do NOT try to solve sustained unvoiced consonants on the proxy TTS; "Suh"-style approximations are acceptable placeholders until ElevenLabs (Matt, 2026-08-08).
- Voice content schema: a line has display `text`, optional spoken `say`, and per-line prosody overrides; the manifest carries every display token (punctuation-only tokens get `start === end` and never highlight); alignment mismatches fail the build (CC, 2026-08-08).
- Audio architecture: one shared AudioContext (`src/audio/context.ts`) for voice and sfx, one unlock; the highlight clock is `ctx.currentTime`, so voice playback bypasses Phaser's sound manager (CC, 2026-08-08).
- Test-shaped code lives only in `src/testHooks.ts`; tests freeze the scene or read high-water marks rather than racing transient effects (CC, 2026-08-08).

## Facts & constraints learned

- Bare `npm install phaser` resolves Phaser 4 — stay pinned `^3` (3.90.0).
- A gamepad is invisible to the browser until a button is pressed on it, and audio stays `suspended` until a user gesture — a title screen ("press the green button!") solves both with one press. Stopgap HUD hint in place; pad code (standard XInput mapping) still unverified on real 360 hardware.
- edge-tts (`edge-tts-universal@1.4.0`): free, no API key, word boundaries in 100-ns ticks, ~1 s/line, zero failures in ~40 calls; unofficial API, could throttle or break someday — fine for placeholder volumes.
- Phonics on edge-tts: any run of s's is read as the letter NAME, once per letter; SSML is unusable (the package escapes it). Voiced continuants (m, n, l, r, v, z, vowels) sustain fine as "Mmm"-style spellings; unvoiced consonants (s, f, th, h) cannot — author them "Suh"-style and treat as placeholders. "Shhh" works for the *sh* digraph. Full experiment: main repo `scratch/voice_pipeline_spike_phonics.json`.
- Timing: non-initial word boundaries are frame-accurate; first and post-pause words highlight ~0.05–0.14 s early (edge-tts ignores leading silence) — deliberately left, since early highlight is the right direction for a child following along. Clips carry ~1 s trailing silence; the bubble holds on manifest duration instead. `npm run voice:inspect` measures alignment.

## Open questions

- Design-level opens: final section of seraphinas-design.md (the Secret's mechanics, dragon care loop, sticker album, content cadence, mystery-request difficulty).
- SpeechBubble anchors only to the player; NPC lines need speaker anchors (arrives with NPCs).
- Line length for a 4-year-old: wrap at 620 px, two-line bubbles untested with Julia — observe in play.

## Plan — near-term candidates (not yet ordered)

- Title screen: pad wake + audio unlock + a voiced "Press the green button!"
- Rooms & doorways: two rooms with a juicy transition; the skeleton grows into the room graph.
- First NPC + quest grammar skeleton (speech bubble gains speaker anchors here).
- Real-pad verification (Matt, manual, no prompt) — still pending.

## Parked

- Sustained unvoiced letter-sounds (s, f, th, h) — until ElevenLabs.
- ElevenLabs provider module — someday, when placeholders need replacing.
