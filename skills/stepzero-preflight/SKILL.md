---
name: stepzero-preflight
description: Verifies a finished task before opening a PR. Runs ./check, confirms Done when and IN/OUT, gathers proof for medium/high risk, and flags high-risk or gate-file scope creep. Use when the user runs /stepzero-preflight, says a task is done, or before creating a pull request.
disable-model-invocation: true
---

# Preflight

Decide whether the current task is ready for a PR. Evidence over claims.

## Steps

1. **Identify the task.** Find it in `docs/PLAN.md`: Branch, IN/OUT, Done
   when, Risk, Depends on, Approved date. If missing, or status isn't
   `doing`/`approved`/`changes requested`, stop. Confirm
   `git branch --show-current` equals Branch and the diff stays inside IN;
   if not, NOT READY.

2. **Run `./check`.** Paste the output tail. If it fails: NOT READY. You may
   fix only in-scope files and re-run; don't start extra work. Never report
   a pass you did not observe.

3. **Verify "Done when" against the diff.** Evidence per item. Each IN
   behavior: a test that would fail without it, or an explicit why-not
   (human must accept). Grep if you're unsure the behavior already lived
   elsewhere — don't walk unrelated packages. Extra functions/files this
   task doesn't need → NOT READY (speculative). Don't require a unit test
   per private helper.

4. **Gather proof for medium/high risk.** Provide what a human needs to trust
   it: how to try it locally, inputs and expected outputs, screenshots or logs
   where relevant. For high risk also confirm the rollback plan and the
   feature flag or staging step exist.

5. **Check scope.** Run `git diff --stat` against the base branch and compare
   touched files to the task. Then run `.stepzero/risk` (if present); it lists
   changed files matching the project's high-risk patterns. Flag:
   - Files or features in OUT or unrelated to IN.
   - Any `.stepzero/risk` match, or other high-risk change (see `AGENTS.md`),
     when the task is not marked high. **Stop and tell the human.** Do not
     downgrade or proceed.
   - Any change to the gates (`check`, `.github/`, `.stepzero/`, `AGENTS.md`,
     `.cursorignore`) or any removed/skipped test, unless the task is
     explicitly about it. This is always NOT READY.
   - New dependencies, env vars, config, CI, or migrations not in the plan.

## Output

```markdown
## Preflight: T<n> <title>
- Branch: <name> (matches plan | MISMATCH)
- IN/OUT: respected | VIOLATION
- Check: PASS | FAIL (output tail below)
- Done when:
  - [x] <item>: <evidence / test name>
- Requirements: IN met, OUT untouched | GAP
- Speculative extra code: none | <files>
- Risk: <tier>; proof: <summary or "n/a (low)">
- Scope: clean | <issues>
- High-risk paths (.stepzero/risk): none | <files>
- Verdict: READY FOR PR | NOT READY (<reason>)

<check output tail>
```

If READY **and** `./check` passed in this turn: write the PR body here
(same template as `/stepzero-report` PR mode), set status `in review`, then
push and open the PR. Don't start a second gather pass over the whole tree.
If NOT READY, stop — no PR body. Never merge. Medium/high: leave the Human
try boxes for the human.
