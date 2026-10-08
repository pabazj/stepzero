# stepzero

A small **workflow kit** for building software with one AI coding agent. It is
not an app, not a multi-agent crew, and not a cloud service. You clone it,
install the skills on your machine, and **opt each project in** with
`scripts/init`.

The agent may plan, write code, run checks, and **open a pull request**. A
**human** approves the plan and **merges to `main`**. One command, `./check`,
is the definition of "done" locally, in the git hook, and in CI.

It only uses free tools: any agent that reads `AGENTS.md` / `SKILL.md`
(Cursor is one), GitHub Free (Actions, Dependabot), and plain git hooks.

## What this repo is

This repository is the kit itself: reusable **skills**, a **template** copied
into your apps, and **scripts** (install, init, pre-push). Your product code
does not live here. After `init`, each project owns its own `AGENTS.md`,
`docs/PLAN.md`, `./check`, and CI.

## What it does

- Keeps project memory in files (`AGENTS.md`, `docs/PLAN.md`,
  `docs/DECISIONS.md`), not in chat history
- Runs a repeatable loop: plan → one task → one branch → prove it → PR →
  human merge → learn
- Blocks direct pushes to `main` and badly named branches; runs the same
  checks in CI
- Matches review effort to risk (low / medium / high)
- Turns corrections into proposed rules for `AGENTS.md` (you approve them)

## Who it's for

- Solo developers and small teams using an AI coding agent
- Personal or org **git** repos you choose to opt in
- Real products you will maintain (web apps, APIs, CLIs) where you can run
  lint + tests as `./check`

**Not for:** throwaway spikes, huge monorepos (one project per repo today),
or running a fleet of agents in parallel. Default is **one task, one chat,
one folder**. Worktrees are optional and human-created, not spawned by the
builder.

## Supported OS

- **macOS** — this is the tested setup (`scripts/install`, `scripts/init`,
  `./check`).
- **Linux** — same POSIX scripts; should work, less tested.
- **Windows** — not native PowerShell/cmd. Use **Git Bash** or **WSL**, then
  the same `sh` commands. GitHub **CI** still runs on Linux for every project.

## Architecture

```text
                    YOU
         approve plan · try it · merge
                      │
     ┌────────────────┴────────────────┐
     │           stepzero              │
     │                                 │
     │  Prompts            Machines    │
     │  /stepzero-plan     ./check     │
     │  /stepzero-preflight   hook     │
     │  /stepzero-review      CI       │
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

Prompts guide the agent. The hook and CI enforce what they can. You remain
the merge lock.

## Principles

1. **Context lives in files, not chats.** `AGENTS.md`, `docs/PLAN.md`, and
   `docs/DECISIONS.md` are the memory. Any new chat or tool can pick up from them.
2. **Plan before code.** The agent proposes; a human approves before building.
3. **Machines decide "done".** One command, `./check` (lint + typecheck +
   tests), runs the same locally, in the pre-push hook, and in CI.
4. **Review depth matches risk.** Low, medium, high (see below).
5. **Every correction becomes a rule.** The agent proposes a rule for
   `AGENTS.md`; a human approves it.

Prompts guide; permissions and checks enforce.

## The workflow (per feature)

```
/stepzero-plan ─► human approves (Approved: date, IN/OUT on each task)
  ─► build on feat/T3-login-form until ./check passes
  ─► /stepzero-preflight ─► /stepzero-report (PR body, status: in review) ─► open PR ─► stop
  ─► /stepzero-review in a fresh chat (skip for low risk)
       CHANGES REQUESTED? fix → ./check → /stepzero-review again
  ─► human tries it (medium/high) ─► human merges
  ─► /stepzero-report (close: done, handoff, proposed rules)
```

A new request mid-task: **continue** (same Done when), **re-plan** this task,
**new task** (park or finish first), or **cancel**. Don't pile extra features
onto the current branch. Low risk (copy, styling, logs, tests): tiny plan →
build → check → human merge; skip fresh-chat review.

**Pipeline:** you don't have to wait for merge to start the next *independent*
task. After T3's PR is open, a **new chat**: Start T4 from `main` if Depends
on is none. If T4 needs T3, wait for merge (or stacked branches only if you
ask). Never put T4 on T3's branch. High-risk: one at a time.
**Chats:** one chat per task until that PR is open; review is always a new chat.

### Risk tiers

| Tier | Examples | Review |
|------|----------|--------|
| Low | copy, styling, logs, tests | skim; skip fresh-chat `/stepzero-review` |
| Medium | normal features | `/stepzero-review` + you try it |
| High | auth, payments, permissions, schema/migrations, data deletion, CI/deploy, env/config, new dependencies | you read every line, try it, flag/staging, rollback. Then you say you approve merge. Never downgrade. |

If a task's scope grows into a high-risk area, the agent stops and tells the human.

## What's in the box

```
skills/                 agent skills (SKILL.md), installed once per machine
  stepzero-plan/        new work or mid-task change; IN/OUT; no code until approved
  stepzero-preflight/   ./check, Done when, IN/OUT, high-risk scope
  stepzero-review/      fresh chat; re-run after CHANGES REQUESTED; no edits
  stepzero-report/      PR body before merge; learn/handoff/rules after merge
