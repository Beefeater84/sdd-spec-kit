# Create SDD Feature

You are helping the developer create a feature using Spec-Driven Development (SDD).

## Step 1 — Pre-flight Check

Before anything else, confirm with the user:
- [ ] Previous feature branch is merged to main
- [ ] Agent context is cleared (`/clear`)
- [ ] Roadmap still reflects the right next feature

If anything is unfinished, stop and help the user resolve it first. A clean start prevents AI fatigue and context bleed.

## Step 2 — Load the Constitution

Read these files:
- `specs/mission.md`
- `specs/tech-stack.md`
- `specs/roadmap.md`

If any of these files are missing or empty, warn the user and stop.

## Step 3 — Identify the Feature

If `$ARGUMENTS` is provided, use it as the feature name.
Otherwise, show the user the roadmap and ask which feature to work on next.

Ask the user to confirm the feature name and which roadmap phase it belongs to.

## Step 4 — Clarify with the User

Before creating any files, discuss the feature with the user. Ask about:
- Any business rules or constraints not obvious from the constitution
- Which modules or packages will be needed
- Preferred validation method (curl, browser, automated tests, specific libraries)
- Whether any task groups should be split for sensitive areas (security, database schema changes)

## Step 5 — Create the Feature Spec Folder

Create a folder at `specs/features/YYYY-MM-DD-<kebab-case-feature-name>/` using today's date.

Inside it, create three files:

**`plan.md`** — the agent fills this based on the discussion:
```markdown
# Plan: <Feature Name>

**Phase:** <phase from roadmap>
**Branch:** feature/<kebab-case-feature-name>
**Status:** planning | in-progress | done

## Goal
<!-- One sentence: what does this feature deliver? -->

## Task Groups

### Group 1: <name>
-

### Group 2: <name>
-

<!-- Keep each group small enough to review in one commit. -->

## Key Decisions
<!-- Format: Decision — choice — reason -->
```

**`requirements.md`** — the agent fills this based on the discussion:
```markdown
# Requirements: <Feature Name>

## Functional Requirements
-

## Modules & Packages
<!-- Libraries and versions needed for this feature. Pin versions. -->
-

## Constraints
<!-- Rules that must be followed: TypeScript strict, no X, always Y -->
-
```

**`validation.md`** — the agent fills this based on the discussion:
```markdown
# Validation: <Feature Name>

## Libraries & Tools
<!-- Testing libraries, linters, or other tools used for validation -->
-

## Scorecard
<!-- How to confirm the feature works. -->
- [ ]

## Smoke Tests
<!-- Minimal manual or automated checks to run after implementation -->
-
```

## Step 6 — Implementation Prompt

Once the spec folder is reviewed and approved, output a ready-to-use implementation prompt:

```
Implement all task groups from specs/features/<folder-name>/plan.md.
Follow requirements in requirements.md.
Work through each group in order. Commit after each group.
Run validation checks from validation.md when done.
```

Tell the user: clear agent context with `/clear` before running this prompt.

## Step 7 — Validation Reminder

After implementation, remind the user to:
1. Review the commit diff — focus on whether features match the spec, not style details
2. If any code changes require spec updates, ask the agent to update both together to avoid drift
3. Update `specs/roadmap.md` to mark the feature as complete
4. Merge the branch
