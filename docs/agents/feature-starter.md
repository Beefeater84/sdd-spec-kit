# `feature-starter`

First step of the SDD multi-agent flow (`.claude/agents/feature-starter.md`, model `haiku`, tools: `Bash`).

**Input:** a GitHub issue number (an epic or a single task), optionally with `tasks`: the epic's sub-issues in this delivery (e.g. `Start issue 8, tasks 9, 29`). Or a task description: the agent then creates the issue with `gh issue create`.

The unit of delivery is an epic: one branch named after the epic, one PR, one commit per sub-issue. Each task in `tasks` must be an open sub-issue of the issue.

The agent fetches `origin`, picks the highest `origin/release/*` by version (`1.10` > `1.9`), checks it matches `origin`, and creates a local branch `<type>/<id>-<slug>` from it without upstream. Then it sets the board Status of the issue and of each task in `tasks` to In progress (only from empty, Backlog or Ready; it never moves a task back). It never pushes and never uses `main`/`master`. It stops if there is no `origin/release/*`, if the branch already exists, or if the working tree is dirty.

Local `release/*` branches are never used as a base, but they are checked for unpushed work. If a local release is newer than every release in `origin`, or the local base branch is ahead of (or diverged from) `origin`, the agent stops and returns a `hint` with the command for a human to run (e.g. `git push -u origin release/1.11`). After that, run the agent again.

**Output** (fixed format for the next agents):

```
FEATURE_STARTER_RESULT
status: ok
id: 8
type: feat
tasks: #9, #29
branch: feat/8-create-orchestrator
base: release/1.10
base_sha: acf3fac1b750c189c3426e56a5d57e6753269b53
issue_url: https://github.com/Beefeater84/sdd-spec-kit/issues/8
board: in_progress
```

`tasks` is `-` when none were given. `board` combines all issues: `in_progress`, `already_in_progress`, `kept` (further along, e.g. In review), `not_on_board`, or `error`. A board failure never undoes the branch.

On failure: `status: error`, `id`, `reason`, `hint` (`-` if there is nothing to suggest).
