# Seraphina's Secret — Design Doc

*v1 basis (Matt, 2026-08-08). Supersedes the retired seraphinas_secret_design.md upload. A cozy, flashy, no-fail exploration game for a 4-year-old. Seraphina, a young girl, secretly raises baby dragons — and her dad doesn't know. Basis for ongoing design; everything here is iterable.*

---

## Player & goals

- **Player:** 4 years old named Julia, but the main character she plays in the game will be called Seraphina. Knows all letter names (not sounds yet), plays Stardew Valley with a standard Xbox 360 controller, can hunt-and-peck letters on a keyboard.
- **Learning goals (in priority order):** interactive engagement first; menu/UI navigation; goal-holding and completion; walking/spatial navigation; occasional ambient reading exposure. Reading is present but deliberately *not* the focus.
- **What she loves in Stardew (design north stars):** wandering between rooms, chopping trees, scything grass, going to bed, meeting/talking to people.

## Design principles

1. **Everything reacts.** The core loop is walk → press green → something delightful happens. Density of low-stakes interactions, not systems.
2. **Pointlessly flashy is load-bearing.** Particles, sounds, and celebrations are the reward loop, not decoration. Every tool swing, every quest completion, every door transition gets juice. Dragons are the flash license (glitter trails, sneeze-sparks, rainbow fire).
3. **No fail states, ever.** No timers, no death, no wrong-answer buzzers. Wrong choices do something mildly funny instead. Autosave everywhere.
4. **Audio-first UI.** She can't read, so *every* piece of text speaks: menu items read themselves aloud on focus, signs talk, all dialog is voiced. Text is always highlighted while spoken — this pairing is the game's main reading instruction. **Narrative voice rule** (Matt, 2026-08-13): anything that is not a character speaking — prop lines, world flavor, narration, the bedtime recap — defaults to Seraphina's voice, since she is the player's companion in the world; only the real characters (dad, Sneak, Hazel) speak in their own. Also standing in the main repo's CLAUDE.md.
5. **Sounds, not letter names.** Letter names are a done skill; sounds are her frontier. Letters are voiced as sounds with a word: "Sss — Sparky!" **Deferred in practice:** letter-sound name intros ("Mmm — malachite!") are DROPPED until the voice can actually do them, i.e. ElevenLabs — the proxy TTS reads a sustained consonant as the letter's name (Matt, 2026-08-11). The principle stands; the delivery waits.
6. **One interaction grammar.** Select-and-confirm works the same in the shop, the wardrobe, and quest menus. All quests display as *N slots that fill up*.
7. **Obstacle density is Stardew-comfortable** (Matt, 2026-08-10, revising the hard "walkable gaps ≥2 tiles everywhere" ratified earlier that day). Occasional 1-tile gaps are fine — Julia navigates Stardew-level obstacles comfortably; prefer props whose collision blocks one tile. Target ~75% of Stardew Valley's obstacle density: err slightly under, never sparse.
8. **The player must never fully disappear** (ratified 2026-08-10). Tall sprites fade when she's behind them.

## Controls (Xbox 360, colors match the physical pad)

| Button | Color | Function |
|---|---|---|
| A | Green | "Do stuff" (interact, confirm, swing tool, advance) |
| B | Red | Exit menu / cancel |
| X | Blue | Switch tool |
| Y | Yellow | "Help me" — replays current quest |
| Stick | — | Walk |

- All on-screen button prompts are **colored dots**, never letters ("press the green button" is literally true on the hardware).
- **Yellow / Help me:** exact replay of the current quest phase — same NPC portrait, same audio clip, same icon, plus the slot-progress row. Objectives sparkle passively whenever they are current, not only while help plays (shipped 2026-08-11).
- Keyboard is not a parallel input mode; it appears as an occasional in-world object (e.g. a "name machine" / typewriter where she hunt-and-pecks letters).

## World

