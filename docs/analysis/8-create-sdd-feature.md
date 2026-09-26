# Analysis: create-sdd-feature as a multi-agent flow

Epic: #8. Task: #10. Follow-up: #9 (rewrite the command) and sub-issues #15–#21.

This document records the design agreed in the #8 discussion (summary comments, parts 1–5). It is the source for the sub-agent issues and for rewriting `.claude/commands/create-sdd-feature.md`.

## 1. Problems in the current command

| Step | Problem |
|---|---|
| 1. Pre-flight | Requires "previous branch merged to main". Our base and target is `release/*`, never `main`. `/clear` is not needed when sub-agents do the work: each starts with a fresh context. |
| 3. Identify feature | Takes the feature from `specs/roadmap.md`. Tasks live in GitHub Issues. There is no task id. |
| 5. Spec folder | Branch `feature/<name>` in `plan.md` vs `<type>/<id>-<slug>` in the agents. Folder `YYYY-MM-DD-<name>` has no id. `Status` in `plan.md` duplicates the board. `Key Decisions` in `plan.md` are invisible to later features. |
| 7. Reminder | "Merge the branch": no push, no PR into `release/*`. |
| — | No board statuses In progress and In review. |

## 2. Principles

- **The issue is the source of truth** for what a task is. The roadmap file is dropped.
- **The main agent is an orchestrator.** It gathers context, agrees the approach with the human, writes the plan, hands task groups to sub-agents and reads their reports. It does not write code or read diffs.
- **Maximum autonomy.** Agents decide and record their decisions; the human reviews them in the PR. Mistakes are analyzed and turned into process fixes. Agents stop only at the points listed in section 4.
- **Only the orchestrator writes to GitHub** during the flow (except the mechanical agents: `feature-starter`, `pr-opener`, `feature-finisher`).
- **Context is gathered once.** Code research is saved to `context.md` and sliced per task group, so implementers do not search the codebase again.

## 3. Flow

```
/create-sdd-feature <issue#>
 1. feature-starter       (haiku)        branch <type>/<id>-<slug> from release/*, In progress
 2. base context          (orchestrator) specs/AGENT.md → mission, tech-stack, codestyle, ADRs
 3. task-context          (haiku)        issue, epic, siblings, ADRs, open PRs of the epic
    relevance check       (orchestrator) mismatch → human
 4. code research         (Explore → code-scout if needed)
 5. approach agreement    (orchestrator + human) → issue comment "## Подход к реализации"
 6. plan                  (orchestrator) plan.md + context.md + validation.md, first commit
 7. implementer × N       (sonnet/opus)  one group → commit → IMPLEMENTER_RESULT
 8. validator             (sonnet)       VALIDATOR_RESULT, up to 2 fix rounds
 9. pr-opener             (haiku)        push, PR into release/*, In review
    ── human: review and merge ──
10. feature-finisher      (haiku)        Done, cleanup, update local release/*
```

| # | Stage | Who | Output |
|---|---|---|---|
| 1 | Branch | `feature-starter` | `FEATURE_STARTER_RESULT` |
| 2 | Base context | orchestrator | constitution in the main context |
| 3 | Task context | `task-context` | `TASK_CONTEXT_RESULT` |
| 3a | Relevance check | orchestrator | issue kept, updated, split or closed |
| 4 | Code research | `Explore` / `code-scout` | code map → `context.md` |
| 5 | Approach | orchestrator + human | issue comment |
| 6 | Plan | orchestrator | `specs/features/<id>-<slug>/`, commit |
| 7 | Implementation | `implementer` | commits, `IMPLEMENTER_RESULT` per group |
| 8 | Validation | `validator` | `VALIDATOR_RESULT` |
| 9 | PR | `pr-opener` | `PR_OPENER_RESULT` |
| 10 | Finish | `feature-finisher` | `FEATURE_FINISHER_RESULT` |

### Stage notes

**2. Base context.** The orchestrator reads a fixed base set, not the whole `specs/` folder (in real projects it holds all docs: modules, SEO, features). `specs/AGENT.md` lists what to read always (`mission.md`, `tech-stack.md`, codestyle link, the index of accepted ADRs) and what to read by topic. `README.md` stays for humans.

