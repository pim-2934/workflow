---
description: Drive a ticket end-to-end through Define → Plan → Build → Verify → Review → Ship, pausing only for spec and plan sign-off.
argument-hint: <paste the ticket description here>
---

# Ticket orchestrator

You are running the full ticket lifecycle for the work described below. The ticket
text is everything in `$ARGUMENTS`. If `$ARGUMENTS` is instead a path to an existing
file (for example `docs/<id>/ticket.md`, as produced by `/intake`), read that file
and treat its contents as the ticket text. If `$ARGUMENTS` is empty, ask the user to
paste the ticket description (or a path to a ticket file) and stop.

This command sequences the agent-skills pipeline rather than reimplementing it. Each
phase reads the previous phase's artifact off disk (`docs/<id>/spec.md`, then
`docs/<id>/plan.md`, then the working-tree changes), so you do not have to carry
context between phases by hand. Drive the phases in order, and honor exactly two
human gates.

## Ticket
$ARGUMENTS

---

## Conventions (apply to every phase)

**Ticket identifier `<id>`.** Settle on it once. If `$ARGUMENTS` was a path like
`docs/<id>/ticket.md`, reuse that `<id>` exactly so every artifact stays in the
folder `/intake` already created. Otherwise derive it from the ticket: use the
tracker key (for example `PROJ-123`) if the ticket has one, or a short kebab-case
slug of the title (for example `disable-survey-close`). Use the same `<id>` for every
artifact path below.

**Artifact location.** All generated markdown lives under `docs/<id>/`, not the repo
root:
- spec: `docs/<id>/spec.md`
- plan: `docs/<id>/plan.md` (includes the task breakdown inline)
- diagram: `docs/<id>/diagram.md` (mermaid view of the change, written in Phase 6)

Create the `docs/<id>/` directory if it does not exist. When a skill would otherwise
write to its own default location (for example `/spec` and `/plan` write to the repo
root or a `tasks/` folder), tell it the `docs/<id>/` path explicitly so it writes
there instead, and pass the prior artifact's path so each phase finds its input.

**Use the context map if present.** If `docs/context/INDEX.md` exists (produced by
`/context`), read it first to get oriented, then load only the area files relevant to
this ticket, plus `seams.md` and `behavior-map.md` if the map includes them. Do not
load the whole map; flooding the context window degrades output. When the work must
align to existing behavior and a `behavior-map.md` exists, use it to jump straight to
the source of truth instead of searching the tree. If the map is missing, explore the
codebase directly and suggest the user run `/context` first.

**No version-control actions.** Never run `git commit` or `git push`, and never open
a PR or branch, at any phase, including the build loop. Leave every change in the
working tree for the user to review and commit themselves. This overrides the default
commit-per-task behavior of `/build auto`.

**No attribution.** Never attribute the work in any file,
artifact, message, or comment. No `Co-Authored-By` trailers, no "Generated with
Claude Code" lines, and no mention of AI in the spec, plan, docs, or code.

---

## Phase 0: Preflight (check required skills)

Before doing anything else, confirm that the agent-skills plugin (addyosmani/agent-skills) and the skills this
command sequences are installed. The
phases below have nothing to invoke without them.

Required skills (each phase fails closed without these):
- `spec-driven-development` (Phase 1)
- `planning-and-task-breakdown` (Phase 2)
- `incremental-implementation` and `test-driven-development` (Phase 3)
- `code-review-and-quality` (Phase 4)
- `shipping-and-launch` (Phase 5)

Conditional skills (only invoked on certain branches; warn but do not block):
- `debugging-and-error-recovery` (Phase 3, on a broken build)
- `doubt-driven-development` (Phase 3, on a high-risk or irreversible step)
- `security-and-hardening` (Phase 4, when review flags a security finding)
- `performance-optimization` (Phase 4, when review flags a performance finding)
- `code-simplification` (Phase 4, when review flags readability or complexity)

To check, inspect your available skills (the Skill tool list or MCP skill list). As a
filesystem fallback, look for the installed plugin and its skill directories:

```bash
# Find the installed agent-skills plugin, then list which skill dirs exist
find ~/.claude/plugins -type d -path '*agent-skills*/skills' 2>/dev/null
```

Then, for each required skill above, verify a matching skill is available.

- If any required skill is missing, STOP. List exactly which ones are missing and
  tell the user to install the plugin (`https://github.com/addyosmani/agent-skills`),
  for example via `/plugin`, then add the `addy-agent-skills` marketplace and install
  `agent-skills`. Do not start Phase 1.
- If only conditional skills are missing, warn the user that those fallback behaviors
  will not be available, then proceed.
- If everything is present, proceed to Phase 1 without further comment.

## Phase 1: Define (GATE 1)

1. Invoke the spec-driven-development skill (`/spec`).
2. Seed it with the ticket text above instead of asking for the objective from
   scratch. Only ask clarifying questions for genuinely missing information
   (acceptance criteria, constraints, boundaries) that the ticket does not cover.
3. Produce the spec at `docs/<id>/spec.md`.
4. STOP. Present the spec and wait for explicit sign-off ("approve", "go", "yes").
   Treat a hedged response as not approved. Do not proceed to Plan until approved.

## Phase 2: Plan (GATE 2)

