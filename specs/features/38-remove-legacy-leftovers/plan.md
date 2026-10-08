# Plan: #38 Remove roadmap and main leftovers from sdd-init-legacy

Issue: #38 · Approach: https://github.com/Beefeater84/sdd-spec-kit/issues/38#issuecomment-5847053950
Delivery: -
Later: -

## Group 1: sdd-init-legacy creates the current constitution

- Task: #38
- Goal: `/sdd-init-legacy` creates `specs/AGENT.md`, `specs/mission.md`, `specs/tech-stack.md` and `specs/decisions/`, with no roadmap and no `main`; the kit's own `specs/roadmap.md` is gone.
- Files: `.claude/commands/sdd-init-legacy.md` (change), `README.md` (change), `specs/roadmap.md` (delete)
- Reuse: `specs/AGENT.md` — the sample for the generated `AGENT.md`; `agent.md` Branching and `.claude/commands/create-sdd-feature.md` Rules — wording for issues/board and `release/*`
- Done when: the command and README mention neither roadmap nor `main`, Step 3 lists the AGENT.md block and `decisions/`, `specs/roadmap.md` does not exist
- Complexity: normal
