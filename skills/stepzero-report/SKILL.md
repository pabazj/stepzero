---
name: stepzero-report
description: Two-phase wrap-up. If the task's PR is not merged, writes the PR description (what/why/how to test/check/risk) and sets status in review. If it is merged, writes the learn/handoff note, sets done, and proposes (never applies) AGENTS.md rules. Use when the user runs /stepzero-report, opens a PR, finishes a task, or asks for a handoff or PR description.
disable-model-invocation: true
---

# Report

Pick the mode from git/GitHub, not from the user remembering a flag:

- PR **not merged** (or no PR yet) → **PR mode** (description + `in review`).
- PR **merged** → **Close mode** (learn + `done`). Never merge it yourself.

## 1. Gather

Don't walk the whole tree. Use this turn's diff and the task card:

- `git diff --stat <base>...HEAD` and `git log <base>..HEAD --oneline`.
- The task in `docs/PLAN.md` (IN/OUT, risk, Done when).
- `./check` output tail from **this turn** if you just ran it (e.g. after
  preflight); otherwise run it once. Don't reuse a previous session.
- `gh pr view <branch> --json state,url` if `gh` works.

## 2. PR mode

Status → `in review`. Use this as the PR body (don't rewrite Project):

```markdown
## T<n> <title>

**What changed:** 1–3 sentences.

**Files:**
- `path`: what and why

**Not changed (out of scope):**
- `path` or area: why (IN/OUT)

**How to test:**
1. Exact steps, expected results.

**Human try (medium/high):**
- [ ] expected behavior
- [ ] failure case

**Risk:** low | medium | high
**Check:**
<output tail>

**Known limitations:**
```

Commit messages on the branch should look like `T<n>: <what>`. If they
don't, say so; don't rewrite history.

Unrelated ideas you noticed: add a line under **Backlog** in `PLAN.md`.
Don't implement them. Add a short handoff only if parking (`blocked`).

## 3. Close mode

Only after a human merged.

- Status → `done`.
- Handoff at the **top** of "Handoff notes":

```markdown
### YYYY-MM-DD: T<n> <title>
- State: what is done, what is not.
- Next: the next task or action.
- Watch out: gotchas, half-finished work.
```

Append new decisions: `YYYY-MM-DD: <decision>. <why>.` Never edit old lines.

Keep `PLAN.md` small:
- 3 newest handoff notes; older → `docs/archive/handoff.md`.
- When a feature's tasks are all `done` or `cancelled`, move details to
  `docs/archive/<YYYY-MM>-<feature>.md`; leave one table row pointing to it.

### Proposed rules (do not apply)

Corrections this session that would generalize. Skip anything already in
`AGENTS.md` or rejected in `DECISIONS.md`.

```markdown
### Proposed AGENTS.md rules
1. Section: Conventions | Files | Change, build, STOP | High-risk
   Rule: "<short, specific, imperative rule>"
   Because: <the correction>
```

**Do not edit AGENTS.md** until the human approves. Keep it under ~150
lines (merge or drop stale rules). Each rejected rule:
`YYYY-MM-DD: Rejected rule: "<rule>". <why>.`
