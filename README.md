# Dev Processes

Development standards, templates, and tools for project setup and feature delivery.

## INSTALL

The kit is installed into another repository by an agent. Open Claude Code in that repository and say:

> Install the SDD kit from https://github.com/Beefeater84/sdd-spec-kit — follow the INSTALL section of its README.

Add a ref (`v0.2.0`, `release/0.2.0`) to install a specific version. Say the same again later to update the kit.

### Instructions for the agent

You are in the target repository. Do the steps in order. Stop and ask the human only where a step says so. Never push to `main`/`master`, never force-push.

**1. Check the environment.**

- The current directory is a git repository with a GitHub `origin`: `gh repo view --json nameWithOwner,defaultBranchRef`.
- `gh auth status` lists the `project` scope. If not, ask the human to run `gh auth refresh -s project` and wait.
- The working tree is clean. If not, stop and ask.

**2. Pick the version.**

- The human named a ref: use it.
- Otherwise the latest tag: `git ls-remote --tags --refs --sort=-v:refname https://github.com/Beefeater84/sdd-spec-kit 'v*' | head -1`.
- No tags: `main`.

**3. Download the kit** into a temporary folder outside the repository:

```bash
KIT=$(mktemp -d)/sdd-spec-kit
git clone --quiet --filter=blob:none --branch <ref> https://github.com/Beefeater84/sdd-spec-kit "$KIT"
KIT_SHA=$(git -C "$KIT" rev-parse HEAD)
```

**4. Create the branch.**

- Find the latest release branch: `git fetch origin && git branch -r --list 'origin/release/*' --sort=-v:refname | head -1`.
- None: stop. Propose to the human to create `release/0.1.0` from the default branch and push it; do it only after a yes.
- Create `chore/install-sdd-kit` from it (with a task: `chore/<issue>-install-sdd-kit`).

**5. Copy the kit files.** Only these three folders, file by file:

| From the kit | To the target |
|---|---|
| `.claude/agents/*.md` | `.claude/agents/` |
| `.claude/commands/*.md` | `.claude/commands/` |
| `.claude/templates/sdd/*` | `.claude/templates/sdd/` |

```bash
mkdir -p .claude/agents .claude/commands .claude/templates/sdd
cp "$KIT"/.claude/agents/*.md .claude/agents/
cp "$KIT"/.claude/commands/*.md .claude/commands/
cp "$KIT"/.claude/templates/sdd/* .claude/templates/sdd/
```

- Kit files overwrite files with the same name. Other files in these folders stay. Before copying, list target files that will be overwritten and are not from a previous install (no `.claude/sdd-kit-version`); name them in the report.
- Never copy anything else from the kit: not `.claude/settings*.json`, `docs/`, `specs/`, `README.md`, `CLAUDE.md`, `agent.md`. Those describe the kit itself.
- Update (`.claude/sdd-kit-version` exists): delete files the kit removed since the installed commit:

  ```bash
  OLD_SHA=$(sed -n 's/^commit: //p' .claude/sdd-kit-version)
  git -C "$KIT" diff --name-only --diff-filter=D "$OLD_SHA" "$KIT_SHA" -- .claude/agents .claude/commands .claude/templates/sdd
  ```

  Delete the listed paths in the target. If `OLD_SHA` is unknown to the kit, skip this and say so in the report.
- Write `.claude/sdd-kit-version`:

  ```
  source: https://github.com/Beefeater84/sdd-spec-kit
  ref: <ref>
  commit: <KIT_SHA>
  ```

**6. Check the task board.** The agents find the board through the project items of an issue, so issues must be on a GitHub Project (v2).

- `gh project list --owner <owner>`. One project: use it. Several or none: ask the human which one (or to create one).
- `gh project field-list <number> --owner <owner>` has a single-select `Status` with `Backlog`, `Ready`, `In progress`, `In review`, `Done`. Missing options: report them, do not edit the board.

**7. Add the project rules** to the target's `CLAUDE.md` (create it if there is none; if it only imports another file with `@file`, edit that file). Add each section only if it is not there yet; fill in the owner, repository and project number:

