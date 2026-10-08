# Validation: #34 task-context: follow Depends on links

Fixture: issue #36 "[test] task-context depends_on fixture for #34" — body is the original body of #13 ("Depends on #8"), native blocked by #13. Closed by the orchestrator after the PR is opened.

## Tools

- bash, grep, `gh` (read-only), the `Agent` tool is not available to the validator — the end-to-end run is done by the orchestrator (see Smoke tests).

## Checklist

- [ ] Step 2 requests `blockedBy` — auto: `grep -n 'blockedBy' .claude/agents/task-context.md`
- [ ] The result format has `depends_on` between `siblings` and `adrs` — auto: `awk '/^TASK_CONTEXT_RESULT/,/^```$/' .claude/agents/task-context.md | grep -nE '^(siblings|depends_on|adrs):'` prints them in that order
- [ ] `description` mentions dependencies — auto: `sed -n 3p .claude/agents/task-context.md | grep -iE 'depend'`
- [ ] Parsing command gives the expected numbers — auto: take the body-parsing command from the Dependencies step of `.claude/agents/task-context.md` verbatim and run it on each input below (feed via a file or stdin as the command expects); compare the printed numbers:
  - `Depends on #8` → `8`
  - `Зависит от #8, #9` → `8 9`
  - `Blocked by: #12` → `12`
  - `requires https://github.com/Beefeater84/sdd-spec-kit/issues/21` → `21`
  - `DEPENDS ON #5 and #6` → `5 6`
  - `после #5`, `см. #7`, `Done in #33` → nothing
  - the body of #36 (`gh issue view 36 --json body -q .body`) → `8`
- [ ] Dependency numbers are in the `Affects` rule and in `in_flight` — auto: `grep -nE 'Affects|in_flight' .claude/agents/task-context.md` shows dependencies in both
- [ ] Stage 3 of the orchestrator uses `depends_on` — auto: `grep -n 'depends_on' .claude/commands/create-sdd-feature.md`
- [ ] Docs mention the new key — auto: `grep -ln 'depends_on' docs/agents/task-context.md docs/analysis/8-create-sdd-feature.md` lists both; `grep -n 'task-context' README.md` row mentions dependencies
- [ ] The problem is solved in substance — manual: read `depends_on` for #8 from the smoke run (in the PR); it must show the decisions that changed the scope of #13: validation moved to the `validator` agent (stage 8), `Key Decisions` removed from `plan.md`, push and PR done by `pr-opener`
- [ ] Real agent run — manual: in a new Claude Code session run the `task-context` agent with `Task 36`; `depends_on` matches the smoke run below

## Smoke tests

Run by the orchestrator with an agent on model haiku whose instructions are the text of the new `.claude/agents/task-context.md`:

- `Task 36` → `depends_on` has `#8` (`source: body`, OPEN, `pr: #30`, `decisions` covers "Итоги обсуждения" parts 1–6 and the correction about decision 33) and `#13` (`source: blocked by`, CLOSED, `pr: #33`); no more than ~10 lines per dependency, no verbatim comment bodies.
- `Task 34` → `depends_on: -`.