- **Zone-based, not one open map.** Big scrollable zones joined by doorway transitions, each transition its own little flourish. Insides in general are big and scrollable (Matt, 2026-08-08).
- **Doors are the Stardew copy** (Matt, 2026-08-10, supersedes "walk-through, no button" of 2026-08-08). Entering a building is an A-press on the door — a standard interactable, with the proximity green dot and the existing flourish. Exiting is an automatic walk-through portal. Automatic entry is reserved for future cellar-type internal transitions.
- **Zone "Outside":** one big scrollable map, 72×50 tiles, holding Seraphina's house, Dad's shed, The Secret Cave, Joey's house (a mermaid friend), Scar's house (a robot friend), with **The Mystic Woods** as the far-left wooded region — not too big; special quests occasionally lead there, and a possible future "adventure mode" would live in the forest. It reads as "a small village with a garden+farm, not a farm with some sheds" (Matt, 2026-08-10/11): the farm/garden is about a quarter of the map, and the landmarks now also include a village hall (a knockable facade), 3 market stalls, a silo, a market square, and a green. The well sits on Seraphina's street; spawn is beside her front door with the interaction dot showing. All Matt-approved 2026-08-11.
- **Zone "House":** the largest indoor space — many connected rooms as ONE big scrollable map, no portals between interior rooms.
- **The map boundary must be clearly obstacle-filled** (Matt, 2026-08-10): visually a wall of stuff, mechanically no walkable route to any map-edge cell (build-gated — see the state doc). North is a mountain cliff with the Secret Cave mouth cut into the face (the Stardew mine pattern); the chamber behind it is deliberately one screen, and its **spell circle is a permanent cave landmark, not quest furniture** (Matt, 2026-08-13) — it stays after quest #1 ends, ready for the next ritual. East and south are fence lines with trees behind. West is the Mystic Woods' own density. One soft closed gap of undergrowth at the far west marks the future woods exit — it reads as a path without being walkable.
- World regenerates overnight: grass regrows, trees regrow, stones respawn, hidden objects re-roll positions.
- **Choppable trees** (shipped 2026-08-11): three whacks fell a tree, the juice escalating with each hit, leaving a stump; two more whacks clear the stump and **the ground becomes walkable**. Choppability is per-placement, not per-species — a flag on each tree, so the world decides exactly which trees are content. Perimeter and boundary trees are never choppable: they shake and shed leaves forever, and the boundary build gate re-runs with every choppable tree's cells already removed, so **she can never chop her way out of the world.** Chopping opening a path is an affordance to design with, not a side effect — the filed **free-the-bunny** idea (chop the trees penning something in, to let it out) is the first quest meant to use it.
- **Buildings collide only at their base** (~2-tile bands measured off the art) and join the tall-sprite fade, so she can walk behind a house and still be visible. The **green proximity dot is reserved for quest-style interactables** — doors, props, quest objects — and deliberately does NOT appear on trees or ordinary tool targets; a chop just connects if a tree is in range when the swing starts. The dot's activation radius is due to shrink ~30% (Matt, 2026-08-12).

### World aesthetics guideposts

*(Matt, 2026-08-10, from reference images)*

> **Interiors:** every room is built around one focal arrangement — a counter, a sofa on a rug under a lamp, a bed — with furniture hugging walls and corners, leaving generous open floor she can cross without steering. Detail lives on walls and edges (shelves, windows, jars, pictures); floors stay clear except for a rug anchoring the centerpiece. Floor-material changes (wood → stone) mark what a sub-area is for without adding walls. Interesting but not crowded: clusters of themed props with real negative space between them, and a warm, cohesive palette per room.
>
> **Exteriors:** wide, clearly drawn roads (2–4 tiles) connect every landmark, with lamp posts, signs, and flowers lining road edges and building fronts rather than scattered across open ground. Decoration clusters where it means something — flower pots at a door, hedges along a wall, a bench beside the path — and grass stays mostly open between clusters, so buildings read from a distance and walking anywhere is easy. The world should look composed, like someone arranged it — not filled.

**Interiors: ratified against a reference image and now implemented** (Matt, 2026-08-10, replacing the "flat peach" placeholder era). Rooms follow the pack-derived reference: real walls with caps/trim and run edging (seamed end tiles at run ends, doorway jambs, room joins); one floor material per room, with a rug anchoring the focal arrangement; furniture hugging walls. Arrangements are baked in — player furniture rearrangement is not planned. Built and approved 2026-08-11, with some edging still incomplete (a known cleanup item, see the state doc).

**Grass correctness rule** (Matt, 2026-08-10). A tile may sit on or beside the base outdoor grass only if its own background IS that base grass; ornamental tilesets carrying a different baked-in green are out. Transparent-background decoration on base grass is fine. The brown ploughed farmland tiles are explicitly approved — brown reads as a dug bed, not a competing green (Matt, 2026-08-11).

## Characters

