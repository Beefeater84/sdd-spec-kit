# Create SDD Feature

You are helping the developer create a feature using Spec-Driven Development (SDD).

## Step 1 — Load the Constitution

Read these files before anything else:
- `specs/mission.md`
- `specs/tech-stack.md`
- `specs/roadmap.md`

If any of these files are missing or empty, warn the user and stop.

## Step 2 — Identify the Feature

If `$ARGUMENTS` is provided, use it as the feature name.
Otherwise, show the user the roadmap and ask which feature to work on next.

Ask the user to confirm the feature name and which roadmap phase it belongs to.

## Step 3 — Create the Feature Spec

Create a file at `specs/features/<kebab-case-feature-name>.md` with this structure:

```markdown
# Feature: <Feature Name>

**Phase:** <phase from roadmap>
**Branch:** feature/<kebab-case-feature-name>
**Status:** planning | in-progress | done

## Goal
<!-- One sentence: what does this feature deliver? -->

## Requirements
<!-- Specific, testable requirements. Pin versions, enforce rules. -->
-

## Task Groups

### Group 1: <name>
<!-- What to build -->
-

### Group 2: <name>
-

<!-- Add groups as needed. Keep each group small enough to review in one commit. -->

## Key Decisions
<!-- Decisions made during spec discussion. Format: Decision — choice — reason -->

## Validation Scorecard
<!-- How to confirm the feature works. Manual steps or automated checks. -->
- [ ]
```

## Step 4 — Clarify with the User

Review the draft spec with the user. Ask about:
- Any business rules or constraints not obvious from the constitution
- Preferred validation method (curl, browser, automated tests)
- Whether any task groups should be split for sensitive areas (security, database schema changes)

Update the spec based on their answers.

## Step 5 — Implementation Prompt

Once the spec is approved, output a ready-to-use implementation prompt:

```
Implement all task groups from specs/features/<feature-name>.md.
Work through each group in order. Commit after each group.
Run the validation scorecard checks when done.
```

Tell the user: clear agent context with `/clear` before running this prompt.

## Step 6 — Validation Reminder

After implementation, remind the user to:
1. Review the commit diff — focus on whether features match the spec, not style details
2. If any code changes require spec updates, ask the agent to update both together to avoid drift
3. Update `specs/roadmap.md` to mark the feature as complete
4. Merge the branch
