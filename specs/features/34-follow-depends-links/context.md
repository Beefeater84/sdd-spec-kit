# Context: #34 task-context: follow Depends on links

## Code map

- `.claude/agents/task-context.md` — the agent (model haiku, tools Bash, Read, Grep). Prescriptive steps with exact bash blocks, run by haiku "as written".
  - L3 frontmatter `description`: lists what is collected — add dependencies.
  - L12-18 Hard rules: read-only; only `specs/decisions/` files are read; verbatim only for issue body, approach comment, open sub-issue bodies, everything else one line per item.
  - L32-46 Step 2: `gh issue view <id> --json number,title,state,url,createdAt,labels,body,comments` — add `blockedBy` (returns `{"nodes":[{number,title,state,...}],"totalCount":N}`; verified with gh 2.101.0). Comment classification: `## Подход к реализации` = approach, `## Отклонение от подхода` = deviation, other = `<date> <author> — <one line>`.
  - L48-75 Step 3: sub-issues (GraphQL `subIssues`), parent epic (GraphQL `parent`). L73: no epic → `epic: -`, `epic_goal: -`, `siblings: -`, "Go to Step 4 if there are sub-issues, else to Step 5" — this routing must now reach the Dependencies step (and its PR lookup) too.
  - L77-105 Step 4: `gh pr list --state all --limit 200 --json number,state,headRefName,files`; PR belongs to task `<n>` when head matches `^(feat|fix|docs|refactor|chore)/<n>-`; `pr` = MERGED `#pr`, else OPEN `#pr open`, else `-`; fallback: closing comment `Done in #<pr>`. L96 approach/deviation comment query with `startswith`. L101-105 `in_flight` for OPEN PRs touching `specs/decisions/`.
  - L107-122 Step 5: ADR included when `Affects` mentions `#<id>`, epic, sibling or sub-issue number (`reason: affects`) or `Date` ≥ `created`.
  - L124-176 Output: success block keys in fixed order (… `sub_issues`, `siblings`, `adrs`, `in_flight`); L167 rule: empty list is `-` on the key line, keys and order never change. Failure block L171-176.
- `.claude/commands/create-sdd-feature.md` L49-59 — stage 3; L53 is the relevance-check sentence listing `adrs`, `in_flight`, siblings' `notes`, deviation and later comments.
- `docs/agents/task-context.md` — L7 prose list of what is collected; L11-35 example TASK_CONTEXT_RESULT (issue #16); L37 failure line.
- `docs/analysis/8-create-sdd-feature.md` — L31 flow line for task-context; L61 "3a. Relevance check" what is compared; L115-126 task-context spec, output keys L120-125; L249 table row.
- `README.md` — L14 agents table row for `task-context`; L41 flow step 3.

## Decided in the approach

- Sources: `blockedBy` → `source: blocked by`; body → `source: body`; both → `source: blocked by, body`.
- Body phrases (case-insensitive): depends on, dependent on, blocked by, requires, зависит от, блокируется; followed by `#N`, a list `#N, #M` (also `и`/`and`), or an issue URL of this repo. Not "после #N". One ready-made command (`grep -oiE` + extraction of numbers), not a prose description.
- Exclude the task itself, its epic, its sub-issues and siblings; dedupe.
- Per dependency: `#<n> <STATE> <title> — source: … — pr: … — notes: <approach/deviation one line or -> — decisions: …`. `decisions`: every comment starting with a `## ` heading other than approach/deviation, one line each (heading — gist), plus one line for all other comments. For an epic dependency add only the count of closed/open sub-issues. No recursion.
- Dependency numbers join the `Affects` rule (Step 5); their OPEN PRs touching `specs/decisions/` join `in_flight`.
- `depends_on` goes after `siblings`, before `adrs`; `-` when none. Keep each dependency within ~10 lines.

## Patterns to follow

- New step — like Step 4 in `.claude/agents/task-context.md`: exact bash block, then bullet rules with explicit `-` fallbacks.
- Nested list item in the result — like `sub_issues` items (two-space item, four-space sub-keys).
- Doc — like the existing `docs/agents/task-context.md`: prose paragraph + realistic shortened example.

## Conventions

- Agent, command and doc files are in English. Comment headings matched by the agent are fixed Russian strings.
- Result block: exactly one block; `key: |` with two-space indentation for multi-line; list items `  - …`, fields separated by ` — ` (em dash).
- No typecheck/lint/tests in the repo (`specs/tech-stack.md` Checks is empty); checks: -.
- Commits `feat(#34): …` / `docs(#34): …` with trailer `Co-Authored-By: Claude Opus 5.5 <noreply@anthropic.com>`.

## Gaps found during implementation