- **Seraphina** (the player) — a young girl secretly raising baby dragons. Customizable outfits (the coin sink).
- **Family (entirely made-up, not the real family):** one dad and **Hazel**, Seraphina's little sister. They wander room-to-room on a simple schedule, always *visibly doing something* (chopping wood, drawing, stacking blocks) — an NPC's "life" is their animation loop.
- **Sneak** (Matt, 2026-08-11) — Seraphina's close friend, around her age, who lives next door in one of the neighbor facades (flavor only, not enterable). A sneaky/hiding type of person, kept subtle and never overemphasized; obsessed with his spell book, which is what makes him the first quest giver. **He is NOT a sibling — never a brother.**
- Sneak and Hazel are paper dolls like Seraphina (the art pack has no child NPC sheets, and no seated or holding poses — so Sneak's book and Hazel's pebble lie at their feet as scenery, an accepted art gap). NPCs are pure data, walk-through with no collision, and turn to face Seraphina when she interacts.
- **Neighbors:** **Joey**, a mermaid friend, and **Scar**, a robot friend — each with a house on the Outside map.
- **Baby dragons:** small, chaotic, adorable. Each dragon has:
  - a name starting with a distinct letter, always introduced by sound ("Sss — Sparky!"), with the name on a little sign by its nest (first environmental reading words);
  - one personality trait that generates its mischief quests (see quests).
  - Starter roster idea: **Sparky** (starts little fires), **Digby** (buries things — canonically explains tools going missing overnight), a sneezer, a mud-lover, a thief who hoards shiny things.

  Dragons are not the only source of quests.

### The Secret (running joke)

Dad doesn't know about the dragons. Seraphina can talk to Julia occasionally, reminding her of things, and say things like "I need your help making sure Dad never finds this!" sort of thing, Seraphina can have a voice for talking to Julia. Proposed mechanics (open to iteration):

- Dragons **hide automatically** whenever dad comes near (dive into pots, under beds, behind trees). Near-miss comedy: dad wanders in, dragons scatter, dad says "hm, did something sparkle?"
- Siblings are in on the secret (co-conspirators, source of dragon-related quests and cover stories).
- Dad occasionally *almost* finds out ("What's this scorch mark?!") — pure flavor, never a fail state.
- Optional quest flavor: "Dad's coming! Help the dragons hide!" (shoo each dragon to a hiding spot — no timer pressure that can fail, just a fun scramble).

## Tools

- Tools are **found in the world**: lightly hidden, or a sibling/dragon has one ("Digby buried your pickaxe again!").
- **Tools do not persist over sleep** — re-finding them is content, framed as dragon/sibling mischief.
- **Guarantee rule:** every morning, any tool required by an active quest is guaranteed to spawn somewhere findable.
- Held tools always clearly visible in the HUD (icon row, current tool highlighted; blue button cycles).
- **The belt is 4 slots** (Matt, 2026-08-11), empty slots drawn so the row never reflows. The **axe is permanent in slot 1** and can never be taken away; slots 2–4 hold quest-granted tools, which are revoked when the quest completes. Blue cycles, skipping empties.
- **Tools talk** (Matt, 2026-08-12): Seraphina barks the tool's name on every switch and an item's name on every pickup — verbose is the point, it is learning material. Swinging the wrong tool at something plays a corrective line naming the RIGHT one ("I need my axe!"). This is the no-fail-state version of a wrong answer: **the wrong tool never damages anything, it just tells her which tool to go and find.**
- V1 tool set (iterable): **scythe** (grass → confetti/coins), **axe** (trees fall with a big crash), **hammer** (colored stones), **watering can** (waters plants; doubles as the fire-hose for dragon fires; washes muddy dragons).

## Economy

