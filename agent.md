# Agent Context

## Task tracking

All tasks for this repository (epics, features, bugs) live in GitHub, not in files in this repo.

- **Repository:** [Beefeater84/sdd-spec-kit](https://github.com/Beefeater84/sdd-spec-kit)
- **Project board:** [@Beefeater84's Spec kit](https://github.com/users/Beefeater84/projects/1) (owner `Beefeater84`, project number `1`)
- **Tasks:** GitHub Issues of this repository. The issue number is the task id and is used in every artifact (branch, plan, commits, PR).
- **Epics:** issues with sub-issues. A feature is a sub-issue of its epic.

Board fields:

| Field | Values |
|---|---|
| Status | Backlog, Ready, In progress, In review, Done |
| Priority | P0, P1, P2 |
| Size | XS, S, M, L, XL |

Use the `gh` CLI to read and update tasks, for example `gh issue view <n>` and `gh project item-list 1 --owner Beefeater84`.

## Branching

- Feature branches are created from the latest `release/*` branch in `origin` and merged back into it.
- Never branch from or target `main` or `master`. They hold released code only.
- Branch name: `<type>/<issue-number>-<short-description>`, where type is one of `feat`, `fix`, `docs`, `refactor`, `chore`.

## Naming tasks in texts for the human

- **Scope.** Any text written for the human to read: stage reports, stops, final reports, issue comments, PR descriptions.
- **Format.** A short name of the task's meaning (3–5 words), then the number in parentheses: `<short meaning> (#N)`. The name may be in the language of the surrounding text.
- **Example.** «эпик „create-sdd-feature как оркестратор агентов“ (#8)», not «#8».
- **Machine exceptions.** Numbers stay bare where a machine or a person copies them verbatim: branch names, commit messages, agent briefings and result blocks, `gh` commands, file and folder names.
