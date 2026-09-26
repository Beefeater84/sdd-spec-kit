# Validation: #38 Remove roadmap and main leftovers from sdd-init-legacy

## Tools

- No typecheck, lint or tests are configured in this repo (Markdown only). `grep`, `git`.

## Checklist

- [ ] The kit's roadmap file is gone — auto: `test ! -e specs/roadmap.md`
- [ ] The command mentions neither roadmap nor main — auto: `! grep -niE 'roadmap|\bmain\b' .claude/commands/sdd-init-legacy.md`
- [ ] No roadmap mentions in active files — auto: `! grep -rni 'roadmap' --exclude-dir=.git --exclude-dir=features --exclude-dir=worktrees --exclude-dir=analysis . | grep -v 'There is no roadmap file'`
- [ ] Step 3 creates AGENT.md and decisions/ — auto: `grep -n 'specs/AGENT.md' .claude/commands/sdd-init-legacy.md && grep -n 'specs/decisions' .claude/commands/sdd-init-legacy.md`
- [ ] README lists the new set — auto: `sed -n '/sdd-init-legacy/,/^---/p' README.md | grep -E 'AGENT.md' `
- [ ] Only planned files changed besides the feature folder — auto: `git diff --stat <base>...HEAD -- . ':!specs/features'` shows only `.claude/commands/sdd-init-legacy.md`, `README.md`, `specs/roadmap.md`
- [ ] The generated AGENT.md block matches the kit's `specs/AGENT.md` in structure — manual: compare sections (Always read, Read by topic, Feature folders, Where decisions go)

## Smoke tests

- Read `/sdd-init-legacy` top to bottom: steps flow, file count matches the blocks, final hint points to `/create-sdd-feature <issue>`.
