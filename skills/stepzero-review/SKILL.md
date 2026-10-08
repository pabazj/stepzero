---
name: stepzero-review
description: Reviews a branch or PR diff in a fresh chat against docs/PLAN.md and docs/DECISIONS.md. Looks for bugs, security issues, edge cases, duplication, performance, tests, and IN/OUT violations. Depth matches risk. Findings ranked must / should / nice. After CHANGES REQUESTED, the author fixes and this skill runs again before merge. Use when the user runs /stepzero-review or asks for a stepzero code review of a branch or PR.
disable-model-invocation: true
---

# Review

You are a reviewer, not the author. Run in a **fresh chat**. If this chat
built the change, say so and stop until a new chat is started. For
medium/high, recommend a different model than the author.

**Do not** edit files, commit, push, merge, change tests, or edit
`docs/PLAN.md` / `docs/DECISIONS.md`. Report only. If the task is **low**
risk, say the workflow allows a skim and keep the review short.

## 1. Gather

This is a new chat: read `AGENTS.md`, this task in `docs/PLAN.md` (IN/OUT,
Done when, risk), `docs/DECISIONS.md`. Don't load `docs/archive/` or the
whole tree.

- Diff: `git diff <base>...HEAD` (base is usually `main`).
- Callers of **changed** functions (grep + open those files). That's the
  repo cross-check. `.stepzero/risk <base>` if present.

## 2. Depth

Risk script hits or a high-risk path → treat as **high** and say so.
Gate edits (`check`, `.github/`, `.stepzero/`, `AGENTS.md`, `.cursorignore`)
or removed/skipped tests → **must fix** unless the task is about them.

- **low**: obvious bugs; skim.
- **medium**: everything in §3.
- **high**: line by line, plus authz, validation, secrets, reversible
  migrations, data-loss paths, rollback, flag or staging.

## 3. Check

- Matches IN/OUT and Done when; no extra feature; no contradicted decision.
- Smallest change: no unused helpers or new patterns next to existing ones.
- Each IN / Done when behavior has a test that would fail without it, or a
  stated reason. Don't demand a test per private helper.
- Bugs, security, edge cases, duplication, performance (see usual lists).

## Output

```markdown
## Review: <branch or PR>  (risk: <tier>)

### Must fix
1. `path:line`: problem. Why. Suggested fix.

### Should fix
1. ...

### Nice to have
1. ...

### Verdict
APPROVE | CHANGES REQUESTED. One sentence.
```

"Must" = bug, security hole, or plan/IN-OUT violation. Empty section: "None."

## After CHANGES REQUESTED

Tell the human: set status `changes requested`; author fixes **only** those
musts (and agreed shoulds); run `./check`; run **this skill again** in a
fresh chat. Do not merge until a later `/stepzero-review` says APPROVE.
