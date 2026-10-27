# Validation: #57 releaser: detect the default branch instead of hardcoding main

## Tools

- `grep`, `git`. The kit has no typecheck, lint or tests (`specs/tech-stack.md` `## Checks` is empty).

## Checklist

- [ ] Step 1 resolves the default branch via `gh repo view --json defaultBranchRef` — auto: `grep -n 'defaultBranchRef' .claude/agents/releaser.md`
- [ ] Fallback via `origin/HEAD` exists — auto: `grep -n 'symbolic-ref' .claude/agents/releaser.md`
- [ ] STOP when neither works and STOP when the default branch is `staging` — auto: `grep -n 'reason:' .claude/agents/releaser.md`
- [ ] No hardcoded `main` left except the never-touch rule and the `main_state` key — auto: `grep -nw 'main' .claude/agents/releaser.md | grep -v 'main_state'` prints only the never-touch rule line
- [ ] Step numbers unchanged (Steps 1–8) — auto: `grep -n '^## Step' .claude/agents/releaser.md`
- [ ] Result block keys unchanged — auto: `git diff origin/staging -- .claude/agents/releaser.md` shows no change to keys in the Output section
- [ ] `docs/agents/releaser.md` says the default branch and explains `main_state` — auto: `grep -n 'default branch' docs/agents/releaser.md`
- [ ] README L15 and the agents table say the default branch — auto: `grep -n 'default branch' README.md`
- [ ] ADR 45 untouched — auto: `git diff --quiet origin/staging -- specs/decisions/`
- [ ] Release in a repo whose default branch is `master` — manual: in a test repo with `master` and `staging`, run releaser twice (open PR, merge, tag) and check PR base, notes range and local `master` fast-forward.
- [ ] Release in a repo whose default branch is `main` — manual: same as above with `main`.

## Smoke tests

- Read Step 1 of `releaser.md` and run its default-branch commands in this repo: they print `main`.
