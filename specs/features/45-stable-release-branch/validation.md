# Validation: #45 Stable release branch name with tracked release versions

## Tools

- `grep`, `git`, `bash -n` for extracted snippets (the repo has no build, lint or tests; `## Checks` in `specs/tech-stack.md` is empty)

## Checklist

- [ ] No versioned release branch is required anywhere outside history and migration text — auto: `grep -rn 'release/' --exclude-dir=.git --exclude-dir=worktrees . | grep -v '^./specs/features/' | grep -v '^./docs/analysis/8-'` prints only migration lines in `README.md` and `specs/decisions/45-*`
- [ ] Releaser has no next-branch leftovers — auto: `! grep -n 'next_release\|next_branch_state\|next_version' .claude/agents/releaser.md docs/agents/releaser.md`
- [ ] Releaser step cross-references are consistent — auto: `grep -n 'Step [0-9]' .claude/agents/releaser.md` (every referenced step exists with the intended title)
- [ ] Starter and finisher step references are consistent — auto: `grep -n 'Step [0-9]' .claude/agents/feature-starter.md .claude/agents/feature-finisher.md`
- [ ] Result keys in agents match docs/agents examples — auto: compare the `RELEASER_RESULT`, `FEATURE_STARTER_RESULT`, `FEATURE_FINISHER_RESULT`, `PR_OPENER_RESULT` key lists in `.claude/agents/*.md` with `docs/agents/*.md`
- [ ] Release notes are limited to PRs not yet in main — auto: `grep -n 'origin/main..origin/staging' .claude/agents/releaser.md`
- [ ] pr-opener accepts only `staging` as base — auto: `grep -n 'staging' .claude/agents/pr-opener.md`
- [ ] ADR exists with Status accepted and Affects #43 — auto: `grep -E '^Status: accepted|^Affects: .*#43' specs/decisions/45-stable-release-branch.md`
- [ ] Branching rule says `staging` in all three places — auto: `grep -n 'staging' agent.md README.md .claude/commands/create-sdd-feature.md`
- [ ] Bash snippets in changed agents parse — auto: extract fenced bash blocks of changed `.claude/agents/*.md` and run `bash -n` on each (placeholders may need quoting)
- [ ] Migration of this repository — manual: after merge, `git push origin origin/release/0.1.0:refs/heads/staging`, retarget open PRs, delete `release/0.1.0` in origin
- [ ] Live run — manual: next `/create-sdd-feature` starts from `origin/staging`; next release run opens `Release v0.1.0` from `staging` into `main`

## Smoke tests

- Read `feature-starter.md` Step 4 and `releaser.md` Steps 1–3 as haiku would: every command is written out, every branch ends in go on / SKIP / STOP.
