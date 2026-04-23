# SDD Init Legacy

Bootstraps an SDD constitution for an existing project by reverse-engineering it from the codebase.
Run this once per project. After this, use the standard SDD feature workflow.

## Step 1 — Explore the Codebase

Read as many of these as exist:
- `README.md` — project description, background
- `TODO.md` or any open task lists
- `package.json`, `pyproject.toml`, `go.mod`, or similar — tech stack and dependencies
- Recent git log (`git log --oneline -20`) — history and direction
- Any existing planning documents (issues exports, spreadsheets, Word docs the user can paste in)

Do not ask the user to explain what you can discover yourself.

## Step 2 — Ask for Missing Context

After exploring, ask the user only what you could not determine from the code:
- Who is the primary audience?
- Are there constraints not visible in the code (legal, compliance, budget, performance)?
- Is there a goal or direction beyond what is in README/TODO (e.g. "improve efficiency while shipping feature X")?
- Are there preferred tools or libraries the team uses that are not yet in the code?

## Step 3 — Create the `specs/` Folder

Create three files. Fill them from what you discovered — do not leave placeholders empty if the answer is already in the code.

**`specs/mission.md`**
```markdown
# Mission

## Vision
<!-- Why does this project exist? What problem does it solve? -->

## Target Audience
<!-- Who are the primary users? How do they interact with the system (UI / API / both)? -->

## Core Business Rules
<!-- Rules the agent cannot infer on its own: lifecycle states, thresholds, algorithms, custom logic. -->

## Key Decisions
<!-- Decisions already made in the codebase. Format: Decision — choice — reason -->

## Scope
<!-- In scope / Out of scope -->

## Constraints
<!-- Non-negotiable constraints: legal, compliance, budget, time -->
```

**`specs/tech-stack.md`**
```markdown
# Tech Stack

## Languages & Frameworks
<!-- Extracted from project files and code -->

## Infrastructure & Deployment
<!-- Hosting, CI/CD, environments -->

## Key Dependencies
<!-- Critical libraries, external services, APIs -->

## Database Schema
<!-- Tables and key fields found in migrations or models -->

## Key Flows
<!-- Critical request pipelines or data flows discovered in the code -->

## Smoke Tests
<!-- Basic checks to verify the system works end-to-end -->

## Constraints
<!-- Technical constraints observed in the codebase -->
```

**`specs/roadmap.md`**
```markdown
# Roadmap

<!-- Derived from TODO, issues, git history, and user input. Living document — update after each feature. -->

## Phase 1: [Name]
**Goal:**
**Features:**
-

## Phase 2: [Name]
**Goal:**
**Features:**
-
```

## Step 4 — Review with the User

Walk the user through each file. Ask:
- Does this reflect the project's actual direction?
- Are there roadmap items missing or in the wrong order?
- Any decisions in the code that need a different explanation?

Update based on their answers.

## Step 5 — Commit

Remind the user to commit the constitution on its own branch (e.g. `chore/sdd-init`) before merging to main.

Specs are part of the versioning strategy — future changes to the constitution should be traceable in git history.

## Step 6 — What's Next

Tell the user:
> The project now has an SDD foundation. From here, use `/create-sdd-feature` to plan and implement the next roadmap item. Allow time for replanning (`/sdd-replan`) after the first feature — you may find things to tune now that specs are in place.