**3a. Relevance check.** The orchestrator compares the issue with decisions made after it was created (ADRs, closed sibling tasks, approach and deviation comments of siblings). On a mismatch it shows it to the human: "the issue says X, ADR `42-...` changed it". The human chooses: update the issue (body + a comment about the scope change), split or close it, or go on as is.

**4. Code research.** Runs after the relevance check, on the up-to-date task. It finds existing functionality: path — what it does — can it be reused. It exists because models often miss existing code and go the wrong way. First try the built-in `Explore` agent with a fixed prompt; build `code-scout` only if the result is unstable.

**5. Approach agreement.** The main checkpoint before code. There is no separate clarification stage: questions appear only when the context has real gaps, and they are part of the draft. The orchestrator writes the draft in chat, the human edits it, and after "agreed" the orchestrator posts it as an issue comment:

```markdown
## Подход к реализации

### Что уже есть
### Как делаем        <!-- modules and approach, no code -->
### Допущения
### Вопросы           <!-- only if there are real gaps -->
### Риски
```

The heading is fixed so that `task-context` and later agents can find the comment.

**6. Plan.** The orchestrator writes the spec itself (no spec-writer: it already holds all the context). No human approval: the spec is reviewed together with the code in the PR.

**7. Implementation.** See `implementer` in section 5. Problem handling:

| Situation | Orchestrator |
|---|---|
| Implementer decided itself, affects only its group | deviation comment, go on |
| Decision changes later groups | update `plan.md` and `context.md`, deviation comment, go on |
| `status: blocked` | one retry with a better briefing, then stop → human |
| `affects_other_tasks` is not `none` | draft ADR, stop → human |

Deviation comments use the fixed heading `## Отклонение от подхода` (planned / did / why). Several small deviations may go into one comment. The issue becomes a timeline: description → approach → deviations.

**8. Validation.** Failures go back to an implementer (the one that worked in that area, or a fresh one), then the validator runs again. At most two rounds, then stop → human. Manual checks never block: they go into the PR under "How to check".

## 4. Human stops

1. Relevance check found a mismatch between the issue and later decisions.
2. Approach agreement (always).
3. Implementer `blocked` after one retry.
4. A decision affects other tasks (ADR with other issues in `Affects`).
5. Validation still fails after two fix rounds.
6. PR review and merge.

## 5. Agents

### Existing

| Agent | Stage | Change |
|---|---|---|
| `feature-starter` | 1 | **Extend:** set board Status to In progress. |
| `feature-finisher` | 10 | None. |
| `releaser` | outside the flow | None. Runs manually for a release. |

### New

#### `task-context`

- **Purpose:** a compact summary of the task, so raw `gh` JSON does not bloat the orchestrator's context.
- **Model:** haiku. **Tools:** Bash (`gh`, `git`), Read, Grep (for `specs/decisions/`).
- **Input:** issue number.
- **Output:** `TASK_CONTEXT_RESULT`:
  - issue: title, body, comments (summarized), approach and deviation comments if present, creation date;
  - epic: number, title, goal (or `-`);
  - siblings: number, title, state, merged PR;
  - adrs: accepted ADRs that list this issue or its modules in `Affects`, and ADRs created after the issue;
  - in_flight: open PRs of the same epic that add files under `specs/decisions/`.
- **Boundaries:** collects and summarizes only. It does not judge relevance, does not edit issues, does not read code.

#### `code-scout` (only if `Explore` is not enough)

- **Purpose:** find existing code relevant to the task and save it as a map for implementers.
- **Model:** sonnet. **Tools:** Read, Grep, Glob.
- **Input:** up-to-date task summary (after the relevance check), `tech-stack.md` hints.
- **Output:** sections of `context.md`: code map (path — symbols/signatures — notes), patterns to follow (file to copy the style from), reuse candidates.
- **Boundaries:** read-only. Does not propose the approach, does not write files: the orchestrator writes `context.md`.

#### `implementer`

