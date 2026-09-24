# Dev Processes

Development standards, templates, and tools for project setup and feature delivery.

## AGENTS

### `feature-starter`

First step of the SDD multi-agent flow (`.claude/agents/feature-starter.md`, model `haiku`, tools: `Bash`).

**Input:** a GitHub issue number, or a task description (the agent then creates the issue with `gh issue create`).

The agent fetches `origin`, picks the highest `origin/release/*` by version (`1.10` > `1.9`), checks it matches `origin`, and creates a local branch `<type>/<id>-<slug>` from it without upstream. It never pushes and never uses `main`/`master`. It stops if there is no `origin/release/*`, if the branch already exists, or if the working tree is dirty.

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

On failure: `status: error`, `id`, `reason`.

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
