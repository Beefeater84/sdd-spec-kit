# Plan: #13 Validate release/* in releaser, remove sdd-validate command

Issue: #13 · Approach: https://github.com/Beefeater84/sdd-spec-kit/issues/13#issuecomment-5846533712
Delivery: -
Later: -

## Group 1: releaser runs project checks before the release PR

- Task: #13
- Goal: in run 1 the releaser runs the project checks on the up-to-date `<release>` in the current working tree and does not open the release PR if any check fails; the result says what failed and why.
- Files: `.claude/agents/releaser.md` (change)
- Reuse: `.claude/agents/validator.md` — Step 2 "Project checks" (`checks` line format, `skipped — not configured`); `.claude/agents/feature-finisher.md` — clean-tree check and `git switch` + fast-forward of the release branch.
- Done when: releaser.md has a run-1 step after "Checks before the merge" that (a) STOPs on a dirty tree, (b) switches to `<release>` if needed and fast-forwards it from origin, (c) takes commands from `## Checks` in `specs/tech-stack.md`, else `package.json` scripts `typecheck`/`lint`/`test`, else skips with a warning, (d) STOPs on any failed check with the failing command and reason; both `RELEASER_RESULT` blocks have a `checks:` key; frontmatter, intro and hard rules no longer say "no tests" and forbid `*:prod`, fix-mode tools and migrations; step numbers and cross-references are consistent; release notes are unchanged.
- Complexity: complex

## Group 2: docs, Checks section, remove sdd-validate

- Task: #13
- Goal: docs describe the new releaser behavior, projects declare their checks in `## Checks`, and `/sdd-validate` is gone.
- Files: `.claude/commands/sdd-validate.md` (delete), `README.md` (change), `docs/agents/releaser.md` (change), `docs/analysis/8-create-sdd-feature.md` (change), `specs/tech-stack.md` (change), `.claude/commands/sdd-init-legacy.md` (change), `.claude/agents/validator.md` (change)
- Reuse: `docs/agents/validator.md` — example result block with failing checks.
- Done when: no mention of `sdd-validate` anywhere except git history and `specs/features/13-*`; README releaser row and `docs/agents/releaser.md` mention the checks, show an example with a failed check and say that the calling session fixes failures with the human; `## Checks` exists in `specs/tech-stack.md` and in the tech-stack template of `sdd-init-legacy.md`; validator Step 2 names `## Checks` in `specs/tech-stack.md` as a source.
- Complexity: normal
