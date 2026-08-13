# systems — Seraphina's Secret

*A compact index of the machinery, so claude.ai can write prompts without the state doc bloating. One line per system: enough to know a thing exists and what its shape is. The detail lives in the MAIN repo's `docs/systems.md` and `docs/engineering.md` (CC-maintained) — cite those by path in prompts. New at checkpoint 6, 2026-08-13.*

- **Session store**: `run` / `world` / `persistent` halves; `resetForSleep` clears run+world and keeps persistent (coins); a cold `reset()` clears all; `run.completed` is the set of quests finished today. Session-scoped — a page reload is still a fresh day.
- **Quest engine**: thought-bubble two-press offers (the second press IS acceptance); phase goal kinds travel / gather / chop-count / lure-follower / park (an instruction-less phase ends the quest); one active quest absolute; `inPlay()` = active + finished-today. **Quest furniture is runtime-spawned and invisible to `world:build` and the static grid checks** — placement is hand-held against it (the gate is parked).
- **Followers**: leash machinery (faeries, bunnies), off-grid, no collision, 250 px/s.
- **Voice**: edge-tts. Prompt-facing rules — no mid-line `!` or `.` (use the say/text split), trailing punctuation is free; pre-cut lines only, so no dynamic numbers; ~1 s per line; the rebuild fingerprint excludes the speaker id.
- **Art**: measure sheets, never assume; pack readmes are license-only; the bunny swap contract is the constants atop `src/world/Bunny.ts`.
- **Suite**: 19 tests (14 fast ~5.2 min, 5 @slow, `test:all` ~6.7 min); 150 s per-test cap, the two-quest e2e nearest at 1.6 min. Lean-suite rules: shared boots, teleport where the walk is not what's under test, cut what Matt's play would catch.
- **Day cycle**: sleep → quests and quest furniture despawn, world regenerates, run+world reset; then dusk. Recap priority faeries > bunnies > stones/trees/errand, max 2 lines + the goodnight.
- **Coins**: `addCoin`, 3 slots, overflow bounce; the grant settles state instantly and defers the flourish.