- **Purpose:** implement one task group from `plan.md` with a fresh context.
- **Model:** sonnet by default; opus for groups the orchestrator marks as complex in `plan.md`.
- **Tools:** Read, Edit, Write, Grep, Glob, Bash (git, project lint/typecheck/test). No `gh`.
- **Input (briefing from the orchestrator):** the group from `plan.md`, the slice of `context.md` for its files, what earlier groups created (from their `changed`), commit message format `<type>(#<id>): ...`.
- **Rules:** read the listed files first; a narrow grep for a symbol is fine; a broad search is a gap in the map and goes to `gaps`. Run lint and tests for touched files, then commit the group. Never push.
- **Output:** `IMPLEMENTER_RESULT`:

  ```
  IMPLEMENTER_RESULT
  status: done | blocked
  group: <n>
  commit: <sha>
  changed:
    - <file> — <symbol> — <signature> — <new | changed>
  decisions:
    - <choice not in the plan>
  deviations:
    - plan: <...>; did: <...>; why: <...>
  gaps:
    - <what had to be searched outside context.md>
  affects_other_tasks: none | #<n> — <why>
  checks: <lint/tests run and results>
  blocker: - | <what prevents finishing>
  ```

- **Continuation:** the orchestrator continues the same implementer (`SendMessage`) for the next group in the same code area, and starts a fresh one for a new area or when the context is large. So `plan.md` groups work by code area, not by layer.
- **Orchestrator uses the report:** `changed` → next briefing; `decisions`, `deviations` → deviation comment; `gaps` → added to `context.md`; `affects_other_tasks` → ADR and stop.

#### `validator`