template/               copied into each project by scripts/init
  AGENTS.md             stack, commands, change/build/STOP, high-risk, workflow
  check                 the single check command (stub, fails until you edit it)
  docs/PLAN.md          project, architecture, tasks, backlog, handoff notes
  docs/DECISIONS.md     one-line dated decisions (and rejected rules)
  .cursorignore         keeps .env, keys, and secrets out of AI context
  .stepzero/high-risk   high-risk path patterns (edit per project)
  .stepzero/risk        lists changed files matching those patterns
  .github/workflows/ci.yml   ./check on PRs and main; checks branch names;
                             flags high-risk PRs; fails loudly if main gets a
                             commit without a merged PR (or merged by an agent)
  .github/dependabot.yml     weekly dependency update PRs
scripts/
  install               symlink skills into ~/.cursor/skills
  init                  copy template into a project + install pre-push hook
  pre-push              block push to main and badly named branches; run ./check
check, tests/           stepzero's own check (shellcheck + smoke tests)
```

## What is enforced vs. guided

Skills and `AGENTS.md` are prompts: they guide the agent, they don't stop it.
Be clear about which layer you're relying on.

| Rule | Enforced by | Can be bypassed? |
|------|-------------|------------------|
| `./check` passes before push | pre-push hook | yes: `git push --no-verify` |
| `./check` passes on every PR | CI | no, but merging a red PR needs branch protection |
| No direct push to main | pre-push hook; CI `guard-main` job | hook: yes. `guard-main` only fails *after* the push |
| One branch per task, named `<type>/T<n>-<slug>` | pre-push hook; CI `branch-name` job; `preflight` | hook: yes. CI job fails the PR |
| Only a human merges | `AGENTS.md` + skills; your tool permissions; `guard-main` with `STEPZERO_AGENT_USERS` | see [Only humans merge](#only-humans-merge) |
| High-risk paths get flagged | `.stepzero/risk` in `/stepzero-preflight` / `/stepzero-review`; CI `risk` job | it warns, it doesn't block |
| Secrets stay out of AI context | `.cursorignore` | yes: an agent can still `cat .env` in a terminal |
| Plan first, no scope creep, don't edit gates | skills + `AGENTS.md` | yes: prompts only |
| Mid-task change (stop / re-plan / new T / cancel) | `AGENTS.md` + `/stepzero-plan` | yes: prompts only |
| Behavior change has a real test | `/stepzero-preflight` + `/stepzero-review` | yes: prompts; `./check` only as good as the tests |

Two honest caveats:

- **GitHub Free doesn't offer branch protection on private repos.** On a
  public repo (or a paid plan), protect `main` and require the `check` status:
  that's the only gate a push can't skip. On a private Free repo, stepzero
  can't make that guarantee. The hook and `guard-main` make mistakes loud,
  but not impossible.
- **`.cursorignore` isn't a permission system.** For real protection, use your
  agent tool's own controls: require approval for terminal commands (or
  allowlist them), and keep production secrets off your dev machine.

### Only humans merge

An agent in your terminal uses *your* git and GitHub credentials, so to GitHub
its merge looks exactly like yours. A rule in `AGENTS.md` alone can't stop
that. Layer these, strongest first:

1. **Don't let the agent run merge commands unattended.** In your agent tool,
   require approval for terminal commands, or never allowlist `gh pr merge`,
   `gh api`, or `git push`. This is the control that matters most.
2. **Give the agent a token that can't merge** (recommended for org repos).
   Create a [fine-grained token](https://github.com/settings/personal-access-tokens)
   for the repo with **Contents: Read and write** (push branches) and
   **Pull requests: Read** (it can't merge, approve, or open PRs; you open
   them). Use it only in the agent's environment. It could still push to
   `main` directly, but the hook and `guard-main` catch that.
3. **Detect agent merges after the fact.** If agents use their own GitHub
   account (e.g. a bot account or GitHub App), set the repo variable
   `STEPZERO_AGENT_USERS` to those logins (Settings → Secrets and variables →
   Actions → Variables). `guard-main` then fails any push to `main` whose PR
   was merged by one of them.
4. **Public repo or paid plan:** a branch rule on `main` that requires a PR
   and the `check` status. If agents have their own account, also require an
   approving review: an agent can't approve its own PR.

## Quick start

```sh
git clone https://github.com/pabazj/stepzero.git ~/Projects/stepzero
~/Projects/stepzero/scripts/install          # once per machine
~/Projects/stepzero/scripts/init ~/Projects/my-app   # once per project
```

`install` links skills into `~/.cursor/skills`. For another tool, point it
elsewhere: `STEPZERO_SKILLS_DIR=~/.other-agent/skills scripts/install`.

## Using it in a project

stepzero is **opt-in per project**. Nothing changes in a repo until you run
`init` on it, and `init` never overwrites existing files. It prints `skip`
for anything already there, so it's safe on existing and org repos. Commit
the generated files like any other change (through a PR).

After `init`:

1. Edit `./check` to run your real lint, typecheck, and test commands.
2. Fill in the `TODO`s in `AGENTS.md` and the Project section of `docs/PLAN.md`.
3. Uncomment the runtime setup block in `.github/workflows/ci.yml`.
4. Enable your package ecosystem in `.github/dependabot.yml`.
5. Add project-specific paths to `.stepzero/high-risk` (e.g. `^src/billing/`).
6. Public repo or paid plan: on GitHub, add a branch rule for `main` that
   requires a PR and the `check` status. Private repo on Free: see the caveats
   above.
7. Decide how you'll stop agents from merging (see
   [Only humans merge](#only-humans-merge)).
8. Commit the setup on a branch such as `chore/T0-stepzero-setup`, push it,
   and open a PR. The first change to main has to go through a PR too, or
   `guard-main` will (correctly) fail.

Then per feature, **type the slash command** (skills load only when invoked):
`/stepzero-plan` (approve IN/OUT), build on the task branch,
`/stepzero-preflight` (if READY, that turn may include the PR body), open a PR,
`/stepzero-review` in a new chat (skip if low risk), you try medium/high work,
you merge, `/stepzero-report` to close. Agents grep and open IN files rather
than reading the whole repo; review still reads callers of changed functions.

The pre-push hook:

- Protects `main`, `master`, and the remote's default branch. To choose a
  different set, run `git config stepzero.protectedBranches "main release"`.
- Requires branch names like `feat/T3-login-form`
  (`<type>/T<n>-<slug>`, types `feat fix refactor perf test docs chore ci`).
  To use another convention, run `git config stepzero.branchPattern '<regex>'`
  and set the same regex in the repo variable `STEPZERO_BRANCH_PATTERN` for
  CI.
- Refuses to push when you have uncommitted changes, or when the commit you're
  pushing isn't the one checked out, because `./check` would be testing
  different code.

### Limitations

- **One project per repo.** `init` needs the repo root, so monorepos with
  several projects aren't supported yet.
- **Keep `check` executable.** If the executable bit gets lost (common on
  Windows or after some copies), CI fails with "permission denied". Fix it with
  `git update-index --chmod=+x check`.
- Cursor's `/plan` (Plan mode) and `/review` (Bugbot) are not these skills.
  Use `/stepzero-plan`, `/stepzero-preflight`, `/stepzero-review`,
  `/stepzero-report`.

## Updating

```sh
git -C ~/Projects/stepzero pull
```

Skills are symlinks, so they update immediately. Re-run `scripts/install` if
new skills were added. If you previously installed the short names (`plan`,
`review`, …), remove those symlinks from `~/.cursor/skills` and run
`scripts/install` again. Re-run `scripts/init <project>` to pick up an updated
pre-push hook (it only replaces hooks it installed) or newly added template
files. Template files you already have are never touched, so merge any
template improvements by hand.

Each project records the stepzero commit it started from in
`.stepzero/version`. To see what has changed in the template since then:

```sh
git -C ~/Projects/stepzero log --oneline <that-commit>..HEAD -- template/
```

## Developing stepzero

`./check` runs `shellcheck` and the smoke tests in `tests/smoke`, which use
temporary directories only. CI runs the same command. Use `SH=dash ./check`
to run the scripts under a strict POSIX shell. You'll need `shellcheck`
installed (`brew install shellcheck`).

## License

MIT. See [LICENSE](LICENSE).
