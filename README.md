# Dev Processes

Development standards, templates, and tools for project setup and feature delivery.

## AGENTS

### `feature-starter`

First step of the SDD multi-agent flow (`.claude/agents/feature-starter.md`, model `haiku`, tools: `Bash`).

**Input:** a GitHub issue number, or a task description (the agent then creates the issue with `gh issue create`).

The agent fetches `origin`, picks the highest `origin/release/*` by version (`1.10` > `1.9`), checks it matches `origin`, and creates a local branch `<type>/<id>-<slug>` from it without upstream. Then it sets the task's board Status to In progress (only from empty, Backlog or Ready; it never moves a task back). It never pushes and never uses `main`/`master`. It stops if there is no `origin/release/*`, if the branch already exists, or if the working tree is dirty.

Local `release/*` branches are never used as a base, but they are checked for unpushed work. If a local release is newer than every release in `origin`, or the local base branch is ahead of (or diverged from) `origin`, the agent stops and returns a `hint` with the command for a human to run (e.g. `git push -u origin release/1.11`). After that, run the agent again.

**Output** (fixed format for the next agents):

```
FEATURE_STARTER_RESULT
status: ok
id: 1
type: feat
branch: feat/1-first-agent
base: release/1.10
base_sha: acf3fac1b750c189c3426e56a5d57e6753269b53
issue_url: https://github.com/Beefeater84/sdd-spec-kit/issues/1
board: in_progress
```

`board` is `in_progress`, `already_in_progress`, `kept` (the task is further along, e.g. In review), `not_on_board`, or `error`. A board failure never undoes the branch.

On failure: `status: error`, `id`, `reason`, `hint` (`-` if there is nothing to suggest).

---

### `task-context`

Context step of the SDD multi-agent flow (`.claude/agents/task-context.md`, model `haiku`, tools: `Bash`, `Read`, `Grep`). Read-only.

**Input:** a task id, e.g. `16`.

The agent collects a compact summary so the orchestrator does not read raw GitHub data: the issue body and the `## Подход к реализации` comment verbatim; `## Отклонение от подхода` and other comments as one line each; the parent epic and its goal; sibling sub-issues with their PRs and approach/deviation notes; ADRs from `specs/decisions/` that affect the task or are newer than it; ADRs proposed in open PRs of sibling tasks. It never judges relevance, edits issues, or reads code.

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
siblings:
  - #10 CLOSED Document create-sdd-feature design and create sub-agent issues — pr: #22 — notes: -
  - #20 OPEN feature-starter: set board status to In progress — pr: #23 open — notes: -
