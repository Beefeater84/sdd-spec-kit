# SDD Validate

Guides the developer through the validation phase after a feature is implemented.

## Step 1 — Load the Feature Spec

If `$ARGUMENTS` is provided, read `specs/features/<argument>.md`.
Otherwise, list all feature specs with status `in-progress` and ask the user which one to validate.

## Step 2 — Run the Validation Scorecard

Go through each item in the feature spec's **Validation Scorecard**:
- Run automated checks (tests, lint, type-check) if configured in `specs/tech-stack.md`
- For manual checks, prompt the user to confirm each one

Report which items pass and which fail.

## Step 3 — Review the Commit Diff

Ask the user to review the commit diff. Focus on:
- Does the implementation match the spec?
- Are there missing requirements or unexpected behaviour?

If issues are found: fix both the code and the spec together. Never let them drift.

## Step 4 — Sync Artifacts

After any fixes, verify that these are still in sync:
- Feature spec task groups match what was actually built
- `specs/roadmap.md` reflects the feature status
- Any impacted constitution files (`mission.md`, `tech-stack.md`) are updated

## Step 5 — Mark Complete and Merge

When all scorecard items pass:
1. Update the feature spec status to `done`
2. Check off the feature in `specs/roadmap.md`
3. Merge the feature branch into main
