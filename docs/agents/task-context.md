# `task-context`

Context step of the SDD multi-agent flow (`.claude/agents/task-context.md`, model `haiku`, tools: `Bash`, `Read`, `Grep`). Read-only.

**Input:** a task id, e.g. `16`: an epic or a single task.

The agent collects a compact summary so the orchestrator does not read raw GitHub data: the issue body and the `## Подход к реализации` comment verbatim; `## Отклонение от подхода` and other comments as one line each; the task's own sub-issues (for an epic, the content of the delivery: open ones with body and comments, closed ones with their PR); the parent epic and its goal; sibling sub-issues with their PRs and approach/deviation notes; the tasks it depends on (native blocked by and `Depends on` links in the body) with their PRs and decisions; ADRs from `specs/decisions/` that affect the task or are newer than it; ADRs proposed in open PRs of sibling and dependency tasks. It never judges relevance, edits issues, or reads code.

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
depends_on:
  - #42 OPEN Redesign the plan template — source: blocked by, body — pr: #45 open
    sub_issues: 2 closed, 1 open
    notes: -
    decisions:
      - Итоги обсуждения — plan template keeps one file per group, no per-task split
adrs: -
in_flight: -
```

On failure: `status: error`, `id`, `reason`.
