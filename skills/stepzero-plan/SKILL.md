---
name: stepzero-plan
description: Plans a feature or change before any code is written. Handles new work and mid-task changes (continue, re-plan, new task, cancel). Reads project context, asks if unclear, proposes an approach with alternatives, and breaks work into small tasks with Branch, IN/OUT, Done when, Risk, and Depends on. Use when the user runs /stepzero-plan, changes scope, parks work, or starts a feature. Never writes code.
disable-model-invocation: true
---

# Plan

Produce a plan the human can approve. **Do not write or edit code.** The only
files you may edit after approval are `docs/PLAN.md` and (append-only)
`docs/DECISIONS.md`. Never rewrite the Project section unless the human
changes the brief. Never edit old decision lines.

## 0. Request during an active task

If a task is `doing`, `in review`, or `changes requested`, classify. If
unsure, stop and ask.

1. **Continue** — wording only; Done when and IN/OUT unchanged. Tell the
   human to keep building. Do not edit PLAN.md.
2. **Re-plan this task** — Done when, risk, or IN/OUT changed. Tell them to
   **stop coding**. Propose the updated task (same T-number and branch).
   After approval: update that task in PLAN.md; set status `approved`.
3. **New task** — different feature. Tell them to **stop** mixing it in.
   Park the current task (`blocked`) or let them finish it. Propose a new
   T-number. After approval: add the task; add a one-line backlog item if
   they want it later instead.
4. **Cancel** — human drops it. After they confirm: status `cancelled`;
   leave the row. Don't delete history.

A brief change (in/out of scope for the whole product): update Project only
after they approve, and append a `DECISIONS.md` line.

## 1. Read context

Don't load the whole repo or `docs/archive/` unless you need the next T-id
(highest `T<n>`). Read: `AGENTS.md`, current `docs/PLAN.md` (latest handoff,
Backlog, tasks), `docs/DECISIONS.md`. Grep for existing behavior; open the
hits and files in likely IN — that's the repo check. If the request
conflicts with a decision, say so.

## 2. Ask if unclear

If goal, scope, or acceptance are ambiguous, ask before planning. Prefer a
few sharp questions. List assumptions.

## 3. Propose

```markdown
## Goal
One or two sentences. What is in scope, what is explicitly out.

## Approach
The recommended approach and why.

## Alternatives
- Option B: trade-off, why not chosen.
  (Skip this section when risk is clearly **low**: copy, styling, logs, tests.
  If unsure, include alternatives.)

## Tasks
### T1: <short title>
- Branch: <type>/T1-<slug>
- IN: <what this task includes>
- OUT: <nearby work this task must not do>
- Done when: <checkable outcome>; `./check` passes
- Risk: low | medium | high (reason)
- Depends on: none | T<n>

## Open questions
```

## Task rules

- Small: one focused session, one reviewable diff.
- One task = one branch = one PR. Never reuse T-ids.
- Set **Depends on** honestly. `none` means the next task may start from
  `main` as soon as this PR is open. If it needs this task's unmerged API/UI,
  depend on it (build waits for merge, unless the human asks to stack).
- Branch: `<type>/T<n>-<slug>` (`feat fix refactor perf test docs chore ci`),
  e.g. `feat/T3-login-form`.
- IN/OUT is mandatory. OUT is where "while I'm here" work goes (Backlog or
  a later task), not this branch.
- "Done when" must be a command, a test, or a concrete manual step.
- Every "Done when" includes: `./check` passes.
- Low: copy, styling, logs, tests. Medium: normal features. High: auth,
  payments, permissions, schema/migrations, data deletion, CI/deploy,
  env/config, new deps, plus `AGENTS.md` high-risk. Any high-risk touch →
  the whole task is high. Isolate it. High-risk tasks name rollback and a
  flag or staging step.
- Status after you write them: `approved` with `Approved: YYYY-MM-DD`.
  Until then they stay `draft` only in the proposal, not in PLAN.md.

## 4. Stop for approval

End with: "Approve this plan, or tell me what to change." Do not start
building. After approval, write the tasks into `docs/PLAN.md` (table +
detail, IN/OUT, Approved date) and append new decisions as one-liners in
`docs/DECISIONS.md`.
