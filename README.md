# Dev Processes

Development standards, templates, and tools for project setup and feature delivery.

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

The skill explores the codebase autonomously — README, TODO, package files, git log, migrations — and reverse-engineers the three constitution files (`specs/mission.md`, `specs/tech-stack.md`, `specs/roadmap.md`). It only asks the user for context it cannot discover itself (audience, hidden constraints, team direction). After review and commit, the project is on an SDD foundation and ready for the standard feature workflow.

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
