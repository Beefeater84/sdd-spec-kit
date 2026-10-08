# Validation: #13 Validate release/* in releaser, remove sdd-validate command

## Tools

- No typecheck, lint or tests are configured in this repo (Markdown only). `grep`, `git`.

## Checklist

- [ ] `/sdd-validate` is removed — auto: `test ! -e .claude/commands/sdd-validate.md && ! grep -rn 'sdd-validate' --exclude-dir=.git --exclude-dir=features .`
- [ ] Releaser no longer claims it does not run tests — auto: `! grep -n -i 'no tests\|run tests,' .claude/agents/releaser.md docs/agents/releaser.md`
- [ ] Both RELEASER_RESULT blocks have a `checks:` key — auto: `grep -c '^checks:' .claude/agents/releaser.md` prints 2 or more
- [ ] `## Checks` section exists in tech-stack and in the sdd-init-legacy template — auto: `grep -n '^## Checks' specs/tech-stack.md .claude/commands/sdd-init-legacy.md`
- [ ] Validator names `## Checks` as a source — auto: `grep -n 'Checks' .claude/agents/validator.md`
- [ ] Step numbering and cross-references in releaser.md are consistent — auto: `grep -n 'Step [0-9]' .claude/agents/releaser.md` (every referenced step exists with the intended title)
- [ ] Release notes generation (Step with `gh pr list --base "<release>" --state merged`) is unchanged — auto: `git diff <base> -- .claude/agents/releaser.md` shows no change inside the jq notes block
- [ ] Releaser switches branch only on a clean tree and updates it fast-forward only — manual: read the new step
- [ ] Live run: on the next release, run releaser from a feature branch with a clean tree; it switches to `release/*`, runs checks (here: skipped, not configured, with a warning) and opens the PR — manual

## Smoke tests

- Read the new releaser step as haiku would: every command is written out, every branch ends in go on / SKIP / STOP.
