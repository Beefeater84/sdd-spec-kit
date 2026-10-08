# Validation: #14 Remove sdd-replan command

## Tools

- No typecheck, lint or tests are configured in this repo (Markdown only). `grep`, `git`.

## Checklist

- [ ] The command file is gone — auto: `test ! -e .claude/commands/sdd-replan.md`
- [ ] No mentions outside feature folders — auto: `! grep -rn 'sdd-replan' --exclude=.git --exclude-dir=.git --exclude-dir=features .`
- [ ] README has no doubled `---` separators around the removed section — auto: `sed -n '/^## SKILLS/,/^## AGENTS/p' README.md` shows each `---` between two sections, never two in a row
- [ ] Only the four planned files changed besides the feature folder — auto: `git diff --stat <base>...HEAD -- . ':!specs/features'`

## Smoke tests

- Read README `## SKILLS`: the list of commands flows without a gap where `/sdd-replan` was.
