# `pr-opener`

PR step of the SDD multi-agent flow (`.claude/agents/pr-opener.md`, model `haiku`, tools: `Bash`). It never merges, never force-pushes and never targets `main`/`master`.

**Input** (from the orchestrator): `id` (the epic or a single task), `tasks` (the sub-issues in this delivery as `#<n> <title>`, or `-`), `type`, `branch`, `base` (`release/*`), `title`, `summary`, `approach` (URL or `-`), `deviations` (URLs or `-`), `manual` (checks for the human, or `-`), `left` (or `-`).

The agent checks that the branch belongs to the task and has commits over the base, pushes it, and opens one PR `<type>(#<id>): <title>` into the base with the body "What was done / How to check / What is left", the list of sub-issues, and `Refs #<id>, #<sub-issue>, ...` (not `Closes`: it does not fire on merges into `release/*`; `feature-finisher` closes the task). Then it sets the board Status of the task and each sub-issue to In review (only forward).

Re-running is safe: an up-to-date push is skipped, and an open PR is reused with its body refreshed.

**Output:**

```
PR_OPENER_RESULT
status: ok
id: 18
tasks: -
branch: feat/18-pr-opener-agent
base: release/0.1.0
sha: 4462666b3f4eacdba1d08a4277ea3aaa4602cd30
push: pushed
pr: 26
pr_url: https://github.com/Beefeater84/sdd-spec-kit/pull/26
pr_state: created
board: in_review
warnings: -
```

`status: partial` means a warning (e.g. the open PR targets another base). On failure: `status: error`, `id`, `reason`, `hint`.
