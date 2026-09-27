# Plan: #45 Stable release branch name with tracked release versions

Issue: #45 · Approach: https://github.com/Beefeater84/sdd-spec-kit/issues/45#issuecomment-5856023578
Delivery: -
Later: -

## Group 1: releaser on a permanent staging branch

- Task: #45
- Goal: `releaser` releases `staging` into `main`; the version comes from the open release PR title, an explicit version, or the latest `v*` tag + bump; no next release branch is created.
- Files: `.claude/agents/releaser.md` (change), `docs/agents/releaser.md` (change)
- Reuse: `releaser.md` Step 2 bump and higher-than check; Step 4/7 peeled tag lookup; Step 8 ff of `main`; README INSTALL step 2 tag listing
- Done when: no `release/` and no `next_release`/`next_branch_state` in both files; steps renumbered with consistent cross-references; release notes only include PRs whose merge commit is in `origin/main..origin/staging`; docs page examples match the result keys.
- Complexity: complex

## Group 2: flow agents use staging as base

- Task: #45
- Goal: `feature-starter`, `feature-finisher`, `pr-opener`, `validator` use the fixed base `staging`.
- Files: `.claude/agents/feature-starter.md`, `.claude/agents/feature-finisher.md`, `.claude/agents/pr-opener.md`, `.claude/agents/validator.md`, `docs/agents/feature-starter.md`, `docs/agents/feature-finisher.md`, `docs/agents/pr-opener.md` (all change)
- Reuse: `feature-starter.md` Step 5 origin/local sync check; `feature-finisher.md` Step 6 ff logic
- Done when: no `release/` in these files; starter STOPs without `origin/staging` (hint to push a local-only `staging`); version-based picking removed; pr-opener STOPs when base is not exactly `staging`; result keys unchanged; step references consistent.
- Complexity: normal

## Group 3: rules, commands, README and ADR

- Task: #45
- Goal: project rules, commands and install guide describe `staging`, with a migration path from `release/*`; the scheme is recorded in an ADR.
- Files: `agent.md`, `README.md`, `.claude/commands/create-sdd-feature.md`, `.claude/commands/sdd-init-legacy.md` (change), `specs/decisions/45-stable-release-branch.md` (create)
- Reuse: ADR template `.claude/templates/sdd/adr.md`; the Branching text in `agent.md` (keep identical in README INSTALL step 7 and `create-sdd-feature.md` Rules)
- Done when: `grep -rn 'release/' --exclude-dir=.git --exclude-dir=worktrees .` finds only `specs/features/*`, `docs/analysis/8-*`, and the migration text in README/ADR; INSTALL step 4 covers staging / only release/* / none; step 7 replaces an outdated Branching section on update.
- Complexity: normal
