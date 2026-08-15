# world — Seraphina's Secret

*Zones, layout, boundary rules, aesthetics, and measured world facts. Split out of the design doc at checkpoint 6 (2026-08-13). Design canon (premise, characters, quests, economy) is in seraphinas-design.md; the machinery index is in seraphinas-systems.md.*

## Zones & layout intent

- **Zone-based, not one open map.** Big scrollable zones joined by doorway transitions, each transition its own little flourish. Insides in general are big and scrollable (Matt, 2026-08-08).
- **Zone "Outside":** one big scrollable map, 72×50 tiles, holding Seraphina's house, Dad's shed, The Secret Cave, Joey's house (a mermaid friend), Scar's house (a robot friend), with **The Mystic Woods** as the far-left wooded region — not too big; special quests occasionally lead there, and a possible future "adventure mode" would live in the forest. It reads as "a small village with a garden+farm, not a farm with some sheds" (Matt, 2026-08-10/11): the farm/garden is about a quarter of the map, and the landmarks now also include a village hall (a knockable facade), 3 market stalls, a silo, a market square, and a green. The well sits on Seraphina's street; spawn is beside her front door with the interaction dot showing. All Matt-approved 2026-08-11.
- **Zone "House":** the largest indoor space — many connected rooms as ONE big scrollable map, no portals between interior rooms.
- **The map boundary must be clearly obstacle-filled** (Matt, 2026-08-10): visually a wall of stuff, mechanically no walkable route to any map-edge cell (build-gated — see the state doc). North is a mountain cliff with the Secret Cave mouth cut into the face (the Stardew mine pattern); the chamber behind it is deliberately one screen, and its **spell circle is a permanent cave landmark, not quest furniture** (Matt, 2026-08-13) — it stays after quest #1 ends, ready for the next ritual. East and south are fence lines with trees behind. West is the Mystic Woods' own density. One soft closed gap of undergrowth at the far west marks the future woods exit — it reads as a path without being walkable.
- World regenerates overnight: grass regrows, trees regrow, stones respawn, hidden objects re-roll positions. Quest furniture is NOT world data and does not regenerate with it — it is runtime-spawned and despawned (see the systems doc).

## As built

Three zones. Outside and House are generated data: `content/world/**` → `tools/world/build.ts` → JSON; map changes are data edits.

**Outside**: 72×50 tiles (3600), 941 sprites, 1197 solid cells (down from 1360 — buildings went base-only; the +1 over 1196 is campfire logs). Seraphina's house (enterable), Dad's shed, Joey's house, Scar's house, the Secret Cave mouth, plus a village hall, 3 market stalls, a silo, a market square and a green (facades, not enterable; interacting gives a wiggle + burst + two knocks, no text). The Mystic Woods hold the west; a fenced vegetable patch, pond, well and joined dirt roads fill the village. Boundary ring 210/216 solid — the 6 open cells are the named woods gap. Of 256 trees, 46 are choppable; perimeter/boundary trees never are. 29 sprites now carry `glow` (wall torches, campfire, and the exterior lamp posts) and light at dusk.

**House**: 40×29, four rooms on ONE scrolling floor plan, no internal transitions, real walls and measured furniture. The living-room sofa/rug arrangement doubles as the storytime reading nook (measured facts below).

**Secret Cave**: 20×11, exactly one screen, deliberate. Grey cobble walls + the pack's cavern floor, four wall torches, a campfire and the spell circle — which is a permanent landmark, not quest furniture (Matt, 2026-08-13). The cave mouth is a real A-press doorway with a violet light; exit is a walk-through portal. Nothing dragon-shaped exists in it yet.

**mountainPath** leaves the top road's west end and runs along the cliff foot to the cave: 16 tiles, none below row 14, wood unbroken. Trade-off accepted (Matt, 2026-08-11): the cave is not visible from the bottom of the wood. `cave_chest` keeps its name in the (kept) old clearing — the id is in the voice manifest.

## World aesthetics guideposts

*(Matt, 2026-08-10, from reference images)*

> **Interiors:** every room is built around one focal arrangement — a counter, a sofa on a rug under a lamp, a bed — with furniture hugging walls and corners, leaving generous open floor she can cross without steering. Detail lives on walls and edges (shelves, windows, jars, pictures); floors stay clear except for a rug anchoring the centerpiece. Floor-material changes (wood → stone) mark what a sub-area is for without adding walls. Interesting but not crowded: clusters of themed props with real negative space between them, and a warm, cohesive palette per room.
>
> **Exteriors:** wide, clearly drawn roads (2–4 tiles) connect every landmark, with lamp posts, signs, and flowers lining road edges and building fronts rather than scattered across open ground. Decoration clusters where it means something — flower pots at a door, hedges along a wall, a bench beside the path — and grass stays mostly open between clusters, so buildings read from a distance and walking anywhere is easy. The world should look composed, like someone arranged it — not filled.

**Interiors: ratified against a reference image and now implemented** (Matt, 2026-08-10, replacing the "flat peach" placeholder era). Rooms follow the pack-derived reference: real walls with caps/trim and run edging (seamed end tiles at run ends, doorway jambs, room joins); one floor material per room, with a rug anchoring the focal arrangement; furniture hugging walls. Arrangements are baked in — player furniture rearrangement is not planned. Built and approved 2026-08-11, with some edging still incomplete (a known cleanup item, see the state doc).

**Grass correctness rule** (Matt, 2026-08-10). A tile may sit on or beside the base outdoor grass only if its own background IS that base grass; ornamental tilesets carrying a different baked-in green are out. Transparent-background decoration on base grass is fine. The brown ploughed farmland tiles are explicitly approved — brown reads as a dug bed, not a competing green (Matt, 2026-08-11).

## Measured facts (2026-08-13)

- The Mystic Woods interior is **very open** — 93.8% walkable, one connected component. A planting pass for woods density remains a parked candidate.
- The only road-free 5×5 clearing in the woods is at **(13, 26)** — it is now the bunny pen site.
- The den is at **(6.5, 16.5)**; the quest's carrots at **(11.5, 21.5)**, **(18.5, 23.5)**, **(13.5, 36.5)**.
- The malachite stone moved to **(20.5, 35.5)**: it previously coincided with the pen's east edge — the class of collision no gate sees, because runtime quest furniture is invisible to `world:build`.
- Dad idles at **(57.44, 12.3)** — `BUILDINGS.shed.door.x + 3`, a row above the top road, facing the street: off the doorstep so the shed's green dot never competes, clear of the flower pot by a tile, inside the doorstep apron (2026-08-15).
- The reading nook (2026-08-15): the red rug spans tiles **26–28 × 6–8**; the sofa blocks (27,6)/(28,6) and the stool (27,8), leaving row 7 the clear strip. During storytime Hazel sits at **(26.5, 7.5)** facing right and the storybook lies at **(28.5, 8.1)** — two tiles apart on purpose, since interact reach is 1.53 tiles and the green dot must never hop from book to Hazel. The fetchable book spawns at **(36.5, 5.5)**, one tile off the bookshelf, same reason.
