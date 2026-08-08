# CLAUDE.md — handoff-docs folder

## What this folder is

The `seraphinas-*.md` files at this folder's root are the handoff documents for the
Seraphina's Secret project — claude.ai's cross-session working memory, and the **single
canonical copy** of the project record. There is no `docs/` subfolder; the docs are flat
at root. The game code lives in a separate repo, `C:\Code\seraphinas-secret`.

Current set: `seraphinas-state.md` (status, decision log, open questions, plan — read
first at session boot), `seraphinas-design.md`, `seraphinas-workflow.md`. The set grows
when a topic earns its own doc.

These docs are written by claude.ai, for a future claude.ai instance. They are not
documentation and not yours to improve.

## Your role: mechanical applier

- Typical work here is a **distillation prompt** at a checkpoint: `seraphinas-state.md`
  rewritten in full, the other docs edited by hunks, sweeping the era's decisions,
  corrections, and lessons into the record.
- A hunk applies only on a VERBATIM, unique match (whitespace included). No match, or
  multiple matches → apply nothing further to that file; report which hunk failed.
- NEVER fix typos, normalize formatting, correct grammar, update numbers, or improve
  wording — anywhere, ever, including inside content you are pasting.
- NEVER create, delete, or rename `seraphinas-*.md` files unless a prompt says so.
- Size targets are SOFT: small justified overage = apply and note; dramatic overage or
  padding = stop and report; targets move only with Matt.

## Decisions carry provenance

Decisions recorded in the docs carry a date and an attribution, e.g. "(Matt, 2026-08-08)"
or "(claude.ai, 2026-08-08)".

## Wrong-instance check

If a prompt references game source code, `package.json`, Phaser, Playwright, or asks you
to build or run the game — it belongs to `C:\Code\seraphinas-secret`. Stop and say so.

## Prompts

Prompts arrive in `C:\Code\seraphinas-drive-sync\prompts\`; a local `prompts/` folder is
gitignored and may also be used. When Matt references a prompt by name, look in both.
Both are read-only input: never create, edit, delete, or rename anything inside them, and
never commit them.

## `C:\Code\seraphinas-drive-sync\` — claude.ai exchange, read-only

**NEVER write anywhere under the exchange folder** — no reports, no outputs, no copies,
nothing. Its `reports\` subfolder is not yours to write to either; Matt ferries reports
from this repo if they are needed there.

The handoff documents in this repo are never copied, synced, or mirrored into the exchange
folder or anywhere else. One version, here, only.

The folder is disposable scratch on Matt's side — it may be empty or absent at any time.
That is normal, not an error; do not recreate or repair it.

## Reports

This repo has no scratch space and writes no report files — report in the console only
(Matt, 2026-08-08).

- Per file: applied/failed, failed hunks verbatim, size overage noted. Plus every
  correction to the prompt, one line each, and unrequested decisions needing attention.
- No process narration, no restating the prompt. Per line: does it change what claude.ai
  decides next?
- Never write a report file anywhere — not in this repo, not in the drive folder.

## Git

Working tree clean before applying. One commit per distillation. Never rebase, amend, or
rewrite history.
