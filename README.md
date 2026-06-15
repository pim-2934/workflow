# workflow

A personal collection of Claude Code slash commands that orchestrate multi-phase
engineering workflows. The commands live here, version-controlled in one place, and are
linked into `~/.claude/commands/` so Claude Code can invoke them globally as `/<name>`.

## Available commands

| Command | File | What it does |
| --- | --- | --- |
| `/context` | [`context.md`](./workflow/context.md) | Generates durable, agent-focused context for a codebase (a lean `CLAUDE.md` plus `docs/context/`) so `/intake` and `/ticket` orient cheaply and load only the slice each ticket needs. Detects the codebase's shape and adds situational layers (regime split, seams, behavior map) only when they apply. |
| `/intake` | [`intake.md`](./workflow/intake.md) | Interviews you one question at a time to turn a vague idea into a well-formed ticket at `docs/<id>/ticket.md`. |
| `/ticket` | [`ticket.md`](./workflow/ticket.md) | Drives a ticket end-to-end through Define, Plan, Build, Verify, Review, and Ship, pausing only for spec and plan sign-off. |

> New commands go in the [`workflow/`](./workflow) folder. They are picked up
> automatically through the single folder link (see **Setup**), so there is nothing
> else to wire up.

## The context → intake → ticket flow

The commands compose into a single arc, from fuzzy idea to shipped change, with
`/context` as the cheap orientation layer underneath the other two:

```
/context  →  CLAUDE.md + docs/context/      (one-time sweep, refreshed as the code changes)
                    │  read first to orient; load only the relevant slice
                    ▼
/intake   →  docs/<id>/ticket.md  →  /ticket docs/<id>/ticket.md  →  spec → plan → build → review → ship → diagram
```

`/context` exists because running the pipeline against a large or complex codebase gets
expensive when every ticket re-explores it from scratch. It pays that exploration cost
once, durably, into an agent-focused map. `/intake` and `/ticket` then read
`docs/context/INDEX.md` and pull in only the areas relevant to the ticket, which avoids
the context flooding that degrades output. The command detects the codebase's shape and
adds situational layers only when they apply: a regime split when legacy and modern
code coexist, `seams.md` for cross-cutting integration points, and a `behavior-map.md`
(behavior to source of truth) when work must align to existing behavior. A uniform,
single-stack codebase just gets the clean area map and nothing it does not need.

`/intake` stops at the ticket on purpose. It interviews you, via the `interview-me`
skill, to capture what and why, writes the ticket, and hands off. You review and edit
the file, then run `/ticket` on it. Because `/intake` reuses `/ticket`'s `docs/<id>/`
convention, the later spec and plan land in the same folder. `/ticket` accepts
either pasted ticket text or a path to a ticket file, so the handoff is a single
command.

## How `/ticket` resolves a ticket

`/ticket` does not reimplement the engineering pipeline; it sequences skills from the
[`agent-skills` plugin by Addy Osmani](https://github.com/addyosmani/agent-skills),
installed via a Claude Code marketplace. Each phase reads the previous phase's artifact
off disk, so context is carried through files rather than the conversation:

| Phase | Skill invoked | Artifact produced | Human gate |
| --- | --- | --- | --- |
| 0. Preflight | none | verifies required skills are installed | none |
| 1. Define | `spec-driven-development` (`/spec`) | `docs/<id>/spec.md` | GATE 1: spec sign-off |
| 2. Plan | `planning-and-task-breakdown` (`/plan`) | `docs/<id>/plan.md` (task breakdown inline) | GATE 2: plan sign-off |
| 3. Build and verify | `incremental-implementation` + `test-driven-development` (`/build auto`) | code + tests (working tree) | autonomous |
| 4. Review | `code-review-and-quality` (`/review`) plus deep-dive escalation | five-axis findings | autonomous |
| 5. Ship | `shipping-and-launch` (`/ship`) | GO/NO-GO + rollback plan | autonomous |
| 6. Document | none (plain mermaid) | `docs/<id>/diagram.md` (on GO) | autonomous |

Conditional fallbacks (they warn rather than block, and fire only on a relevant
branch):
- **Phase 3:** `debugging-and-error-recovery` (broken build), `doubt-driven-development` (high-risk or irreversible step).
- **Phase 4:** the review escalates to its own deep-dive skills when a finding warrants it:
  `security-and-hardening` (security finding), `performance-optimization` (performance finding),
  and `code-simplification` (readability or complexity, scoped to changed files and reported separately).

### Conventions enforced by `/ticket`
- **Artifacts** live under `docs/<id>/` (the ticket key, for example `PROJ-123`, or a kebab slug).
- **No version-control actions.** It never commits, pushes, or opens a PR; all changes are left
  in the working tree for the human to review.

## Dependencies

- **Claude Code.**
- The **`agent-skills`** plugin by Addy Osmani
  ([`addyosmani/agent-skills`](https://github.com/addyosmani/agent-skills)). Install it via
  `/plugin`: add the `addy-agent-skills` marketplace, then install `agent-skills`. Each command runs a Phase 0 preflight. `/intake`
  needs `interview-me`, and `/ticket` needs the spec, plan, build, review, and ship skills; both
  hard-stop if a required skill is missing. `/context` leans on `context-engineering` but warns
  and proceeds if it is absent. So dependencies fail loudly rather than mid-run.

## Setup (linking commands into Claude Code)

Claude Code discovers global commands in `~/.claude/commands/`, and subdirectories there act as
namespaces. All commands live in [`workflow/`](./workflow), so you only need to symlink that one
folder. Every command in it becomes available, new commands you add show up automatically, and
the repo's `README.md`, which sits outside the folder, is never exposed as a command. The folder
name becomes the namespace, so the commands appear as `workflow:context`, `workflow:intake`, and
`workflow:ticket`. (The `workflow\workflow` in the paths below is intentional: the `workflow`
repo contains a `workflow/` command folder.) If you cloned the repo somewhere other than
`~/Projects/workflow`, point the link target at wherever it actually lives.

```powershell
# Windows, no admin needed. A directory junction works without Developer Mode:
New-Item -ItemType Junction `
  -Path   "$env:USERPROFILE\.claude\commands\workflow" `
  -Target "$env:USERPROFILE\Projects\workflow\workflow"

# Or a symbolic link, which needs Developer Mode on
# (Settings > Privacy & security > For developers) or an elevated shell:
New-Item -ItemType SymbolicLink `
  -Path   "$env:USERPROFILE\.claude\commands\workflow" `
  -Target "$env:USERPROFILE\Projects\workflow\workflow"
```

```bash
# macOS / Linux:
ln -s ~/Projects/workflow/workflow ~/.claude/commands/workflow
```

After linking, `/context`, `/intake`, and `/ticket <ticket-text-or-file>` are available in any
project, grouped under the `workflow` namespace in `/help`.

## License

[MIT](./LICENSE). The sequenced skills belong to the separate
[`agent-skills`](https://github.com/addyosmani/agent-skills) project by Addy Osmani, under
its own license.
