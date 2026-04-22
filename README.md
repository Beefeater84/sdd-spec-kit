# Dev Processes

Development standards, templates, and tools for project setup and feature delivery.

## SKILLS

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
