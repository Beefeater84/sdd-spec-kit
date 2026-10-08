# `validator`

Validation step of the SDD multi-agent flow (`.claude/agents/validator.md`, model `sonnet`, tools: `Read`, `Grep`, `Glob`, `Bash`). It never changes files, commits or uses `gh`: it checks, it does not fix.

**Input:** `id`, `branch`, `base`, `folder` (e.g. `specs/features/42-user-login/`).

The agent runs the project's typecheck, lint and tests for the whole project; every item of `validation.md` it can automate; checks that every `plan.md` group is done in its own commit `<type>(#<task>): ...` and no files outside the plan were added; and checks the diff against accepted ADRs in `specs/decisions/`. A check it cannot run is `skipped`, never a pass. Items that need a human (browser, real account, judgment) go to `manual` and end up in the PR under "How to check".

If it fails, the orchestrator sends the failures to an implementer and runs the validator again, at most two rounds, then stops for the human.

**Output** (a run that caught an extra file and an ADR violation, shortened):

```
VALIDATOR_RESULT
status: fail
id: 42
branch: feat/42-todo-due-dates
checks:
  - typecheck — skipped — not configured
  - lint — pass — npm run lint
  - test — pass — npm test; 9 passed, 0 failed
  - plan: group 1 — pass — src/store.js and test/store.test.js changed as listed
  - plan: extra — fail — src/debug.js
  - adr: specs/decisions/7-lenient-due-dates.md — fail — src/store.js contradicts the decision
failures:
  - plan: extra — src/debug.js is not in any plan group ...
  - adr: 7-lenient-due-dates.md says createTodo never throws on a due date; src/store.js throws INVALID_DUE_DATE ...
manual:
  - The due date reads well in the UI date picker — open the app and pick a date
reason: -
```

`status` is `pass`, `fail`, or `error` (wrong branch, uncommitted changes, missing `plan.md` or `validation.md`).
