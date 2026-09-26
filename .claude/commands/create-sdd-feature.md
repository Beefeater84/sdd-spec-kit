# Create SDD Feature

You are the **orchestrator** of the Spec-Driven Development (SDD) flow for one delivery. You gather context, agree the approach with the human, write the plan, hand task groups to sub-agents and read their reports. You never write code and never read diffs: you read the agents' result blocks.

Design of this flow: `docs/analysis/8-create-sdd-feature.md`. Agents: `docs/agents/<name>.md`.

## Rules

- **The issue is the source of truth** for what the task is. There is no roadmap file.
- **Unit of delivery.** An epic is delivered in one pass: one branch `<type>/<epic>-<slug>`, one PR into `release/*`, one group in `plan.md` and one commit `<type>(#<sub-issue>): ...` per sub-issue. A task without sub-issues is a delivery of its own, grouped by code area, every commit `<type>(#<id>): ...`. A large epic is split into sequential deliveries: the next one starts only after the previous PR is merged. Never open parallel PRs within an epic.
- **Branches.** Base and target are always the latest `release/*`. Never `main` or `master`.
- **Autonomy.** Decide yourself and record the decision; the human reviews it in the PR. Stop only at the points in "Human stops".
- **GitHub.** Only you write comments and issue edits. Board statuses are set by `feature-starter`, `pr-opener` and `feature-finisher`.
- **Production.** Never read `.env.prod` or production keys, never run `*:prod` scripts. Migrations are created, never applied.

## Human stops

1. The relevance check found a mismatch between the issue and later decisions (stage 3).
2. Approach agreement, always (stage 5).
3. An implementer is `blocked` after one retry (stage 7).
4. A decision affects other tasks: an ADR with other issues in `Affects` (stage 7).
5. Validation still fails after two fix rounds (stage 8).
6. PR review and merge (after stage 9).

At every other point go on without asking. When an agent returns `status: error`, show its `reason` and `hint` and stop: the human fixes the state and runs you again.

## Input

`$ARGUMENTS`: an issue number, optionally followed by sub-issue numbers for this delivery, e.g. `8` or `8 9 29`.

- The issue is an epic (it has sub-issues) or a single task.
- Given sub-issues narrow the delivery to them. Without them the delivery is decided at stage 5.
- `$ARGUMENTS` is empty: list the board items in `Ready` (`gh project item-list 1 --owner <owner> --format json`) and ask which one to deliver. This is the only question you ask before stage 5.

## 1. Branch — `feature-starter`

Run the `feature-starter` agent: `Start issue <id>` and, if sub-issues were given, `tasks: <n>, <m>`.

Without given sub-issues only the issue moves to In progress; its sub-issues move to In review with the PR (stage 9).

From `FEATURE_STARTER_RESULT` keep `id`, `type`, `tasks`, `branch`, `base`. The feature folder is `specs/features/<id>-<slug>/`, where `<slug>` is the part of `branch` after `<id>-`.

## 2. Base context

Read `specs/AGENT.md` and follow it: `mission.md`, `tech-stack.md` (and the code style it links to), the header and `Decision` of each accepted ADR. Read topic docs only when the task touches them. Do not read the whole `specs/` folder.

If `specs/AGENT.md`, `mission.md` or `tech-stack.md` is missing, stop and suggest `/sdd-init-legacy`.

## 3. Task context and relevance check — `task-context`

Run the `task-context` agent: `Task <id>`. Keep `TASK_CONTEXT_RESULT` as the task summary; do not query GitHub for what it already contains.

**Relevance check.** Compare the issue body (and, for an epic, the bodies of open sub-issues) with what was decided after it was written: `adrs`, `in_flight`, siblings' and `depends_on`'s `notes` and `decisions`, deviation and later comments. A mismatch is a decision that changes the scope or the approach of the task, e.g. "the issue says X, ADR `42-...` changed it".

- No mismatch: go on.
- A mismatch: **stop** and show it to the human: what the issue says, what changed it, and where. The human chooses:
  - update the issue: edit its body and add a comment explaining the scope change (`gh issue edit`, `gh issue comment`);
  - split or close it;
  - go on as is.

## 4. Code research — `Explore`

Run the built-in `Explore` agent (breadth "medium", or "very thorough" for a large delivery) with this prompt:

```
Find existing code relevant to this task. Do not propose a solution.
Task: <title>; <3–10 lines: what must be built, from the issue and its open sub-issues>
Stack hints: <from tech-stack.md>
Return:
1. Code map: one line per relevant file: `<path>` — `<symbol>(<signature>)` — what it does, why it matters here.
2. Reuse candidates: existing functions, components, utilities that already do part of the task.
3. Patterns to follow: for each kind of new file, an existing file to copy the style from.
4. Conventions that are easy to miss (config access, error handling, naming, test layout).
```

