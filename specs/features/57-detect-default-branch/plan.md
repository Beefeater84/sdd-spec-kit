# Plan: #57 releaser: detect the default branch instead of hardcoding main

Issue: #57 · Approach: https://github.com/Beefeater84/sdd-spec-kit/issues/57#issuecomment-6437825428
Delivery: -
Later: -

## Group 1: releaser agent resolves the default branch

- Task: #57
- Goal: `releaser` resolves the default branch in Step 1 as `<main>` and uses it everywhere it used `main`.
- Files: `.claude/agents/releaser.md` (change)
- Reuse: Step 1 of `releaser.md` — `gh repo view` and the "prints X. Use them below." placeholder style; STOP with `reason`/`hint` like Step 1 and `feature-starter.md` Step 4.
- Done when: the lookup (gh primary, `origin/HEAD` fallback, STOP when neither works, STOP when it is `staging`) is in Step 1; no command, STOP reason, hint or text hardcodes `main` except the never-touch rule and the `main_state` key; step numbers are unchanged.
- Complexity: normal

## Group 2: docs say "the default branch"

- Task: #57
- Goal: the releaser docs and README describe releasing into the default branch, not `main`.
- Files: `docs/agents/releaser.md` (change), `README.md` (change)
- Reuse: existing "default branch" wording in `README.md` and `docs/agents/feature-starter.md`.
- Done when: `docs/agents/releaser.md` mirrors the new Step 1 behaviour and explains that `main_state` refers to the default branch; README lines 15 and 154 say the default branch (`main` or `master`); rule-only mentions of `main`/`master` and ADR 45 are untouched.
- Complexity: normal
