---
name: continue-roadmap
description: Resume work from a plans roadmap — verify state, pick the next unblocked stage, wait for go-ahead. Use when the user says /continue-roadmap, "continue the roadmap", "what's next on the roadmap", or asks to resume multi-session work tracked in .claude/plans/.
argument-hint: "[roadmap name, e.g. question-pipeline]"
---

Resume work from a roadmap in `.claude/plans/`.

## 1. Find the roadmaps

```bash
ls -d .claude/plans/*/ 2>/dev/null | grep -v '/finished/$'
```

**Never read anything under `.claude/plans/finished/`** — not to list, not to
search, not for context. Finished roadmaps are archived there precisely so they
stay out of the agent's context. Only open one if the user names it explicitly.

For each `ROADMAP.md` found, read its stage table and note: the roadmap's title,
how many stages are `todo` / `in progress`, and which stage is next unblocked (the
first `todo` whose deps are `merged`).

A roadmap with no `todo` and no `in progress` rows is finished but was never
archived — exclude it from the choice and offer to archive it (see step 8).

## 2. Choose which one

- `$ARGUMENTS` given → match it against the directory names (substring, case
  insensitive). One match: use it. No match: say which roadmaps exist and stop.
- Nothing given, exactly one roadmap has pending stages → use it, say which.
- Nothing given, several have pending stages → **ask with AskUserQuestion**: one
  question, one option per roadmap, each labelled with the roadmap name and
  described by its next unblocked stage plus how many stages remain. Do not guess.
- No roadmaps at all → say so and stop. Do not invent one.

## 3. Load the context

Read, in this order: the chosen `ROADMAP.md`, its sibling `CONTINUE.md` if one
exists (it carries project-specific gotchas and known-correct failures — respect
them), and the repo's `CLAUDE.md`.

Then load the `roadmap` skill from this plugin. Its rules keep applying while you
work a stage, not only when a roadmap is written: verify facts before asking, label
anything unverified `ASSUMPTION`, and brief worktree agents by pasting requirements,
never by path.

## 4. Verify the state against the repo

The stage table is hand-updated and can be stale. Check before trusting it:

```bash
git log --oneline -15
git branch -a
gh pr list --state all -L 10
```

Plus whatever the project uses to run its tests and its CLI. **If the table
disagrees with the repo, say so explicitly** rather than silently picking a side.

## 5. Propose, then wait

State: which roadmap, which stage, what its requirements are in a few lines, and
anything the verification turned up. **Then stop and wait for go-ahead before
writing code.** If two unblocked stages are both marked parallel-safe, say so and
let the user choose.

## 6. Rules while working

- The roadmap's decisions table is **settled**. Do not re-litigate it, do not
  silently deviate. If one looks wrong once you are in the code, say so in a
  sentence or two and keep building under it; changing one means editing that table
  and recording the reason where the roadmap says the log lives.
- Stage entries are **requirements, not plans** — the implementation is yours. If a
  requirement turns out wrong, propose the change and update the roadmap in the
  same PR.
- Build nothing listed under the roadmap's follow-ups or outside its stated scope.
- Honour the repo's own rules from `CLAUDE.md` over anything general you would
  otherwise assume.

## 7. Closing a stage

Update the stage's row (status + PR link), append the log entry the roadmap
requires, and state plainly anything you left out or deferred.

Right after marking the stage `merged`, record this session so the conversation
can be resumed later. Read the ID from the environment:

```bash
echo "$CLAUDE_CODE_SESSION_ID"
```

Append a row to the roadmap's **Sessions** table (create the section at the end of
`ROADMAP.md` if it is missing): `| <stage #> | <YYYY-MM-DD> | <session id> |`. One
row per session that worked the stage — a stage that took two sessions gets two
rows. Never guess the ID: if the variable is empty, write `unknown` and say so.
Resume with `claude --resume <session id>` from the same directory the session ran
in — sessions are stored per project path, so one started inside a worktree is
only found from that worktree's path.

## 8. Closing the roadmap

When the stage you just closed was the last one (no `todo`, no `in progress` rows
left), archive the whole roadmap directory — `ROADMAP.md`, `CONTINUE.md` and
anything else in it:

```bash
mkdir -p .claude/plans/finished
mv .claude/plans/<work-name> .claude/plans/finished/<work-name>
```

If `finished/<work-name>` already exists, stop and ask instead of overwriting.
Say that you archived it and where. Follow-ups listed in the roadmap do not move
into a new roadmap on their own — mention them so the user can decide.

## Note for worktree agents

Roadmaps under `.claude/plans/` may be gitignored, so a git worktree will not
contain them. If the files are missing, ask for the stage requirements to be
pasted rather than reconstructing them from the code.
