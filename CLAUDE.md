# CLAUDE.md

## What this repo is
A personal suite of Claude Code slash commands (Markdown prompt specs) that orchestrate
multi-phase engineering workflows. The "source" is the command prose — there is **no
application code, no build system, and no test suite**.

- `workflow/context.md` — `/context`: generates the agent-facing codebase map.
- `workflow/intake.md` — `/intake`: interview → `docs/<id>/ticket.md`.
- `workflow/ticket.md` — `/ticket`: drives a ticket Define→Plan→Build→Verify→Review→Ship.
- `README.md` — overview + setup; `LICENSE` — MIT.

## Build / test / run
None. Validation is reading the Markdown and (manually) invoking the commands in Claude
Code. The commands are made live by symlinking/junctioning the `workflow/` folder into
`~/.claude/commands/workflow` (see `README.md:86`), which namespaces them as
`workflow:context`, `workflow:intake`, `workflow:ticket`.

## Boundaries & no-touch zones
- These commands run against *other* projects. Honor each target project's own
  `CLAUDE.md` / memory guardrails (off-limits dirs, etc.).
- **No version-control actions** and **no AI attribution** — these rules are stated in
  every command file and must hold for any edits to them too (no `Co-Authored-By`, no
  "Generated with" lines).
- `README.md` must stay *outside* `workflow/` so it is never exposed as a command.
- Shared conventions are duplicated across all command files (and the README) with no
  single source of truth — change one, change all (see `docs/context/seams.md`).

## Dependencies
External `agent-skills` plugin (`addyosmani/agent-skills`). Skill names are the coupling
surface; each command preflights its required skills.

## Context map
For anything beyond this file, read `docs/context/INDEX.md` and load only the relevant
area file. Loading the whole map degrades output. Refresh with `/context <area>` when an
area changes.
