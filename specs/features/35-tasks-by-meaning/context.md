# Context: #35 Name tasks by meaning, not by number, in texts for the human

## Code map

- `agent.md` — project rules of this repo (sections: Task tracking, Branching). Loaded via `CLAUDE.md` (`@agent.md`). NOT shipped to projects installing the kit (README.md:59).
- `.claude/commands/create-sdd-feature.md` — orchestrator command. `## Rules` (bullets, bold lead-in), `## Human stops`, stage 5 approach template (`### Состав поставки` comment: sub-issues in order), stage 7 deviation comments, stage 9 `pr-opener` input, `## Final report` (last section, one paragraph).
- `.claude/agents/pr-opener.md` — PR body already lists `- #<n> <title>`; no change.

## Patterns to follow

- Rule bullets in `create-sdd-feature.md` `## Rules`: `- **Name.** One or two sentences.`
- `agent.md` sections: `## Heading`, short prose + bullets, English.

## Conventions

- Docs are in English; comment headings and human-facing examples may be Russian (e.g. «эпик „create-sdd-feature как оркестратор агентов“ (#8)»).
- Numbers stay in machine places: branch names, commit messages, agent briefings and result blocks, `gh` commands, file/folder names.
- Rule format: short name of 3–5 words, then `(#N)`. Scope: stage reports, stops, final report, issue comments, PR description.
- Keep the command copy short; full wording lives in `agent.md`.
