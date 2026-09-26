# Plan: #34 task-context: follow Depends on links

Issue: #34 · Approach: https://github.com/Beefeater84/sdd-spec-kit/issues/34#issuecomment-5846711489
Delivery: -
Later: -

## Group 1: task-context agent

- Task: #34
- Goal: `task-context` finds the task's dependencies (native blocked by and text links in the body) and returns a compact `depends_on` summary for each.
- Files: `.claude/agents/task-context.md` (change)
- Reuse: `.claude/agents/task-context.md` — Step 2 comment classification, Step 4 PR matching and `pr` rule, `notes` summary, `in_flight`; Step 5 `Affects` matching.
- Done when: the agent file has `blockedBy` in Step 2, a Dependencies step with one ready-made parsing command, dependency numbers in `Affects` and `in_flight`, and `depends_on` in the result format between `siblings` and `adrs`; `description` mentions dependencies.
- Complexity: complex

## Group 2: orchestrator and docs

- Task: #34
- Goal: the relevance check in `create-sdd-feature` uses `depends_on`, and all docs describe the new key.
- Files: `.claude/commands/create-sdd-feature.md` (change), `docs/agents/task-context.md` (change), `docs/analysis/8-create-sdd-feature.md` (change), `README.md` (change)
- Reuse: `docs/agents/task-context.md` — existing example block.
- Done when: stage 3 lists `depends_on`; the doc example shows `depends_on`; design doc `task-context` spec and step 3a mention dependencies; README agent row mentions dependencies.
- Complexity: normal