- **Coins: max 3**, displayed as three discrete coin slots (subitizable at a glance; teaches 1–2–3 correspondence; prices shown as coin icons).
- Every minigame / fetch quest awards **1 coin**. Coins **persist over sleep**.
- When full, extra coins bounce off with a happy sound — nothing lost that matters.
- **V1 coin sink: outfits** (Seraphina's wardrobe; costs 1–3 coins). Natural extensions later: dragon accessories (bows, hats), new books.
- The art pack's paper-doll layers make outfits pure data; the purchased pack's unused layer sheets are the wardrobe inventory.

## Menus (the UI curriculum)

Menus are intrinsically motivated, never administrative. All share select-and-confirm, all items speak on focus:

- **Wardrobe / dress-up** — grid navigation, categories, confirm/cancel; also changes the outfit of the NPC she's talking to.
- **Shop** — the transaction flow: select → see price in coin icons → confirm → equip.
- **Chest color picker** — a 4-swatch menu on chests; speaks color names on focus ("green!"). Same grammar as the wardrobe.
- **Sticker/collection album** — possible later addition: paging + selection practice, endless genAI art sink.

## Sleep & the day cycle

- **Bed = save point, session-ender, and the clock.** Sleep → new day → world regenerates, tools relocate, family reschedules, mail arrives, **eggs hatch**.
- **The bed is the sleep trigger, on the ordinary two-press grammar** (shipped 2026-08-13): the first green press speaks Seraphina's bed line, a second green press inside the speech-bubble window sleeps. Walking off, letting the bubble expire, or pressing red all mean no — the offer lives on the balloon, so there is nothing to unset. The night runs on its own curtain drawn above the HUD, not a camera fade: the light drains out, moon and stars and drifting motes come up, and the new day is swapped in at the darkest instant. No floating "Zzz" — an unspoken letter on screen would break the all-text-speaks rule.
- **ALL quests reset on sleep, no exceptions unless a need appears** (Matt, 2026-08-12) — phase, progress, quest items and quest-granted tools all clear, faeries included; she simply re-summons them the next day. This **supersedes** the earlier ruling that quest progress persists through sleep (2 of 3 stones staying 2 of 3), and with it the reason to lean on the tool guarantee for waking mid-quest. Everything else the world holds regenerates too: chopped wood grows back, broken gems return, poked props reset. **Coins are the stated exception** — they persist over sleep (see Economy), and will be the first thing that makes a night's clearing differ from a cold start.
- **The day "ends" softly and then holds** (shipped 2026-08-13): roughly eight minutes of daylight, a two-minute ramp, and then dusk stays forever — the game never reaches night and **nothing is ever forced or interrupted**. Dusk is an exterior-only tint that stops at a cozy floor and sits UNDER every interactive element, so nothing she is asked to press is ever dimmed; lamps and torches take on a glow that is invisible in daylight, and fireflies come out. Dad calls her in **once a day** at dusk onset — one line from the front door, held until she is not being spoken to already — and that call is the entire pressure. She may go to bed at any time, or never.
- Occasionally sleep *is* a quest ("Time for bed!" from dad → coin in the morning).
- **Bedtime recap** (sneaky reading), shipped 2026-08-13 folded into the sleep sequence rather than as a separate screen: while the night sky holds, at most two "today you…" lines are spoken over the stars — faeries summoned, an errand still going, stones broken, trees chopped — always closing on Seraphina's goodnight, and the night beat stretches to fit whatever plays. A quiet day IS just the goodnight.
- Overnight hatches are the biggest reward in the game — the gentlest possible pressure toward the bedtime ritual.

## Quests

**Grammar:** a family member, a friend (or a dragon situation) presents a request — voiced aloud, with an icon, in a big speech bubble; thought-bubbles are visible from across the room so wandering has purpose. Progress always renders as *N slots filling up*. Completion = big celebration + 1 coin.

**Shipped grammar (quest engine v0, 2026-08-11/12).** A quest is multi-phase, not a two-step chain: an ordered list of phases, each carrying its own voiced instruction and its own slot row. Three goal kinds so far — **gather** (a free-order slot row; each item fills its matching slot), **travel** (arriving completes the phase), **ritual** (an ordered sequence of button presses). The offer is a thought-bubble marker visible from a distance; A-press to hear it, and the second A-press IS acceptance — there is no yes/no menu. Yellow replays the *current phase's* line, from anywhere, spoken over Seraphina in the giver's voice; a ritual never claims Yellow, so mid-sequence Yellow answers with the color it is waiting for. Current objectives carry a passive sparkle — no arrows, no minimap. A quest ends on a phase with NO instruction: that absence removes the yellow dot, restores the giver's idle lines, and blocks re-offer. One active quest at a time. Quests, quest items and quest-granted tools all reset at sleep (Matt, 2026-08-11).

**Quest #1 — the faerie summoning, now canon** (Matt, 2026-08-11). Sneak's spell book needs three magic stones. Find the hammer near the well → break a malachite, a ruby and a sapphire rock, two hammer hits each, in any order → meet Sneak at the Secret Cave → the ritual at the campfire spell circle, where he reads red → green → blue from the book one at a time and each correct press sends that gem into the fire (a wrong press fizzles and giggles, and progress never regresses) → the summoning, the biggest celebration in the game so far → three faeries follow her from then on. **The gem colors are the pad's button colors on purpose:** green/red/blue stones teach the controller's color vocabulary, the same vocabulary every dot and menu uses.

**V1 templates (swappable colors/objects make each feel infinite):**

1. **Break N colored things** — "Break 1 yellow, 1 green, 1 blue stone." Color matching + counting + pickaxe.
2. **Find all N** — "Find all 4 dragon eggs!" Hidden-object search; eggs hide behind choppable trees, in scythable grass, in other rooms (search reuses tool verbs). Found eggs hatch overnight.
3. **Recolor** — "Change all the chests out front to be green." Menu-based color picker; UI practice inside a quest.
4. **Fetch (with a grow step)** — "Bring me an apple" → water the apple tree → apple grows → deliver.
5. **Read a book to your little sister** — the reading flagship. Big two-page book UI: picture on one side, one word / three-word sentence on the other; audio reads with per-word highlighting; green button turns pages; sister reacts with delight; ends in a hug + coin. Books are 4-page genAI-generated sets — endless new content.
6. **Dragon mischief cleanup** — "Sparky got loose, put out the fires he made!" (watering can), "Digby buried your pickaxe — find it!", "Wash the muddy dragon."

**More template ideas (for iteration):**

- **Feed each dragon its favorite food** — match food to dragon by color or first-letter sound ("Mmm — Munchy wants the *m*ango").
- **Cut the grass by X's window** — scythe verb + landmark navigation.
- **Deliver the mail** — take a letter to a named family member (learning family names as words); the letter gets read aloud with highlighting on delivery.
- **Hide the dragons, dad's coming!** — shoo dragons to nests; pure comedy, cannot fail.
- **Bedtime story for the dragons** — book quest variant at the nests; dragons fall asleep page by page.
- **Egg-sitting** — keep an egg warm/pet it N times today; hatches overnight.
- **The name machine** — hunt-and-peck her (or Seraphina's, or a dragon's) name on the in-world keyboard; it appears on a sign, voiced letter-sound by letter-sound.

## Reading integration (summary of where it lives)

Deliberately ambient, never a quiz; wrong picks do something funny:

- Every menu/sign/dialog: text highlighted while spoken (the workhorse).
- Letters voiced as **sounds** + a word, everywhere.
- Dragon names on nest signs; family names on doors and mail.
- The book quests (flagship) and bedtime recap.
- Occasional "mystery request": bubble shows a first letter + three objects; voiced as "it starts with sss…".
- The keyboard name machine as a special in-world object.

## Assets

- **Primary art is the purchased Cute Fantasy RPG pack** (Kenmi) — side-loaded, never in the repo. Player, tiles, buildings, and props come from it.
- genAI image generation / custom art remains for what the pack lacks: dragons, eggs/nests, Joey, Scar, book pages, stickers.
- Best genAI sinks: outfits, stickers, book pages (image + word), dragon variants, room decorations.
- Since animation is hard, can grab stock assets from somewhere where appropriate.

## V1 scope checklist

- Two-zone scrollable world — big outside village map (Seraphina's house, Dad's shed, Joey's house, Scar's house, village hall, 3 market stalls, silo, market square, green, cave mouth, garden/farm, Mystic Woods region) + multi-room scrollable house interior — with juicy doorway transitions; Secret Cave and friends' house interiors come later.
- Seraphina + dad + Hazel + Sneak on a simple wandering schedule, each with a visible activity loop (Sneak and Hazel are placed, voiced and talking; schedules and activity loops are not built yet)
- 2–3 starter dragons with names, nests + signs, one mischief trait each; hide-from-dad behavior
- 4 tools with find/respawn rules + morning tool guarantee
- 3-slot coin HUD + tool row; outfit shop + wardrobe
- Green/red/blue/yellow button scheme with colored-dot prompts; yellow = exact quest replay + sparkle hint
- Quest templates 1–6 above with slot-row progress
- Sleep/day loop: soft day end holding at dusk, all-quests-reset-on-sleep, bedtime recap, overnight hatching
- TTS + highlight-on-speak for all text

## Open questions / iterate later

- Mechanics of the Secret (auto-hide details, siblings-in-on-it) — proposed above, not yet decided
- Whether dragons get a care loop (feeding/petting daily) beyond quests
- Sticker album as second collection system
- How many books / outfits at launch; cadence of adding genAI content
- Whether "mystery request" letter-picks land at her level or wait for v2
