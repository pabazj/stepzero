# stepzero

A **workflow kit** for one AI coding agent. Built and tested with **Cursor**
on **macOS**. Not an app, crew, or cloud service. Clone it, install skills
once, opt each **git** project in with `scripts/init`. Product code does not
live here.

The agent plans, codes, checks, and **opens the PR**. You approve the plan
and **merge to `main`**. You do not open the PR. `./check` (lint + typecheck
+ tests) is "done" locally, in the hook, and in CI. Memory is files
(`AGENTS.md`, `docs/PLAN.md`, `docs/DECISIONS.md`), not chat. Free tools:
Cursor, GitHub Free, git hooks. Other agents that read `AGENTS.md` /
`SKILL.md` can use it too.

**For:** small teams, opted-in repos, apps you can test. **Not for:** spikes,
monorepos (`init` is repo root only), or a fleet of agents. One task, one
chat, one folder. Worktrees are human-created.

**OS:** macOS (tested). Linux should work (less tested). Windows: Git Bash
or WSL only, not PowerShell/cmd. CI still runs on Linux.

## How it works

```text
                    YOU
         approve plan · try it · merge
                      │
     ┌────────────────┴────────────────┐
     │           stepzero              │
     │  Prompts              Machines  │
     │  /stepzero-plan       ./check   │
     │  /stepzero-preflight  hook      │
     │  /stepzero-review     CI        │
     │  /stepzero-report               │
     │  AGENTS.md                      │
     └────────────┬────────────────────┘
                  │
     ┌────────────┴────────────┐
     │  Your project (opt-in)  │
     │  PLAN.md  DECISIONS.md  │
     │  feat/T3-…  →  PR       │
     └─────────────────────────┘
```

Prompts guide. Hook and CI enforce what they can. You are the merge lock.

## Skills

Type the slash command for plan / review / close. Opening the PR is part of
**build** — it happens after `./check` even if you never type
`/stepzero-preflight`. Cursor `/plan` and `/review` are **not** these.

| Command | When | Does |
|---------|------|------|
| `/stepzero-plan` | New work or mid-task change | Tasks with Branch, IN/OUT, Done when, Risk, Depends on. **No code** until you approve. |
| `/stepzero-preflight` | Task looks done / `./check` passed | `./check`, IN/OUT, high-risk. If READY, **opens the PR**. Never merges. |
| `/stepzero-review` | Medium/high, **new chat** | Review the diff. No edits. Skip for low risk. CHANGES REQUESTED → fix → `./check` → run again. |
| `/stepzero-report` | PR, or after merge | Unmerged: PR body, status `in review`. Merged: handoff, `done`, **proposed** AGENTS.md rules (you approve). |

Other agent: `STEPZERO_SKILLS_DIR=~/.other-agent/skills scripts/install`.

## Workflow

```
/stepzero-plan → you approve → build until ./check passes
  → agent opens the PR → stop
  → /stepzero-review (new chat; skip if low) → you try medium/high → you merge
  → /stepzero-report
```

After `./check` passes the agent **must** open the PR (preflight + `gh pr
create`). Don't wait for `/stepzero-preflight` or "open a PR."

Mid-task: **continue** (same Done when) / **re-plan** this task / **new
task** (park or finish first) / **cancel**. Don't pile extra work on the
branch.

**Pipeline:** after this PR is **open**, the next *independent* task can
start from `main` in a **new chat** if Depends on is none. If it needs this
branch, wait for merge (stack only if you ask). Never put T4 on T3's branch.
High-risk: one at a time. One chat per task until that PR is open; review is
always a new chat. Build greps IN files; it doesn't dump the whole repo.
Review still reads callers of changed functions.

| Risk | Examples | Review |
|------|----------|--------|
| Low | copy, styling, logs, tests | skim; skip `/stepzero-review` |
| Medium | normal features | `/stepzero-review` + you try it |
| High | auth, payments, permissions, schema, data deletion, CI/deploy, env, new deps | you read every line, try it, flag/staging, rollback, then say you approve merge. Never downgrade. |

If scope hits a high-risk area, the agent stops and tells you.

## What's in the box

```
skills/stepzero-{plan,preflight,review,report}/   install once per machine
template/          copied by init (never overwrites existing files)
  AGENTS.md        stack, commands, change/build/STOP, workflow
  check            stub — you fill in lint + tests
  docs/PLAN.md     brief, tasks, backlog, handoff
  docs/DECISIONS.md  append-only one-liners
  .cursorignore    keep .env / keys out of AI context
  .stepzero/       high-risk patterns + risk script
  .github/         CI (`./check`, branch names, guard-main) + Dependabot
scripts/install    symlink skills → ~/.cursor/skills
scripts/init       copy template + pre-push hook
scripts/pre-push   block main / bad branch names; run ./check
check, tests/      this repo's shellcheck + smoke tests
```

## Quick start

```sh
git clone https://github.com/pabazj/stepzero.git ~/Projects/stepzero
~/Projects/stepzero/scripts/install                 # once per machine
~/Projects/stepzero/scripts/init ~/Projects/my-app  # git repo; never overwrites
```

`init` prints `skip` for files already there. Safe on existing/org repos.
Commit the generated files through a PR.

After `init`:

1. Point `./check` at your real lint / typecheck / tests.
2. Fill `TODO`s in `AGENTS.md` and the Project section of `docs/PLAN.md`.
3. Uncomment runtime setup in `.github/workflows/ci.yml`.
4. Enable your ecosystem in `.github/dependabot.yml`.
5. Add project paths to `.stepzero/high-risk` (e.g. `^src/billing/`).
6. Public or paid GitHub: protect `main`, require a PR and the `check`
   status. Private Free: no branch protection — hook + `guard-main` only.
7. Stop agents merging: require tool approval for `gh pr merge` / `git push`,
   or a token that can't merge. Optional repo var `STEPZERO_AGENT_USERS`.
8. Commit setup on `chore/T0-stepzero-setup` and open a PR. The first change
   to `main` must go through a PR or `guard-main` fails.

## Hook and CI

Pre-push runs `./check`, blocks push to `main` / `master` / the remote
default, requires `<type>/T<n>-<slug>` (`feat fix refactor perf test docs
chore ci`), and refuses a dirty tree or pushing a commit that isn't checked
out.

```sh
git config stepzero.protectedBranches "main release"
git config stepzero.branchPattern '<regex>'   # also set repo var STEPZERO_BRANCH_PATTERN for CI
```

CI: same `./check`, branch names, high-risk **warning**, `guard-main` if
`main` got a commit with no merged PR (or merged by an agent).

Keep `check` executable. CI "permission denied":
`git update-index --chmod=+x check`.

`--no-verify` skips the hook. `.cursorignore` does not stop `cat .env`.
High-risk warns; it does not block.

## Update

```sh
git -C ~/Projects/stepzero pull
```

Skills are symlinks. Re-run `scripts/install` if new skills appear. Old short
names (`plan`, `review`, …): delete those links in `~/.cursor/skills` and
install again. Re-run `scripts/init <project>` for a newer hook or new
template files; existing files are never overwritten — merge those by hand.
`.stepzero/version` is the kit commit at `init`:

```sh
git -C ~/Projects/stepzero log --oneline <that-commit>..HEAD -- template/
```

This repo's `./check` is shellcheck + `tests/smoke`. `SH=dash ./check` for a
strict POSIX shell. Needs `shellcheck` (`brew install shellcheck`).

## License

MIT. See [LICENSE](LICENSE).
