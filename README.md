# stepzero

A **workflow kit** for one AI coding agent. Built and tested with **Cursor**
on **macOS**. Not an app, crew, or cloud service. Clone it, install skills
once, opt each **git** project in with `scripts/init`. Product code does not
live here.

The agent may plan, code, check, and **open a PR**. You approve the plan and
**merge to `main`**. `./check` (lint + typecheck + tests) is "done" locally,
in the hook, and in CI. Memory is files (`AGENTS.md`, `docs/PLAN.md`,
`docs/DECISIONS.md`), not chat. Free tools: Cursor, GitHub Free, git hooks.
Other agents that read `AGENTS.md` / `SKILL.md` can use it too.

**For:** small teams, opted-in repos, apps you can test. **Not for:** spikes,
monorepos, or a fleet of agents. One task, one chat, one folder.

**OS:** macOS (tested). Linux should work (less tested). Windows: Git Bash
or WSL only, not PowerShell/cmd.

## How it works

```text
                    YOU
         approve plan · try it · merge
                      │
     ┌────────────────┴────────────────┐
     │  Prompts              Machines  │
     │  /stepzero-*          ./check   │
     │  AGENTS.md            hook, CI  │
     └────────────┬────────────────────┘
                  │
           your project (opt-in)
           PLAN.md → feat/T3-… → PR
```

Prompts guide. Hook and CI enforce what they can. You are the merge lock.

## Skills

Type the slash command (they do not auto-run). Cursor `/plan` and `/review`
are **not** these.

| Command | When | Does |
|---------|------|------|
| `/stepzero-plan` | New work or mid-task change | Tasks with Branch, IN/OUT, Done when, Risk, Depends on. **No code** until you approve. |
| `/stepzero-preflight` | Task looks done | `./check`, IN/OUT, high-risk. If READY, may write the PR body. Never merges. |
| `/stepzero-review` | Medium/high, **new chat** | Review the diff. No edits. Skip for low risk. CHANGES REQUESTED → fix → `./check` → run again. |
| `/stepzero-report` | PR, or after merge | Unmerged: PR body, status `in review`. Merged: handoff, `done`, **proposed** AGENTS.md rules (you approve). |

Other agent: `STEPZERO_SKILLS_DIR=~/.other-agent/skills scripts/install`.

## Workflow

```
/stepzero-plan → you approve → build until ./check passes
  → /stepzero-preflight → open PR → stop
  → /stepzero-review (new chat; skip if low) → you try medium/high → you merge
  → /stepzero-report
```

Mid-task: **continue** / **re-plan** / **new task** / **cancel**. Don't pile
onto the branch.

After a PR is **open**, the next *independent* task can start from `main` in
a **new chat** (Depends on none). Don't put it on this branch. High-risk:
one at a time. Review is always a new chat.

| Risk | Review |
|------|--------|
| Low (copy, styling, logs, tests) | skim; skip `/stepzero-review` |
| Medium | `/stepzero-review` + you try it |
| High (auth, payments, schema, CI, secrets, new deps, …) | you read it, try it, flag/staging, rollback, then say you approve merge. Never downgrade. |

## Quick start

```sh
git clone https://github.com/pabazj/stepzero.git ~/Projects/stepzero
~/Projects/stepzero/scripts/install                 # once → ~/.cursor/skills
~/Projects/stepzero/scripts/init ~/Projects/my-app  # git repo; never overwrites
```

Then: real `./check`; fill `TODO`s in `AGENTS.md` and PLAN Project; uncomment
CI runtime; enable Dependabot; add paths to `.stepzero/high-risk`; commit
setup on `chore/T0-stepzero-setup` as a PR.

MIT. See [LICENSE](LICENSE).
