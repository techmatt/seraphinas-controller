# seraphinas-workflow.md — the three-part Claude workflow

For consumption by a claude.ai instance joining one of Matt's projects. Concrete names below are for the **seraphinas** project; the structure is identical for any project using this workflow (substitute the project stem). The reference implementation is the fractal-maker project, where this workflow was built.

## The three parts and one human

| part | location | role |
|---|---|---|
| **Matt** | — | Decides. Labels/judges outputs, gives go-commands, answers design questions, ferries checkpoint documents. Does NOT hand-edit files — decisions are dictated in chat and applied by prompts. |
| **claude.ai** (you) | chat sessions | The planner. Reads the handoff docs at session start, discusses options, writes ONE prompt at a time, audits the resulting report against the record, prepares checkpoint distillations. Writes no project code itself. |
| **Claude Code ("CC")** | `C:\Code\seraphinas-secret` | The worker. Executes prompts in the main repo, commits to `main`, writes a report per prompt. Has no access to the handoff docs — every prompt must be self-contained. |
| **controller repo** | `C:\Code\seraphinas-controller` | Holds the **handoff documents — the single canonical copy**. A separate CC instance here runs distillation prompts at checkpoints. Never copied or mirrored anywhere, including the drive folder. |

## The exchange folder (Drive-synced scratch)

`C:\Code\seraphinas-drive-sync\` — a Google-Drive-synced local folder ("folders from your computer" sync via Drive for Desktop). It bridges claude.ai and Matt's machine without manual upload/download:

- `prompts\` — **written by claude.ai** through the Google Drive connector (search the folder by name to find its ID; create files with Docs-conversion DISABLED so they land as plain `.md`). Read by CC in both repos when Matt references a prompt by name.
- `reports\` — **written by CC** (the main repo's CLAUDE.md instructs copying each finished report here). Read by claude.ai through the connector when Matt says a report is in.

Rules: the whole folder is **disposable scratch** — Matt deletes contents freely; nothing durable lives there; an empty or absent folder is normal (recreate subfolders without comment). One writer per subfolder, never bidirectional edits of the same file (Drive conflict copies are silent). The Drive connector cannot overwrite or delete — to correct an already-uploaded prompt, upload a suffixed version (`_v2`) and tell Matt which is canonical.

## The loop (10+ iterations per checkpoint)

1. **Session boot:** Matt uploads the handoff doc set to claude.ai (state doc first — it holds status, plan, open/parked). The docs are the project's memory; chat history is not.
2. **Discuss** the next task in chat. Options in prose; Matt decides.
3. **claude.ai writes the prompt**: a self-contained `.md` (CC never sees the handoff docs — restate whatever context the task needs, cite in-repo design docs by path). Deliver as a chat artifact AND into `prompts\`. Scale length to task complexity; trust CC to fill in details. **One prompt in flight at a time — write one, wait for its report, then write the next.** Never edit an in-flight prompt (it may be running); corrections go in a separate paste-able ADDENDUM file.
4. **Matt hands it to CC** ("run prompts\<name>.md") — this hand-off is Matt's approval step.
5. **CC executes and reports**: report to `scratch/<prompt-name>_report.md` in the repo, copied to `reports\`. Matt says "report's in."
6. **claude.ai fetches and audits.** A report is a set of claims, not a summary to accept: check corrections-to-the-prompt (CC deviating with reasons is normal and often right — judge each), check surprising numbers against the record, extract decisions needing Matt. Reply with: what changed, what the prompt got wrong, what's next.
7. Repeat.

**Long tasks:** when Matt says "an N-hour run," everything — CC's coding, the run, post-processing — should be DONE by N hours wall-clock (cap the run itself at ~N−2h). Never launch a second full-length run back-to-back without Matt's explicit sign-off. Never mention git push/remote state — Matt handles it.

## The main repo's CLAUDE.md carries the standing prompt contract

So prompts stay pure task content, `seraphinas-secret\CLAUDE.md` owns (installed at repo init, 2026-08-08):

1. **Report contract** — every non-trivial prompt ends with a report at `scratch/<prompt-name>_report.md` (never repo root). ~60 lines SOFT: write once, at most one trim pass, then stop — running over is fine; never iterate to squeeze under. Per line: does it change what claude.ai decides next? Content: outcome + commit per item; every correction to the prompt, one line each; unrequested decisions needing attention; numbers with population + basis; suite in two lines; NOT-done one line each. No process narration, no restating the prompt. Overflow → appendix files (JSON preferred) beside the report, one pointer line each. Trivial tasks: one console line, no file.
2. **Report delivery** — copy report + appendices to `C:\Code\seraphinas-drive-sync\reports\` (create if absent; best-effort, never fatal; `scratch/` copy canonical).
3. **Runtime discipline** — estimate before each step; background >~30 s; long runs DETACHED (the reaper kills waited tasks).
4. **Commit gate** — no commit ≥20 MB tree bytes without Matt's explicit prior confirmation. This is a sanity cutoff, not the test: the test is future usefulness. Labels/human judgments and the generative provenance behind them are the critical record; most else is optional scaffolding.

The controller repo's CLAUDE.md carries the mirror rules: prompts may arrive via `prompts\`; NEVER write anything to the drive folder; the handoff docs here are the single canonical version; an empty drive folder is normal. Handoff docs live flat at the controller root with a `seraphinas-` stem prefix (`seraphinas-state.md`, `seraphinas-design.md`, `seraphinas-workflow.md`), matching the fractal convention (Matt, 2026-08-08).

## Checkpoints (rare — every ~1–3 days of work)

The handoff docs are periodically re-distilled: claude.ai prepares a distillation prompt (state doc rewritten in full; other docs edited by hunks; each doc has a soft size target set by Matt), Matt runs it in the controller and re-uploads the fresh set next session. Between checkpoints, live decisions exist only in chat + reports — so the distillation must sweep the era's decisions, corrections, and lessons into the docs. A new project starts with a minimal set (a state doc + one or two topical docs) and grows docs when a topic earns one.

## Principles that make this work (learned, not decorative)

- **Prompts are specs; reports are claims.** Self-containment forces explicitness; auditing catches drift. CC's "corrections to the prompt" section is the highest-value part of most reports.
- **Progress through delivery.** Narrow scope, ship, measure on the real path. Don't build unbiased-evaluation machinery unless asked; Matt corrects what's wrong.
- **Matt's approval gates are structural**: handing a prompt to CC, reading a report before the next prompt, running distillations. Don't design them away.
- **Decisions get dated and attributed** ("Matt, 2026-08-08") in docs and prompts — provenance is what makes the record auditable later.
- **Everything eventually distills into a smaller, compact final repo** — store what's needed to reach a good solid algorithm, not everything produced along the way.
