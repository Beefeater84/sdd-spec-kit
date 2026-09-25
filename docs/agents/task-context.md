# `task-context`

Context step of the SDD multi-agent flow (`.claude/agents/task-context.md`, model `haiku`, tools: `Bash`, `Read`, `Grep`). Read-only.

**Input:** a task id, e.g. `16`: an epic or a single task.

The agent collects a compact summary so the orchestrator does not read raw GitHub data: the issue body and the `## Подход к реализации` comment verbatim; `## Отклонение от подхода` and other comments as one line each; the task's own sub-issues (for an epic, the content of the delivery: open ones with body and comments, closed ones with their PR); the parent epic and its goal; sibling sub-issues with their PRs and approach/deviation notes; ADRs from `specs/decisions/` that affect the task or are newer than it; ADRs proposed in open PRs of sibling tasks. It never judges relevance, edits issues, or reads code.

**Output** (shortened):

```
TASK_CONTEXT_RESULT
status: ok
id: 16
title: Create implementer agent
url: https://github.com/Beefeater84/sdd-spec-kit/issues/16
state: OPEN
created: 2026-09-25
labels: -
epic: #8 Analyze create-sdd-feature command and align it with agents
epic_goal: |
  ...
body: |
  ...
approach: -
approach_body: -
deviations: -
comments: -
sub_issues: -
siblings:
  - #10 CLOSED Document create-sdd-feature design and create sub-agent issues — pr: #22 — notes: -
  - #20 OPEN feature-starter: set board status to In progress — pr: #23 open — notes: -
adrs: -
in_flight: -
```

On failure: `status: error`, `id`, `reason`.
