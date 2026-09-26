# Context: #13 Validate release/* in releaser, remove sdd-validate command

## Code map

- `.claude/agents/releaser.md` — frontmatter `description` (l.3: "Does not merge, run tests, or touch feature branches"); intro l.8 ("no merges, no tests, no builds"); Hard rules l.14-21; Step 1 repo/fetch/`<release>`,`<tag>`; Step 2 next version; Step 3 find release PR (MERGED → Step 6; OPEN → Step 4 then Step 5 "PR exists"; none → Step 4 then Step 5 "no PR"); Step 4 "Checks before the merge" l.112-137 (tag absent, no open PRs into `<release>`, commits ahead of main); Step 5 notes + PR; Step 6 tag + GitHub Release (run 2); Step 7 update local main (l.254 "do not switch the user's branch", `git fetch origin main:main`); Step 8 next release branch; Output l.308-345.
- `RELEASER_RESULT` finished keys, in order: status, version, release, pr, pr_url, pr_state, tag, tag_state, gh_release, release_url, main_state, next_release, next_branch_state, warnings, hint. Error block: status: error, version, reason, hint.
- `.claude/agents/validator.md` — Step 2 "Project checks" l.35-39: sources `<folder>/validation.md` → "Tools", then `specs/tech-stack.md`, then manifest (`package.json` scripts, `pyproject.toml`, `Makefile`); line format `<name> — pass|fail|skipped — <command>; <short detail>`; missing kind → `skipped — not configured`. Hard rules l.12: no `--fix`/`--write`, no `*:prod`, no migrations. Output has list keys with `  - ` items; empty list is `key: -`.
- `.claude/agents/feature-finisher.md` — l.215-230: dirty-tree check before `git switch`, fast-forward update of local release branch.
- `docs/agents/releaser.md` — l.3 "It never merges and does not run tests."; l.8 run-1 description with stop conditions; example output block; last line explains statuses.
- `docs/agents/validator.md` — example output block with fail lines (pattern for a failed-check example).
- `README.md` — l.19 AGENTS row for releaser `| [\`releaser\`](docs/agents/releaser.md) | Manual: release PR ... | haiku |`; SKILLS section `/sdd-validate` block l.56-64 with `---` separators at l.54 and l.66 (remove the block and one separator).
- `docs/analysis/8-create-sdd-feature.md` — l.173 "Reusable by `sdd-validate` (#13)."; l.292 "Outside the epic: #13 (`sdd-validate`), #14 (`sdd-replan`)."
- `specs/tech-stack.md` — empty template sections; "## Code Style" l.19, "## Smoke Tests" l.22 with HTML comment placeholders.
- `.claude/commands/sdd-init-legacy.md` — ~l.71 its tech-stack template with "## Smoke Tests".
- `.claude/commands/sdd-validate.md` — to delete.

## Patterns to follow

- New releaser step — like releaser Step 4: bash block, then "If it prints X, STOP with `reason: ...` and `hint: ...`". Steps titled `## Step N. <Title>`; renumber following steps and fix every "Go to Step N" / "Step 4, then Step 5" reference.
- `checks` lines and list key — like validator Step 2 and its Output.
- docs page — like `docs/agents/validator.md`: path/model/tools paragraph, `**Input:**`, behavior, `**Output:**` example, statuses line.
- `## Checks` section in tech-stack — same style as other sections: heading + HTML comment placeholder with an example (one command per line, in run order, e.g. install, typecheck, lint, test).

## Conventions

- Agents are English, terse; STOP `reason:` lowercase, no period; `hint:` is an imperative for the human; "You never run a `hint` yourself."
- Output section first sentence stays verbatim: "Your final message is exactly one block and nothing else: ...". "Keys and their order never change."
- Re-running must stay safe. Never force-push, reset or rewrite history; branch updates are fast-forward only.
- Release notes (Step 5) text and generation stay unchanged.
- No project-level lint/tests exist in this repo (Markdown only).

## Gaps found during implementation
