# SDD Replan

Guides the developer through a replanning session between features.

## Step 1 — Load the Constitution

Read these files:
- `specs/mission.md`
- `specs/tech-stack.md`
- `specs/roadmap.md`

Also read any existing feature specs in `specs/features/`.

## Step 2 — Review What Changed

Ask the user:
1. Did anything change in the product direction since the last feature? (e.g. new stakeholder input, user feedback, new constraints)
2. Does the tech stack need updates? (e.g. new testing framework, tooling, policies)
3. Are there any spec-vs-code drifts to fix?

## Step 3 — Apply Constitution Updates

If the user has changes to the constitution:
- Update the relevant `specs/` files
- If the change affects existing feature specs or code, ask the user whether to propagate it now (small change) or schedule it as a new roadmap feature (large change)
- Propagate small changes: update affected feature specs and code in the same branch

## Step 4 — Review the Roadmap

Show the current roadmap phases. Ask:
- Does the next feature still make sense?
- Should any upcoming features be merged, split, or reordered?

Update `specs/roadmap.md` to reflect decisions.

## Step 5 — Remind to Commit and Branch

Remind the user:
- Do replanning work on its own branch (`replan/<topic>`) to track which version of the constitution produced which code
- Commit in small steps — one concern per commit
- Merge when the constitution and roadmap are stable