- **Purpose:** independent check of the whole feature. The one who checks is not the one who wrote.
- **Model:** sonnet. **Tools:** Read, Grep, Glob, Bash (typecheck, lint, tests, curl, `git diff`). No Edit/Write.
- **Input:** id, branch, base, path to the feature folder.
- **Checks:** full project typecheck, lint and tests; every item of `validation.md` that can be automated; every `plan.md` group done and nothing extra; no violation of accepted ADRs.
- **Output:** `VALIDATOR_RESULT`: `status: pass | fail`, list of checks with pass/fail and details, `manual:` items the agent cannot check.
- **Boundaries:** never fixes anything. (#13: release validation moved into `releaser`, which runs the project checks before the release PR; the separate validate command was removed.)

#### `pr-opener`

- **Purpose:** publish the branch and open the PR.
- **Model:** haiku. **Tools:** Bash (`git`, `gh`).
- **Input:** id, type, branch, base, title, short summary of what was done, link to the approach comment, links to deviation comments, manual checks from the validator.
- **Actions:** `git push -u origin <branch>`; `gh pr create --base <base>` with title `<type>(#<id>): <title>` and body "What was done / How to check / What is left" plus `Refs #<id>` (not `Closes`: it does not fire on merge into `release/*`); set board Status to In review.
- **Output:** `PR_OPENER_RESULT`: status, PR url, board status.
- **Boundaries:** safe to re-run: an existing PR is reused, not recreated. Never merges.

### Rejected candidates

| Candidate | Why not |
|---|---|
| Constitution loader | The orchestrator needs the constitution in its own context for the relevance check and the approach. A sub-agent would only add a hop. |
| Spec-writer | After the approach is agreed, the orchestrator holds all the context. A sub-agent would need all of it passed again. |
| Roadmap updater | `roadmap.md` is dropped: epics and issues on the board replace it. |
| Clarification agent | Sub-agents cannot talk to the human; clarifications are part of the approach draft. |

## 6. Artifacts

### `specs/AGENT.md`

Instructions for agents: the base set to read always, and which docs to read by topic.

### Decision records (ADR)

Cross-cutting decisions (they change the approach for other tasks) live in `specs/decisions/<id>-<slug>.md`, one file per decision. The file is named by the task id: no number conflicts between parallel branches, and the decision is linked to its task.

```markdown
# <Title>

Status: proposed | accepted | superseded by <file> | deprecated
Date: YYYY-MM-DD
Issue: #<id>
Affects: #<n>, #<m>; <modules>

## Context
## Decision
## Consequences
```

- An accepted ADR is never edited. A new decision is a new ADR; the old one only gets `superseded by`.
- Staleness is always explicit. It is caught by `task-context`, PR review and `sdd-replan`.
- Parallel features: an ADR lives in its branch until merge. Affected open tasks learn about it from a comment in their issue and from `task-context` (open PRs of the epic).
- Local decisions (only this feature) are deviation comments in the issue, not ADRs.

### Feature folder `specs/features/<id>-<slug>/`

| File | Reader | Content |
|---|---|---|
| `plan.md` | human | Link to the issue and the approach comment. Task groups: goal, files (create/change), what is reused, done-when, complexity mark. No code, signatures or step-by-step instructions. |
| `context.md` | agents | Code map (path — symbols/signatures — notes), patterns to follow. Saved code research, extended with implementers' `gaps`. |
| `validation.md` | validator | Tools, checklist, smoke tests. |

Removed: `requirements.md` (requirements are in the issue, packages and constraints in the approach), `Key Decisions` and `Status` in `plan.md`.

### Board statuses

In progress (`feature-starter`) → In review (`pr-opener`) → Done (`feature-finisher`).

## 6a. Unit of delivery (#29)

The first version of this flow treated every issue as its own delivery. Sub-issues #15–#18, #20, #21 of this epic went as separate branches and PRs. They all edited the same place in `README.md`, so after each merge the rest conflicted (#25, #27 twice), and the human approved PRs one by one while the agent waited. The cause: the **unit of tracking** (issue, board status, comments) was mixed up with the **unit of delivery** (branch, PR).

The rule, built into the command and the agents (it is a process rule, so it is not an ADR — agents must not have to look for it):

- A discussed epic is implemented in one pass: one branch `<type>/<epic>-<slug>`, one PR.
- A sub-issue of the epic = one group in `plan.md` = one commit `<type>(#<sub-issue>): ...`. Sub-issues stay on the board for tracking and get their own deviation comments.
- A large epic is split into sequential deliveries: the next branch starts only after the previous PR is merged. No parallel PRs within an epic. The split is decided at the plan stage.
- A task without sub-issues is a delivery of its own, as before.

| Agent | Role in a delivery |
|---|---|
| `feature-starter` | Branch named after the epic; In progress for the epic and the sub-issues given in `tasks`. |
| `task-context` | For an epic: its sub-issues (body, comments, state) as the content of the delivery. |
| `implementer` | Commit number = the group's sub-issue. Sub-issues of the same delivery are not "other tasks". |
| `validator` | One commit per group. |
| `pr-opener` | One PR; lists the sub-issues; `Refs #<epic>, #<sub-issue>, ...`; In review for all. |
| `feature-finisher` | Closes the sub-issues found in the PR commits; closes the epic only when all its sub-issues are closed. |

Agent descriptions moved from `README.md` to `docs/agents/<name>.md`: `README.md` keeps a short table, so parallel work does not conflict on it.

## 7. What goes where (for #9)

**Covered by existing agents** — the command calls them and drops its own steps:
- branch creation, base `release/*`, clean-tree check → `feature-starter`;
- closing the task, deleting the branch, updating `release/*` → `feature-finisher`;
- releasing into `main` → `releaser` (outside the command).

**Extend existing agents:**
- `feature-starter`: set In progress.

**New agents:** `task-context`, `implementer`, `validator`, `pr-opener`; `code-scout` only if `Explore` is not enough.

**Stays in the command (orchestrator):**
- base context (stage 2) and the relevance check (3a);
- code research prompt for `Explore` (4);
- approach agreement and posting it (5);
- writing `plan.md`, `context.md`, `validation.md` (6);
- briefings, reading reports, deviation comments, ADRs, problem handling (7–8);
- the list of human stops.

**Removed from the command:** roadmap, pre-flight "merged to main", `/clear`, `requirements.md`, `Key Decisions`, `Status`, "merge the branch".

## 8. Follow-up issues

Sub-issues of #8:

- #15 task-context agent
- #16 implementer agent
- #17 validator agent
- #18 pr-opener agent
- #19 code-scout agent (Backlog: only if `Explore` is not enough)
- #20 feature-starter: set In progress
- #21 templates: `specs/AGENT.md`, ADR, `plan.md`, `context.md`, `validation.md`
- #9 rewrite the command, after the issues above

Outside the epic: #13 (release checks moved into `releaser`, the separate validate command removed), #14 (`sdd-replan`).
