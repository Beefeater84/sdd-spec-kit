# Dev Processes

Development standards, templates, and tools for project setup and feature delivery.

## AGENTS

### `feature-starter`

First step of the SDD multi-agent flow (`.claude/agents/feature-starter.md`, model `haiku`, tools: `Bash`).

**Input:** a GitHub issue number, or a task description (the agent then creates the issue with `gh issue create`).

The agent fetches `origin`, picks the highest `origin/release/*` by version (`1.10` > `1.9`), checks it matches `origin`, and creates a local branch `<type>/<id>-<slug>` from it without upstream. It never pushes and never uses `main`/`master`. It stops if there is no `origin/release/*`, if the branch already exists, or if the working tree is dirty.

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
```

On failure: `status: error`, `id`, `reason`, `hint` (`-` if there is nothing to suggest).

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
