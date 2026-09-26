# Context: #14 Remove sdd-replan command

## Code map

- `.claude/commands/sdd-replan.md` — the whole command; delete.
- `README.md` — l.56–62 `### \`/sdd-replan\`` section under `## SKILLS`; sections are separated by `---`, keep exactly one separator between the neighbours.
- `docs/analysis/8-create-sdd-feature.md` — l.217 "Staleness is always explicit. It is caught by `task-context`, PR review and `sdd-replan`." → drop `sdd-replan`; l.292 "Outside the epic: #13 (...), #14 (`sdd-replan`)." → say the command was removed.
- `.claude/commands/sdd-init-legacy.md` — l.117 closing note: drop the sentence about replanning (`/sdd-replan`). Leave the rest (roadmap, main) as is: out of scope.

## Conventions

- Markdown only; no typecheck, lint or tests in this repo.
- Do not touch `specs/features/*` of other tasks or `specs/roadmap.md`.
- Short, plain English sentences.
