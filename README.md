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
