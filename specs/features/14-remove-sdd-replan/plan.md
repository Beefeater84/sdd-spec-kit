# Plan: #14 Remove sdd-replan command

Issue: #14 · Approach: https://github.com/Beefeater84/sdd-spec-kit/issues/14#issuecomment-5846861465
Delivery: -
Later: -

## Group 1: remove the command and its mentions

- Task: #14
- Goal: `/sdd-replan` no longer exists and nothing in the kit points to it.
- Files: `.claude/commands/sdd-replan.md` (delete), `README.md` (change), `docs/analysis/8-create-sdd-feature.md` (change), `.claude/commands/sdd-init-legacy.md` (change)
- Reuse: -
- Done when: the command file is gone; the `/sdd-replan` section and its `---` separator are gone from README; in the analysis doc ADR staleness is caught by `task-context` and PR review, and #14 in "Outside the epic" says the command was removed; the closing note of `sdd-init-legacy.md` no longer mentions `/sdd-replan` or replanning; `grep -rn sdd-replan` finds nothing outside `specs/features/`.
- Complexity: normal
