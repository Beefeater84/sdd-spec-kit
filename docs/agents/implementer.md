# `implementer`

Implementation step of the SDD multi-agent flow (`.claude/agents/implementer.md`, model `sonnet`; the orchestrator runs groups marked complex on `opus`; tools: `Read`, `Edit`, `Write`, `Grep`, `Glob`, `Bash`). It never pushes and never uses `gh`.

**Input:** a briefing from the orchestrator: `id`, `type`, `branch`, `task` (the sub-issue of this group, or `id` for a single task), `delivery` (all sub-issues on the branch), one `group` from `plan.md`, the matching slice of `context.md`, `previous` (what earlier groups created), `checks` (lint/test commands), `commit` format.

The agent reads the listed files (a broad search is reported as a gap in the code map), implements the group, decides the details itself and records its choices, runs lint and tests, and commits the group by explicit paths as `<type>(#<task>): ...`: one commit per sub-issue. If the group needs a change that affects other tasks (sub-issues of the same delivery do not count), it does not make it: it stops with `status: blocked` and `affects_other_tasks`, and the orchestrator asks the human.

For the next group in the same code area the orchestrator continues the same agent (`SendMessage`), so it does not read the files again.

**Output:**

```
IMPLEMENTER_RESULT
status: done
group: 1
commit: 0edcfd9
changed:
  - src/store.js — createTodo — createTodo(title, dueDate?): Todo — changed
  - test/store.test.js — - — - — changed
decisions:
  - Missing dueDate normalizes to null, so the field is always present.
deviations: -
gaps: -
affects_other_tasks: none
checks: npm run lint — pass; npm test — 5/5 pass
blocker: -
```

The orchestrator uses it: `changed` → the next briefing; `decisions`, `deviations` → a deviation comment in the issue; `gaps` → `context.md`; `affects_other_tasks` → an ADR and a stop.
