---
name: roadmap
description: Build a roadmap for work that will not finish in one session — multi-PR work, parallel agents, anything that would otherwise be handed off. Use when the user asks for a roadmap or a plan spanning several PRs/sessions, or says /roadmap. Not for single-session tasks, however many files they touch.
argument-hint: "[work name, e.g. question-pipeline]"
---

# Roadmaps

## When

Build a roadmap for work that will not finish in one session — multi-PR work,
parallel agents, anything the user would otherwise hand off. Not for single-session
tasks, however many files they touch. Always when asked for one.

## How to produce one, in order

1. **Verify the facts in the environment first.** Never ask the user for something
   you can look up. Quote the real number in the roadmap.
2. **Grill:** one question at a time, each with your recommended answer, deepest
   dependency first. Never batch questions.
3. **Write the roadmap.** Every answer becomes a numbered decision row.
4. **No code until a stage is agreed.**

## Where

`<project>/.claude/plans/<work-name>/ROADMAP.md`, gitignored — add `.claude/plans/`
to the project's `.gitignore`. Because it is untracked, an agent in a git worktree
cannot see it: brief it by pasting requirements, never by path.

When every stage is `merged`, the roadmap moves to
`<project>/.claude/plans/finished/<work-name>/`. Nothing under `finished/` is read
again — not to list, search or gather context — unless the user names it. Before
creating a roadmap, check that `<work-name>` is not already taken there.

## Mandatory sections

- **Context** and an explicit scope statement, including what this roadmap does
  *not* cover.
- **Stage table** — `# | Stage | Deps | Par | Status | PR`. Statuses: `todo` →
  `in progress` → `merged`.
- **Decisions table** — `D1..Dn`, each row the choice *and the why*, one line.
  Settled: changing one means editing the row and recording why.
- **Per-stage requirements**, numbered — requirements, never an implementation
  plan; the detail is decided when the stage starts, because requirements move as
  the work proceeds. Scope does not move.
- **"Done when"** per stage: runnable or observable. Never "the module works".
- **Follow-ups / out of scope** — one line each, no design.
- **Working agreements** (below), repeated in every roadmap.
- **Sessions** — `Stage | Date | Session ID`, one row per session that closed or
  worked a stage, so the conversation can be resumed with
  `claude --resume <session id>`. Filled by `/continue-roadmap` when a stage
  closes; the ID comes from `$CLAUDE_CODE_SESSION_ID`, never guessed.
- A sibling **`CONTINUE.md`** carrying this work's gotchas and known-correct
  failures.

## Working agreements every roadmap states

- Update the stage row (status + PR link) and the project's implementation log in
  the same PR that lands the stage.
- Code ships as a PR the user approves. No direct pushes to `main`.
- Name the next stage and what it involves, then **wait for the user's go-ahead**
  before writing code.
- Known-correct failures are listed explicitly and never "fixed" — designed
  behaviour that looks like a bug gets written down.

## Facts

Every constraint in a roadmap cites something actually checked. Anything not
verified is labelled `ASSUMPTION`.

## Resume

`/continue-roadmap` (this plugin) finds the roadmap, verifies the table against the
repo, proposes the next unblocked stage and waits. When it closes the last stage,
it archives the roadmap under `.claude/plans/finished/`.

## Skeleton

```markdown
# <Work name> — Roadmap

## Context
<why this work exists, verified facts with real numbers>

**In scope:** …
**Not in scope:** …

## Stages
| # | Stage | Deps | Par | Status | PR |
|---|-------|------|-----|--------|----|
| 1 | … | — | no | todo | |

## Decisions
| # | Decision | Why |
|---|----------|-----|
| D1 | … | … |

## Stage 1 — <name>
**Requirements**
1. …

**Done when:** <command to run or behaviour to observe>

## Follow-ups / out of scope
- …

## Sessions
| Stage | Date | Session ID |
|-------|------|------------|

## Working agreements
- Update the stage row (status + PR link) and the implementation log in the same PR that lands the stage.
- Code ships as a PR the user approves. No direct pushes to `main`.
- Name the next stage and what it involves, then wait for go-ahead before writing code.
- Known-correct failures are listed in CONTINUE.md and never "fixed".
```
