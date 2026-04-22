# SDD Validate

Guides the developer through the validation phase after a feature is implemented.

## Step 1 — Load the Feature Spec

If `$ARGUMENTS` is provided, find the matching folder in `specs/features/` and read `plan.md`, `requirements.md`, `validation.md` from it.
Otherwise, list all feature folders in `specs/features/` with status `in-progress` in their `plan.md` and ask the user which one to validate.

## Step 2 — Run the Validation Scorecard

Go through each item in `validation.md`'s **Scorecard**:
- Use the libraries and tools listed in `validation.md` to run automated checks
- For manual checks, prompt the user to confirm each one

Report which items pass and which fail.

## Step 3 — Review the Commit Diff

Ask the user to review the commit diff. Focus on high-level concerns:
- Does the implementation match the spec?
- Are there missing requirements or unexpected behaviour?
- Avoid nitpicking style details (variable names, formatting) — focus on correctness and intent.

If issues are found: fix both the code and the spec together. Never let them drift.

If a decision was made during review that wasn't in the original spec, add it to the **Key Decisions** section. Omissions are not failures — they are spec evolution.

## Step 4 — Deep Review (optional, recommended for large features)

For complex features, spawn sub-agents to do a deep review of the entire project with this feature applied:

```
Spawn several sub-agents to do a deep review of the project with the changes from this feature branch.
Each sub-agent should focus on a different concern: correctness, consistency with specs, edge cases, test coverage.
Report findings and fix any issues found.
```

Using sub-agents preserves the main context window. This step often catches issues a single pass misses.

## Step 5 — Sync Artifacts

After any fixes, verify that these are still in sync:
- Feature spec task groups match what was actually built
- `specs/roadmap.md` reflects the feature status
- Any impacted constitution files (`mission.md`, `tech-stack.md`) are updated

## Step 6 — Mark Complete and Merge

When all scorecard items pass:
1. Update the feature spec status to `done`
2. Check off the feature in `specs/roadmap.md`
3. If a changelog skill is configured, run it now to document changes
4. Merge the feature branch into main
