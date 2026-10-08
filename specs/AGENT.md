# Agent guide to `specs/`

This file tells agents what to read in `specs/`. `README.md` files are for humans.
Do not read the whole folder. Read the base set, then only what the task needs.

## Always read

- `mission.md` — what the project is and its business rules.
- `tech-stack.md` — stack, infrastructure, constraints, code style.
- Accepted decisions in `decisions/`. List them with:

  ```bash
  grep -rl --include='*.md' '^Status: accepted' specs/decisions/
  ```

  Read the title, `Affects` and `Decision` of each. Read the rest only if the task touches it.

## Read by topic

<!-- Add one row per doc area of this project. Keep the "When" column specific. -->

| When the task touches | Read |
|---|---|
| <!-- e.g. billing --> | <!-- e.g. `specs/modules/billing.md` --> |

## Feature folders

`features/<id>-<slug>/` holds one task's `plan.md`, `context.md` and `validation.md`.
Read only the folder of the current task. Read another one only when the task context points to it.

## Where decisions go

- Affects only the current task: a deviation comment in its GitHub issue (`## Отклонение от подхода`).
- Changes the approach for other tasks: an ADR in `decisions/<id>-<slug>.md` (template: `.claude/templates/sdd/adr.md`).
- Never edit an accepted ADR. Write a new one and set the old one to `superseded by <file>`.