1. Invoke the planning-and-task-breakdown skill (`/plan`).
2. Read `docs/<id>/spec.md` and the relevant codebase. Slice the work vertically, and
   write tasks with acceptance criteria and verification steps.
3. Produce `docs/<id>/plan.md`, with the task breakdown included inline as a checklist
   the Build phase ticks off. Do not write a separate `todo.md`.
4. STOP. Present the plan and wait for explicit sign-off. This is the last human gate.

## Phase 3: Build and verify (autonomous)

After plan approval, run `/build auto`, which runs the incremental-implementation
skill alongside test-driven-development. For each task: write a failing test (RED),
implement the minimum to make it pass (GREEN), run the full suite to catch
regressions, run the build, then mark the task complete. Verification is folded into
this loop, so every task earns a passing test.

Do not commit. Tell `/build auto` to skip its per-task commits and leave all changes
in the working tree (see the no-version-control rule in Conventions). The plan and
spec already exist under `docs/<id>/` and are approved, so `/build` must not re-spec
or re-plan. Point it at `docs/<id>/plan.md` and `docs/<id>/spec.md`, and treat the
`docs/<id>/` artifacts as the expected, already-present baseline rather than
unexpected uncommitted work.

Stop and ask the user only on a hard blocker: a test that cannot be made to pass, a
broken build with no obvious fix (use the debugging-and-error-recovery skill), an
ambiguous or uncovered decision, or a high-risk, irreversible step such as auth or
permission changes, destructive migrations, payments, deletions, deploys, or secrets
(use the doubt-driven-development skill and get explicit sign-off).

## Phase 4: Review and deep-dive escalation (autonomous)

1. Invoke the code-review-and-quality skill (`/review`) on the changes from Phase 3
   across all five axes (correctness, readability, architecture, security,
   performance). Categorize findings as Critical, Important, or Suggestion.

2. Escalate to the specialist skills the review warrants. These are the deep-dive
   skills that `code-review-and-quality` itself points to: its security axis to
   `security-and-hardening`, and its performance axis to `performance-optimization`.
   Run each one only on a relevant finding, and if the skill is missing, warn and
   continue rather than block.
   - On a security finding, run `security-and-hardening` for a focused pass.
   - On a performance finding, run `performance-optimization`. Measure before
     optimizing, and do not tune on speculation.
   - On a readability or complexity finding, run `code-simplification`, scoped only to
     the files changed in Phase 3, never refactoring untouched code. Keep it separate
     from the feature work, as that skill requires: report the simplifications in
     their own section of the summary rather than folding them into the feature
     changes, and skip it entirely when the changed code is already clean.

3. If any Critical finding appears, from the review or from an escalation, fix it
   before continuing, re-entering the build loop if needed. Carry the Important and
   Suggestion findings, including the specialist ones, into the final summary.

## Phase 5: Ship (autonomous)

Invoke the shipping-and-launch skill (`/ship`). It fans out in parallel to
code-reviewer, security-auditor, and test-engineer, then synthesizes a single GO or
NO-GO decision with a mandatory rollback plan.

- On NO-GO, report the blockers and stop.
- On GO, summarize what is ready in the working tree and hand it to the user to review
  and commit. Per the Conventions, do not commit, push, or open a PR yourself, even on
  GO and even if asked mid-run. If the user wants those steps, point them to the
  relevant git and PR commands to run themselves.

## Phase 6: Diagram the change (autonomous)

Produce a mermaid diagram of the code this ticket created or changed, written to
`docs/<id>/diagram.md` as one or two fenced ` ```mermaid ` blocks, each with a
one-line caption. This is documentation of the working-tree result, so generate it on
a GO; skip it on a NO-GO (there is nothing to document yet).

- **Scope to the change.** Diagram the code added or modified in Phase 3, plus only the
  immediate collaborators needed to make it legible. Do not map the whole codebase;
  that is `/context`'s job, not this one.
- **Pick the type that fits, do not force one.** A request or call flow → `sequenceDiagram`;
  a data model or schema change → `classDiagram` or `erDiagram`; module or control-flow
  wiring → `flowchart`; a state machine → `stateDiagram-v2`. If the change does not lend
  itself to a useful diagram (for example a pure config or copy tweak), say so and skip
  rather than draw a trivial one.
- **Bounded.** One or two diagrams, not an exhaustive set. Favor the one view that best
  explains the change to a future reader.
- **Plain markdown only.** Emit the ` ```mermaid ` block as text; GitHub and most IDEs
  render it natively. Do not call any renderer or validation tool, and do not add an
  image dependency.

## Final summary

Report the spec path (`docs/<id>/spec.md`), the plan and tasks completed, the tests
added, the files changed in the working tree (left uncommitted for the user), the
review findings by severity (including any security or performance deep dives and the
separate simplification pass), the diagram at `docs/<id>/diagram.md` (or a note that it
was skipped), and the ship decision with its rollback plan. Flag
anything you skipped or left for the user.

---

### Notes
- This command depends on the agent-skills plugin (addyosmani/agent-skills) being
  installed, since that is what the skill names above resolve to. Phase 0 enforces
  this by checking for the required skills up front and stopping with a clear,
  actionable message if any are missing, rather than failing mid-pipeline.
- Respect any per-project guardrails defined in that project's `CLAUDE.md` or memory,
  such as directories that must not be modified. Stop at the nearest gate and flag the
  conflict rather than working around it.