adrs: -
in_flight: -
```

On failure: `status: error`, `id`, `reason`.

---

### `implementer`

Implementation step of the SDD multi-agent flow (`.claude/agents/implementer.md`, model `sonnet`; the orchestrator runs groups marked complex on `opus`; tools: `Read`, `Edit`, `Write`, `Grep`, `Glob`, `Bash`). It never pushes and never uses `gh`.

**Input:** a briefing from the orchestrator: `id`, `type`, `branch`, one `group` from `plan.md`, the matching slice of `context.md`, `previous` (what earlier groups created), `checks` (lint/test commands), `commit` format.

The agent reads the listed files (a broad search is reported as a gap in the code map), implements the group, decides the details itself and records its choices, runs lint and tests, and commits the group by explicit paths. If the group needs a change that affects other tasks, it does not make it: it stops with `status: blocked` and `affects_other_tasks`, and the orchestrator asks the human.

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

---

### `validator`

Validation step of the SDD multi-agent flow (`.claude/agents/validator.md`, model `sonnet`, tools: `Read`, `Grep`, `Glob`, `Bash`). It never changes files, commits or uses `gh`: it checks, it does not fix.

**Input:** `id`, `branch`, `base`, `folder` (e.g. `specs/features/42-user-login/`).

The agent runs the project's typecheck, lint and tests for the whole project; every item of `validation.md` it can automate; checks that every `plan.md` group is done and no files outside the plan were added; and checks the diff against accepted ADRs in `specs/decisions/`. A check it cannot run is `skipped`, never a pass. Items that need a human (browser, real account, judgment) go to `manual` and end up in the PR under "How to check".

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

---

### `pr-opener`

PR step of the SDD multi-agent flow (`.claude/agents/pr-opener.md`, model `haiku`, tools: `Bash`). It never merges, never force-pushes and never targets `main`/`master`.

**Input** (from the orchestrator): `id`, `type`, `branch`, `base` (`release/*`), `title`, `summary`, `approach` (URL or `-`), `deviations` (URLs or `-`), `manual` (checks for the human, or `-`), `left` (or `-`).

The agent checks that the branch belongs to the task and has commits over the base, pushes it, and opens a PR `<type>(#<id>): <title>` into the base with the body "What was done / How to check / What is left" and `Refs #<id>` (not `Closes`: it does not fire on merges into `release/*`; `feature-finisher` closes the task). Then it sets the board Status to In review (only forward).

Re-running is safe: an up-to-date push is skipped, and an open PR is reused with its body refreshed.

**Output:**

```
PR_OPENER_RESULT
status: ok
id: 18
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

---

### `feature-finisher`

Last step of the SDD multi-agent flow (`.claude/agents/feature-finisher.md`, model `haiku`, tools: `Bash`). Runs after the feature PR is merged. It does not run tests.

**Input:** task id and task branch, e.g. `Finish task 1, branch feat/1-feature-starter-agent`.

The agent checks that the branch belongs to the task and its PR is merged; otherwise it changes nothing. Then it:
1. closes the issue with a comment `Done in #<pr>` (needed because `Closes #N` does not fire on merges into `release/*`); for a sub-issue it reports the epic's progress but never closes the epic;
2. sets the board Status to Done;
3. fast-forwards the local latest `release/*` from `origin` (switches to it only if you are on the task branch);
4. deletes the task branch locally and in `origin`, but only if its tip is exactly what was merged.

Re-running is safe: completed steps are skipped.

**Output:**

```
FEATURE_FINISHER_RESULT
status: ok
id: 1
pr: 3
branch: feat/1-feature-starter-agent
base: release/0.1.0
issue: already_closed
epic: -
board: already_done
release: release/0.1.0
release_state: up_to_date
local_branch: absent
remote_branch: deleted
warnings: -
```

`status: partial` means something was kept on purpose (see `warnings`). On failure: `status: error`, `id`, `reason`, `hint`.

---

### `releaser`

Release step of the SDD multi-agent flow (`.claude/agents/releaser.md`, model `haiku`, tools: `Bash`). Run it manually when the release is ready. It never merges and does not run tests.

**Input:** optional version and optional next version, e.g. `Release 0.1.0` or `Release, next patch`. Without a version the highest `origin/release/*` is used. The next version is `major`, `minor` (default) or `patch` bump, or an explicit `X.Y.Z`.

The agent works in two runs; it finds out which one from the state of the release PR:
1. **Before the merge:** it stops if the tag already exists, if PRs into `release/X.Y.Z` are still open, or if there is nothing new over `main`. Otherwise it builds release notes from the PRs merged into the release branch, grouped by type (`feat`, `fix`, `docs`, `refactor`, `chore`, other), and opens the PR `release/X.Y.Z` → `main` with them (or refreshes the body of the open PR). Ends with `status: awaiting_merge`. The human merges the PR.
2. **After the merge:** it creates the tag `vX.Y.Z` on the merge commit and a GitHub Release with the PR body as notes, fast-forwards the local `main`, and creates and pushes `release/<next>` from `origin/main`.

Re-running is safe: an existing PR, tag, GitHub Release or next release branch is skipped. It never moves or overwrites a tag.

**Output:**

```
RELEASER_RESULT
status: ok
version: 0.1.0
release: release/0.1.0
pr: 12
pr_url: https://github.com/Beefeater84/sdd-spec-kit/pull/12
pr_state: merged
tag: v0.1.0
tag_state: created
gh_release: created
release_url: https://github.com/Beefeater84/sdd-spec-kit/releases/tag/v0.1.0
main_state: updated
next_release: release/0.2.0
next_branch_state: created
warnings: -
hint: -
```

After run 1 `status: awaiting_merge` and `hint` says which PR to merge. `status: partial` means something was kept on purpose (see `warnings`). On failure: `status: error`, `version`, `reason`, `hint`.

## SKILLS

### `/sdd-init-legacy`

Bootstraps an SDD constitution for an existing (legacy) project. **Run once per project.**

**Usage:** `/sdd-init-legacy`

The skill explores the codebase autonomously — README, TODO, package files, git log, migrations — and reverse-engineers the three constitution files (`specs/mission.md`, `specs/tech-stack.md`, `specs/roadmap.md`). It only asks the user for context it cannot discover itself (audience, hidden constraints, team direction). After review and commit, the project is on an SDD foundation and ready for the standard feature workflow.

---

### `/create-sdd-feature`

Creates a feature spec file following the Spec-Driven Development (SDD) workflow.

**Usage:** `/create-sdd-feature` or `/create-sdd-feature "feature name"`

The skill reads the project constitution (`specs/mission.md`, `specs/tech-stack.md`, `specs/roadmap.md`), identifies the target feature, and generates a structured spec file at `specs/features/<feature-name>.md`.

The spec includes: Goal, Requirements, Task Groups, Key Decisions, and a Validation Scorecard.

After spec approval, the skill outputs a ready-to-use implementation prompt and guides the developer through the validation phase — keeping spec and code in sync before merging.

**Requires:** a filled-in project constitution in `specs/`.

---

### `/sdd-validate`

Runs the validation phase after a feature is implemented.

**Usage:** `/sdd-validate` or `/sdd-validate "feature-name"`

Loads the feature spec, walks through the Validation Scorecard (automated + manual checks), reviews the commit diff, and ensures spec and code stay in sync. Marks the feature as `done` and updates the roadmap when all checks pass.

**Requires:** a feature spec with a Validation Scorecard in `specs/features/`.

---

### `/sdd-replan`

Guides a replanning session between features.

**Usage:** `/sdd-replan`

Reviews product direction changes, propagates constitution updates to affected specs and code, and reassesses the roadmap. Small changes are applied immediately; large ones are scheduled as new roadmap features. Encourages working on a dedicated `replan/<topic>` branch to track which version of the constitution produced which code.

## TEMPLATES

Artifacts of the multi-agent `create-sdd-feature` flow (design: `docs/analysis/8-create-sdd-feature.md`).

- `specs/AGENT.md` — a project file, filled in like `mission.md`. Tells agents what to read in `specs/` always (mission, tech stack, accepted decisions) and what by topic, and where decisions go.
- `.claude/templates/sdd/adr.md` — a cross-cutting decision, saved as `specs/decisions/<id>-<slug>.md`. Accepted ADRs are never edited; a new one supersedes the old.
- `.claude/templates/sdd/plan.md` — the plan of one task, for the human: task groups with goal, files, reuse and done-when. No code.
- `.claude/templates/sdd/context.md` — the code map of one task, for agents: paths, symbols, patterns.
- `.claude/templates/sdd/validation.md` — the validator's checklist: automated and manual checks.

A task's files live in `specs/features/<id>-<slug>/`.
