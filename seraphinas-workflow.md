# seraphinas-workflow.md — the three-part Claude workflow

For consumption by a claude.ai instance joining one of Matt's projects. Concrete names below are for the **seraphinas** project; the structure is identical for any project using this workflow (substitute the project stem). The reference implementation is the fractal-maker project, where this workflow was built.

## The three parts and one human

| part | location | role |
|---|---|---|
| **Matt** | — | Decides. Labels/judges outputs, gives go-commands, answers design questions, ferries checkpoint documents. Does NOT hand-edit files — decisions are dictated in chat and applied by prompts. |
| **claude.ai** (you) | chat sessions | The planner. Reads the handoff docs at session start, discusses options, writes ONE prompt at a time, audits the resulting report against the record, prepares checkpoint distillations. Writes no project code itself. |
| **Claude Code ("CC")** | `C:\Code\seraphinas-secret` | The worker. Executes prompts in the main repo, commits to `main`, writes a report per prompt. Has no access to the handoff docs — every prompt must be self-contained. |
| **controller repo** | `C:\Code\seraphinas-controller` | Holds the **handoff documents — the single canonical copy**. A separate CC instance here runs distillation prompts at checkpoints. Never copied or mirrored anywhere, including the drive folder. |

## The document set (five here, two in the main repo)

Split 3→5 at checkpoint 6 (2026-08-13). The controller docs are claude.ai-facing planning context; engineering detail lives in the main repo and is CC-maintained there.

- `seraphinas-state.md` — status, plan, verification ledger, watch flags, parked, open questions. Rewritten in full every checkpoint; read first at session boot.
- `seraphinas-design.md` — design canon: premise, principles, controls, characters, quests, reading, economy, menus.
- `seraphinas-world.md` — zones, layout intent, boundary rules, aesthetics guideposts, measured world facts.
- `seraphinas-workflow.md` — this doc: the three-part workflow (mostly stable between checkpoints).
- `seraphinas-systems.md` — a compact machinery index, one line per system, so prompts can be written without the state doc bloating.
- Main repo: `docs/systems.md` (how the machinery works) and `docs/engineering.md` (facts, constraints, gotchas) — CC-maintained, cited by path in prompts, never mirrored here.

## The exchange folder (Drive-synced scratch)

`C:\Code\seraphinas-drive-sync\` — a Google-Drive-synced local folder ("folders from your computer" sync via Drive for Desktop). It bridges claude.ai and Matt's machine without manual upload/download:

- `prompts\` — **written by claude.ai** through the Google Drive connector (search the folder by name to find its ID; create files with Docs-conversion DISABLED so they land as plain `.md`). Read by CC in both repos when Matt references a prompt by name.
- `reports\` — **written by CC** (the main repo's CLAUDE.md instructs writing each finished report and its appendices here directly). Read by claude.ai through the connector when Matt says a report is in.

Rules: the whole folder is **disposable scratch** — Matt deletes contents freely; nothing durable lives there; an empty or absent folder is normal (recreate subfolders without comment). One writer per subfolder, never bidirectional edits of the same file (Drive conflict copies are silent). The Drive connector cannot overwrite or delete — to correct an already-uploaded prompt, upload a suffixed version (`_v2`) and tell Matt which is canonical.

## The loop (10+ iterations per checkpoint)

1. **Session boot:** Matt uploads the handoff doc set to claude.ai (state doc first — it holds status, plan, open/parked). The docs are the project's memory; chat history is not.
2. **Discuss** the next task in chat. Options in prose; Matt decides.
3. **claude.ai writes the prompt**: a self-contained `.md` (CC never sees the handoff docs — restate whatever context the task needs, cite in-repo design docs by path). Deliver as a chat artifact AND into `prompts\`. Scale length to task complexity; trust CC to fill in details. **One prompt in flight at a time — write one, wait for its report, then write the next.** Never edit an in-flight prompt (it may be running); corrections go in a separate paste-able ADDENDUM file.
4. **Matt hands it to CC** ("run prompts\<name>.md") — this hand-off is Matt's approval step.
5. **CC executes and reports**: report written straight to `reports\` as `<prompt-name>_report.md`. Matt says "report's in."
6. **claude.ai fetches and audits.** A report is a set of claims, not a summary to accept: check corrections-to-the-prompt (CC deviating with reasons is normal and often right — judge each), check surprising numbers against the record, extract decisions needing Matt. Reply with: what changed, what the prompt got wrong, what's next.
7. Repeat.

**Reports and screenshots:** claude.ai reads reports as Drive *text* — that is the whole channel, and it is enough for almost everything. Ask for a screenshot only when the prompt names one and numbers genuinely cannot carry the answer.

**Long tasks:** when Matt says "an N-hour run," everything — CC's coding, the run, post-processing — should be DONE by N hours wall-clock (cap the run itself at ~N−2h). Never launch a second full-length run back-to-back without Matt's explicit sign-off. Never mention git push/remote state — Matt handles it.

## The main repo's CLAUDE.md carries the standing prompt contract

So prompts stay pure task content, `seraphinas-secret\CLAUDE.md` owns the standing contract in full (installed at repo init, 2026-08-08): the report contract and its ~60-line soft target, the screenshot rule, runtime discipline, and the commit gate. It is the authority and CC maintains it — read it there rather than restating it in prompts or carrying a copy here.

Era amendment (`7df5785`): **reports emit no appendix files by default** — an appendix exists only when the prompt names one. Data that CC itself will consume later goes in-repo, not to the drive. The report alone is the default.

The controller repo's CLAUDE.md carries the mirror rules: prompts may arrive via `prompts\`; NEVER write anything to the drive folder; the handoff docs here are the single canonical version; an empty drive folder is normal. Handoff docs live flat at the controller root with a `seraphinas-` stem prefix (the five listed above), matching the fractal convention (Matt, 2026-08-08). **The controller repo never writes reports** — distillation results are console lines only; report files are a main-repo concept (Matt, 2026-08-08).

## Checkpoints (rare — every ~1–3 days of work)

The handoff docs are periodically re-distilled: claude.ai prepares a distillation prompt (state doc rewritten in full; other docs edited by hunks; each doc has a soft size target set by Matt), Matt runs it in the controller and re-uploads the fresh set next session. Between checkpoints, live decisions exist only in chat + reports — so the distillation must sweep the era's decisions, corrections, and lessons into the docs. A new project starts with a minimal set (a state doc + one or two topical docs) and grows docs when a topic earns one.

## Principles that make this work (learned, not decorative)

- **Prompts are specs; reports are claims.** Self-containment forces explicitness; auditing catches drift. CC's "corrections to the prompt" section is the highest-value part of most reports.
- **Progress through delivery.** Narrow scope, ship, measure on the real path. Don't build unbiased-evaluation machinery unless asked; Matt corrects what's wrong.
- **The suite stays lean** (Matt, 2026-08-12). Exhaustive automated testing is a poor fit for a video game Matt plays on hardware, so weaker coverage is an acceptable price for speed: merge assertions into shared page boots, teleport where the walk isn't what's under test, and cut anything Matt's own playing would catch. 18 tests is the working size: the 13-test fast subset runs 3.3 min, `test:all` 6.7 min. Diets are one pass, not optimization campaigns.
- **Matt's approval gates are structural**: handing a prompt to CC, reading a report before the next prompt, running distillations. Don't design them away.
- **Decisions get dated and attributed** ("Matt, 2026-08-08") in docs and prompts — provenance is what makes the record auditable later.
- **Everything eventually distills into a smaller, compact final repo** — store what's needed to reach a good solid algorithm, not everything produced along the way.
