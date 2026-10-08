# AGENTS.md

Instructions for AI coding agents. Humans: keep under ~150 lines. Fill every `TODO`.

## Project

TODO: one or two sentences. Details: `docs/PLAN.md`.

## Stack

- Language / runtime: TODO
- Framework: TODO
- Data store: TODO
- Package manager: TODO

## Commands

- Install: `TODO`
- Run: `TODO`
- **Check: `./check`** — lint + typecheck + tests. Same command in the hook
  and CI. A task is not done until it passes.

## Structure

```
TODO: top-level folders
docs/PLAN.md        brief, architecture, tasks, backlog, handoff
docs/DECISIONS.md   append-only one-line decisions
docs/archive/       finished features, old handoff notes
.stepzero/          high-risk patterns, risk script, kit version
```

## Conventions

- TODO: naming, layout, errors, logging.
- Match nearby files (names, structure, error handling). Don't introduce a
  new pattern in the same PR.
- Smallest change that finishes IN. Don't add functions, files, or helpers
  this task doesn't use. Reuse existing code; extract a helper only if this
  task would otherwise duplicate logic.
- Each IN / Done when behavior: a test that would fail if it broke, or say
  in the PR why not (human accepts). Don't unit-test every private helper.
- Commit subject: `T<n>: <what>` e.g. `T12: add product search API`.

## Files (history)

- `PLAN.md` **Project**: human-only, unless they change the brief.
- `DECISIONS.md`: append only; never edit or delete old lines.
- Tasks: after approval, add rows, change status, update IN/OUT. Don't rewrite
  old Done when to match the code.
- Don't edit gates (`check`, `.github/`, `.stepzero/`, `AGENTS.md`,
  `.cursorignore`) unless the approved task is about them.

## Branches and PRs

- One task = one branch = one PR into `main`.
- Name: `<type>/T<n>-<slug>` from the task (types: `feat fix refactor perf
  test docs chore ci`). The hook rejects other names.
- Start from up-to-date main: `git switch main && git pull && git switch -c <branch>`
- **Agent opens the PR** when the task is ready (`./check` passed, preflight
  READY). Don't wait for the human to say "open a PR" or `/stepzero-preflight`.
  Paste the PR URL, then stop.
- **Humans merge.** Never `gh pr merge`, merge/approve via API or UI, or
  `git merge` into `main`.

## Change, build, STOP

A new request during an active task — if unsure, stop and ask:

1. **Continue** — wording only; Done when and IN/OUT unchanged.
2. **Re-plan this task** — Done when, risk, or IN/OUT changed. Stop coding.
   `/stepzero-plan`, update `PLAN.md`, wait for approval.
3. **New task** — different feature. Park current (`blocked`) or finish it.
   `/stepzero-plan` a new T-number. Don't mix work on this branch.
4. **Cancel** — human drops it. Status `cancelled`. Leave the row.

**Build** (status is `approved` or `doing`):

- Confirm task id, branch, PLAN, DECISIONS. Stay inside IN SCOPE.
- Don't dump the whole repo. Grep for existing behavior; open the hits,
  files in IN, and 1–2 neighbors for pattern. That's the repo ↔ requirement
  check. Review (not build) reads callers of changed functions.
- No extra deps, architecture, or unrelated refactors. Notice something?
  Add it under Backlog in `PLAN.md`; don't implement.
- Loop `./check` until it passes. Don't weaken tests or checks to get there.
- Don't read, print, or commit secrets.
- When `./check` passes: re-read IN, OUT, Done when. Diff matches IN; each
  Done when item has test (or why-not) evidence. Then **open the PR** (body
  from `/stepzero-report` PR mode, status `in review`, push, `gh pr create`).
  Don't wait for a slash command. If not ready, stop — no PR.
- **Pipeline:** after this PR is **open**, the next **approved** task may
  start from up-to-date `main` if **Depends on** is none (or already merged).
  Don't wait for merge unless it needs this branch's code. Don't add that
  work onto this PR's branch. Don't pipeline a second **high-risk** task.
  Rebase onto `main` and `./check` before that next merge. Stacked PRs
  (T4 from T3's branch) only if the human asks.
- **New chat per task.** Same chat until this PR is open. `/stepzero-review`
  is always a new chat. Next task = new chat (*Start T4 from main*).

**STOP** and tell the human (don't "try your best"): secrets, data deletion,
auth/payments/permissions, prod/CI/gates, new deps, huge unrelated diffs,
requirement change (use the rules above), or `.stepzero/risk` match when the
task isn't high. Never downgrade.

## High-risk

Human reads every line, tests by hand, ships behind a flag or staging, has a
rollback. Patterns: `.stepzero/high-risk`. Run `.stepzero/risk`. A match is high.

- Gates, auth, payments, schema/migrations, data deletion, CI/deploy,
  env/secrets, new dependencies
- TODO: project paths (e.g. `src/auth/`)

## Workflow

Statuses: `draft` → `approved` → `doing` → `in review` → (`changes requested`
→ fix → `/stepzero-review` again) → `done`. Also `blocked`, `cancelled`.
`done` = a human merged.

1. **Plan** `/stepzero-plan`. Each task: Branch, IN/OUT, Done when, Risk, Depends on.
   No code. Human approves (`Approved: YYYY-MM-DD`).
2. **Build** on the task branch (rules above) until `./check` passes, then
   **open the PR**. One chat until it is open. Stop. Next independent task:
   **new chat**, from `main`.
3. **Review** — skip for **low** (copy, styling, logs, tests): human skims
   and merges. **Medium**: `/stepzero-review` in a fresh chat + human tries it.
   **High**: same, plus High-risk above, then explicit "I approve merge."
   CHANGES REQUESTED → fix → `./check` → `/stepzero-review` again. Don't merge yet.
4. **Merge** human only. Medium/high: CI green, review APPROVE, human tried it.
5. **Close** `/stepzero-report` after merge: status `done`, handoff, proposed rules
   (human approves each before they land in this file).

Start each session from this file, the latest handoff in `PLAN.md`, and
`DECISIONS.md`.