```markdown
## Task tracking

All tasks (epics, features, bugs) live in GitHub, not in files in this repo.

- **Repository:** <owner>/<repo>
- **Project board:** <owner>'s project number <number>
- **Tasks:** GitHub Issues of this repository. The issue number is the task id and is used in every artifact (branch, plan, commits, PR).
- **Epics:** issues with sub-issues. A feature is a sub-issue of its epic.

Board fields: Status (Backlog, Ready, In progress, In review, Done), Priority (P0, P1, P2), Size (XS, S, M, L, XL).

Use the `gh` CLI to read and update tasks, for example `gh issue view <n>` and `gh project item-list <number> --owner <owner>`.

## Branching

- Feature branches are created from the latest `release/*` branch in `origin` and merged back into it.
- Never branch from or target `main` or `master`. They hold released code only.
- Branch name: `<type>/<issue-number>-<short-description>`, where type is one of `feat`, `fix`, `docs`, `refactor`, `chore`.
```

**8. Commit and open the PR.**

- One commit: `chore: install sdd-spec-kit <ref> (<short KIT_SHA>)` (`update` instead of `install` on an update).
- Push the branch and open a PR into the release branch from step 4 with `gh pr create --base release/<x.y.z>`.
- Remove the temporary folder.

**9. Report to the human:**

- the installed ref and commit, the PR link;
- files overwritten or deleted, missing board options, anything skipped;
- next steps:
  1. Review and merge the PR, then restart Claude Code so the new agents and commands load.
  2. If `specs/AGENT.md`, `specs/mission.md` or `specs/tech-stack.md` is missing, run `/sdd-init-legacy` once. It asks questions, so the human runs it, not you.
  3. Deliver tasks with `/create-sdd-feature <issue#>`.

## AGENTS

Sub-agents of the SDD multi-agent flow, in `.claude/agents/`. Each one has a page in `docs/agents/` with its input, behavior and output format.

The unit of delivery is an epic (or a single task without sub-issues): one branch `<type>/<epic>-<slug>`, one PR into `release/*`, one commit `<type>(#<sub-issue>): ...` per sub-issue. Sub-issues stay on the board for tracking and deviation comments. A large epic goes as several sequential deliveries.

| Agent | Step | Model |
|---|---|---|
| [`feature-starter`](docs/agents/feature-starter.md) | Branch `<type>/<id>-<slug>` from the latest `release/*`; In progress for the issue and its sub-issues in the delivery. | haiku |
| [`task-context`](docs/agents/task-context.md) | Compact summary of the task: issue, sub-issues, epic, siblings, dependencies, ADRs, open PRs. Read-only. | haiku |
| [`implementer`](docs/agents/implementer.md) | Implements one `plan.md` group and commits it as `<type>(#<task>): ...`. | sonnet / opus |
| [`validator`](docs/agents/validator.md) | Independent check: typecheck, lint, tests, `validation.md`, plan coverage, ADRs. Never fixes. | sonnet |
| [`pr-opener`](docs/agents/pr-opener.md) | Push, one PR into `release/*` listing the sub-issues; In review. | haiku |
| [`feature-finisher`](docs/agents/feature-finisher.md) | After merge: closes delivered sub-issues, the epic once all are closed; Done; branch cleanup. | haiku |
| [`releaser`](docs/agents/releaser.md) | Manual: runs the project checks (`## Checks` in `specs/tech-stack.md`), release PR `release/X.Y.Z` → `main`, then tag, GitHub Release, next `release/*`. A failed check stops it; the calling session fixes it with you, then re-runs. Never fixes. | haiku |

## SKILLS

### `/sdd-init-legacy`

Bootstraps an SDD constitution for an existing (legacy) project. **Run once per project.**

**Usage:** `/sdd-init-legacy`

The skill explores the codebase autonomously — README, TODO, package files, git log, migrations — and reverse-engineers the constitution: `specs/AGENT.md`, `specs/mission.md`, `specs/tech-stack.md` and an empty `specs/decisions/`. It only asks the user for context it cannot discover itself (audience, hidden constraints, team direction). After review and commit, the project is on an SDD foundation and ready for the standard feature workflow.

---

### `/create-sdd-feature`

Delivers a GitHub issue (an epic or a single task) through the multi-agent SDD flow. The main agent is the orchestrator: it gathers context, agrees the approach with you, writes the plan and runs the agents; it never writes code itself.

**Usage:** `/create-sdd-feature <issue#>` or `/create-sdd-feature <epic#> <sub-issue#> ...` (limit the delivery to these sub-issues).

1. `feature-starter` — branch `<type>/<id>-<slug>` from the latest `release/*`, In progress.
2. Base context from `specs/AGENT.md`: mission, tech stack, accepted ADRs.
3. `task-context` — task summary; relevance check against later decisions.
4. Code research with `Explore`.
5. Approach agreement with you → issue comment `## Подход к реализации`.
6. Plan: `specs/features/<id>-<slug>/` with `plan.md`, `context.md`, `validation.md`, first commit.
7. `implementer` per group → one commit per sub-issue; deviations → issue comments `## Отклонение от подхода`.
8. `validator` — up to two fix rounds.
9. `pr-opener` — one PR into `release/*`, In review.
10. After you merge: `feature-finisher` — closes the sub-issues, the epic once all are closed, cleans up.

It stops for you only at: a mismatch between the issue and later decisions, approach agreement, an implementer blocked after a retry, a decision that affects other tasks (ADR), validation failing after two rounds, PR review.

**Requires:** `specs/AGENT.md`, `specs/mission.md`, `specs/tech-stack.md`; the agents in `.claude/agents/`. Design: `docs/analysis/8-create-sdd-feature.md`.

## TEMPLATES

Artifacts of the multi-agent `create-sdd-feature` flow (design: `docs/analysis/8-create-sdd-feature.md`).

- `specs/AGENT.md` — a project file, filled in like `mission.md`. Tells agents what to read in `specs/` always (mission, tech stack, accepted decisions) and what by topic, and where decisions go.
- `.claude/templates/sdd/adr.md` — a cross-cutting decision, saved as `specs/decisions/<id>-<slug>.md`. Accepted ADRs are never edited; a new one supersedes the old.
- `.claude/templates/sdd/plan.md` — the plan of one task, for the human: task groups with goal, files, reuse and done-when. No code.
- `.claude/templates/sdd/context.md` — the code map of one task, for agents: paths, symbols, patterns.
- `.claude/templates/sdd/validation.md` — the validator's checklist: automated and manual checks.

A task's files live in `specs/features/<id>-<slug>/`.