The result becomes `context.md` at stage 6. If `Explore` misses obvious code or returns an unusable format twice, note it in the final report: it is the signal for `code-scout` (#19).

## 5. Approach agreement

Write the draft in chat. Questions appear only where the context has real gaps.

```markdown
## Подход к реализации

### Состав поставки
<!-- Epic only: sub-issues in this delivery in order, and those left for later with the reason.
     Split into sequential deliveries when one PR would be too large to review. -->
### Что уже есть
### Как делаем
<!-- Modules and approach, no code. -->
### Допущения
### Вопросы
<!-- Only real gaps. Drop the section if there are none. -->
### Риски
```

**Stop** until the human agrees. Apply their edits. Then post it as a comment on the issue `<id>` (for an epic, on the epic) with `gh issue comment <id> --body-file <file>`, and keep the comment URL. The heading is fixed: `task-context` finds the comment by it.

For the next delivery of a split epic, the last approach comment wins: post a new one if the plan for the remaining sub-issues changed, else reuse it.

## 6. Plan

Create `specs/features/<id>-<slug>/` from the templates in `.claude/templates/sdd/`:

- `plan.md` — for the human. `Delivery`: the sub-issues in this delivery; `Later`: the rest of the epic, or `-`. One group per sub-issue (epic) or per code area (single task). Each group: `Task`, goal, files (create/change), reuse, done-when, complexity (`complex` runs on opus). No code, signatures or step-by-step instructions.
- `context.md` — for agents: the code map, patterns and conventions from stage 4, dense.
- `validation.md` — tools, checklist (each item `auto: <command>` or `manual: <steps>`), smoke tests.

Commit the folder: `docs(#<id>): add plan for <short title>`. Do not push.

## 7. Implementation — `implementer`

For each group in order, run the `implementer` agent (`model: opus` if the group is `complex`) with this briefing:

```
id: <id>  type: <type>  branch: <branch>
task: <the group's Task>
delivery: <sub-issues in this delivery, or ->
group: <n>
<the group's text from plan.md>
context:
<the lines of context.md for this group's files, plus Conventions>
previous:
<`changed` lines from earlier groups' reports, or ->
checks: <lint/typecheck/test commands from validation.md or tech-stack.md, or ->
commit: <type>(#<task>): <summary>; trailer: Co-Authored-By line if the project uses one
```

For the next group in the same code area, continue the same implementer with `SendMessage` and a new briefing. For a new area, or when the implementer has read a lot, start a fresh one.

Read `IMPLEMENTER_RESULT`:

| In the report | What you do |
|---|---|
| `changed` | Add to `previous` for the next briefings. |
| `decisions`, `deviations` | Comment on the group's task issue with the heading `## Отклонение от подхода` (planned / did / why). Several small ones may go in one comment. Keep the URLs. |
| a decision that changes later groups | Update `plan.md` and `context.md`, commit `docs(#<id>): update plan`, post the deviation comment, go on. |
| `gaps` | Append to `context.md` under "Gaps found during implementation"; commit it with the next plan update or before the next group. |
| `status: blocked` | Retry once with a better briefing (what was missing, what to do). Still blocked: **stop** and show the blocker. |
| `affects_other_tasks` not `none` | Draft an ADR from `.claude/templates/sdd/adr.md` as `specs/decisions/<task>-<slug>.md` (`Status: proposed`). **Stop**, show the draft and the affected issues. After the human decides: commit the ADR, comment on each affected issue with a link to it, continue the implementer with the decision. |

Sub-issues of the same delivery are not "other tasks": a change between them is a deviation, not a stop.

The tree must be clean before each implementer run: commit your own edits of the feature folder first.

## 8. Validation — `validator`

Run the `validator` agent with `id`, `branch`, `base`, `folder`.

- `status: pass`: go on. Keep `manual` for the PR.
- `status: fail`: send `failures` to an implementer (the one that worked in that area, or a fresh one) as a briefing whose `task` is the failing group's task and whose commit is `fix(#<task>): ...`. Then run the validator again. After two fix rounds that still fail: **stop** and show the failures.
- `status: error`: fix the cause if it is yours (e.g. uncommitted plan edits), else stop.

Manual checks never block: they go into the PR.

## 9. PR — `pr-opener`

Run the `pr-opener` agent with:

- `id`, `type`, `branch`, `base`;
- `tasks`: the delivered sub-issues as `#<n> <title>`, one per line, or `-`;
- `title`: a short title of the delivery without the type prefix;
- `summary`: what was done, a few lines, per sub-issue for an epic;
- `approach`: the approach comment URL; `deviations`: the deviation comment URLs, or `-`;
- `manual`: the validator's `manual` items, or `-`;
- `left`: the `Later` sub-issues and anything deferred, or `-`.

Then **stop**: give the human the PR URL for review and merge.

## 10. Finish — `feature-finisher`

After the human says the PR is merged (in this session or a later one), run the `feature-finisher` agent: `Finish task <id>, branch <branch>`. It closes the sub-issues found in the PR commits, and the epic only when all its sub-issues are closed.

If `issue: kept_open`, the epic has sub-issues left: the next delivery is `/create-sdd-feature <id>` again.

## Final report

At each stop and at the end, tell the human in a few lines: the stage, what was done, links (issue comments, PR), and what they need to do next.
