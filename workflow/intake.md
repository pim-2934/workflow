---
description: Interview the user one question at a time to turn a vague idea into a well-formed ticket, written to docs/<id>/ticket.md for /ticket to consume.
argument-hint: <optional: a rough idea, problem statement, or paste of raw notes>
---

# Ticket intake (interview)

You are turning a rough, possibly half-formed idea into one well-formed ticket that
`/ticket` can later drive through its full pipeline (Define, Plan, Build, Verify,
Review, Ship). Your only deliverable is a single artifact on disk:
`docs/<id>/ticket.md`.

This command sequences the interview-me skill rather than reimplementing it, and it
stops at the ticket on purpose. The seed material, if any, is everything in
`$ARGUMENTS`: it may be empty, a one-line idea, or a paste of raw notes.

## Seed
$ARGUMENTS

---

## Scope: what this command does and does not do

**Does:** draw out the user's real intent through a one-question-at-a-time interview,
then write a clear ticket to `docs/<id>/ticket.md`.

**Does not:** write a spec, plan, tasks, or code; commit or open a PR; or invoke
`/ticket`. Those come later. In particular, do not drift into a specification. Stay at
ticket level, which is the what and the why, with high-level acceptance criteria.
Writing the formal, testable spec is `/spec`'s job inside `/ticket` Phase 1, and
duplicating it here only makes that phase re-ask the same questions.

---

## Conventions

**Ticket identifier `<id>`.** Settle on it once the interview has converged on a
working title. Use the tracker key (for example `PROJ-123`) if the user has one, or a
short kebab-case slug of the agreed title (for example `disable-survey-close`). Use
this same `<id>` for the artifact path. It matches `/ticket`'s `<id>` convention, so a
later `/ticket` run reuses the same `docs/<id>/` folder for `spec.md` and `plan.md`.

**Artifact location.** The ticket lives at `docs/<id>/ticket.md`. Create the
`docs/<id>/` directory if it does not exist.

**Use the context map if present.** If `docs/context/INDEX.md` exists (from
`/context`), skim it before interviewing. Knowing the codebase's areas, and where
similar behavior already lives, lets you ask sharper questions and avoid asking what
the code already answers. Load only what is relevant, and do not flood the context.

**No version-control actions.** Never run `git commit` or `git push`, and never branch
or open a PR. Leave the new file in the working tree for the user.

**No attribution.** Never attribute the work in the ticket, in
any message, or in any comment. No `Co-Authored-By` trailers and no "Generated with"
lines.

---

## Phase 0: Preflight (check required skill)

Confirm the interview-me skill (from the agent-skills plugin, addyosmani/agent-skills)
is available, since this command has nothing to drive without it. Inspect your
available skills (the Skill tool or MCP skill list); as a filesystem fallback, look
for the installed plugin:

```bash
find ~/.claude/plugins -type d -path '*agent-skills*/skills/interview-me' 2>/dev/null
```

- If interview-me is missing, STOP. Tell the user to install the plugin
  (`https://github.com/addyosmani/agent-skills`), for example via `/plugin`, then add
  the `addy-agent-skills` marketplace and install `agent-skills`. Do not start the
  interview.
- If present, proceed to Phase 1 without further comment.

## Phase 1: Interview (interview-me)

Invoke the interview-me skill and follow its rules:

1. If `$ARGUMENTS` has seed material, start from it and never re-ask what the user
   already told you. If it is empty, open by asking what they want to build or fix.
2. Ask one question at a time, each with your current best guess attached and a
   running confidence estimate. Never batch several questions together.
3. Keep going until you reach roughly 95% confidence in the underlying intent, the
   point where you can predict the user's answers before they give them.

Across the interview, surface enough to write a real ticket and no more:
- **Problem and context:** what is wrong or missing, and why it matters now.
- **Who it is for:** the affected user or system.
- **Desired outcome:** what done looks like, as observable behavior.
- **Scope:** what is explicitly in, and what is explicitly out.
- **Constraints and guardrails:** deadlines, technical limits, and anything the
  project's `CLAUDE.md` or memory marks as off-limits.
- **Risks and unknowns:** anything irreversible or genuinely open.

Stay at ticket level throughout: capture what and why, not how.

## Phase 2: Restate and confirm (GATE)

Following interview-me, once you are confident, restate your understanding in plain
language and ask the user to confirm it before you write anything. Treat a hedged
response as not confirmed. Do not write the ticket until the user explicitly approves
the restatement.

## Phase 3: Write the ticket

Write `docs/<id>/ticket.md` as the kind of clear ticket a person would file, not a
spec. Use this structure:

```markdown
# <Title>

## Problem / context
<why this, why now>

## Goal / desired outcome
<what done looks like, as observable behavior>

## Scope
**In:** <…>
**Out:** <…>

## Acceptance criteria
- <high-level, outcome-focused>

## Constraints & guardrails
- <deadlines, tech limits, off-limits areas>

## Open questions / assumptions
- <anything still unresolved or assumed>
```

## Final summary and handoff

Report the artifact path (`docs/<id>/ticket.md`) and a one-line recap of the ticket.
Then tell the user the next step: review and edit the file, then run
`/ticket docs/<id>/ticket.md` to drive it through the full pipeline. Do not run
`/ticket` yourself.
