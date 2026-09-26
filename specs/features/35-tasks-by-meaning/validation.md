# Validation: #35 Name tasks by meaning, not by number, in texts for the human

## Tools

- grep (no code, no test suite in this repo)

## Checklist

- [ ] `agent.md` has the rule — auto: `grep -n "3–5" agent.md`
- [ ] `create-sdd-feature.md` Rules has the rule — auto: `sed -n '/^## Rules/,/^## Human stops/p' .claude/commands/create-sdd-feature.md | grep -n "(#"`
- [ ] Final report points to the rule — auto: `sed -n '/^## Final report/,$p' .claude/commands/create-sdd-feature.md | grep -in "name\|rule"`
- [ ] The next `/create-sdd-feature` run reports tasks by meaning — manual: run the command on any task and read its stage reports and stops

## Smoke tests

- Read the changed sections: rule is consistent in both files, machine places keep bare numbers.
