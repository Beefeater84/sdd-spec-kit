# Context: #57 releaser: detect the default branch instead of hardcoding main

## Code map

- `.claude/agents/releaser.md` (440 lines) — steps: Hard rules L14-24; Input L26-29; Step 1 "Repository and staging" L31 (L34 `gh repo view --json owner,name ...` → `<owner> <repo>`; L40 `git fetch origin --prune`; STOP `staging not found in origin`); Step 2 Release PR L56; Step 3 Version L106; Step 4 Checks before the merge L144; Step 5 Project checks L171; Step 6 Release notes and PR L268; Step 7 Tag and GitHub Release L319; Step 8 "Update the local main" L362; Output L401-440.
- `main` occurrences in releaser.md: L3 frontmatter description; L8 intro; L19 never-touch rule (2nd sentence names local `main` in Step 8); L61 `gh pr list --head staging --base main --state open`; L81 same `--state merged`; L166 `git rev-list --count origin/main..origin/staging`; L169 STOP `staging has no commits that are not in main`; L270 prose; L274 `git rev-list origin/main..origin/staging`; L297 `gh pr create --base main`; L317 hint `merge PR #<pr> into main ... ancestor of main`; L362 heading; L366 `merge-base --is-ancestor "<merge_sha>" origin/main`; L369 STOP `... not in origin/main`; L373 `rev-parse --verify -q refs/heads/main`; L376 "no local `main`"; L378 "`<current>` is `main`"; L381 `merge --ff-only origin/main`; L387 `git fetch origin main:main`; L390 warning `local main has commits not in origin, not updated`; L393-394 `rev-parse refs/heads/main` / `origin/main`; L397, L419 `main_state` key.
- Cross-references by step number: L19, L24, L28, L75, L87, L94, L103, L104, L196, L230, L317, L321, L384, L420, L433-434 — keep step numbers unchanged.
- `docs/agents/releaser.md` — prose mirror: L5, L8, L9 say `main`; L27 `main_state: updated` in output example.
- `README.md` — L15 "a release PR into `main`"; L154 agents table "release PR `staging` → `main`". L34, L121 are never-push rules (keep). L46 "No tags: `main`" is about the kit's own install ref (keep). L38 already uses `defaultBranchRef`.

## Patterns to follow

- Record a value: "It prints `<owner> <repo>`. Use them below." (releaser L37); "`<latest>` is the output (empty if there are no tags)." (L118).
- Primary + fallback + STOP: `.claude/agents/feature-starter.md` L100-115: `- It prints ...: ...` / `- It prints nothing: STOP with \`reason: ...\` and \`hint: ...\``.
- Default-branch vocabulary: feature-starter L115 `hint: create staging in origin from the default branch`; pr-opener L153 "`staging`, which is not the default branch".

## Conventions

- Placeholders `<name>`, not shell variables: each Bash call starts a new shell. Use them inside quotes in commands, STOP reasons and hints.
- STOP format: STOP with `reason: <one line>` and optional `hint: <action>`, hints often end with ", then run releaser again".
- Output: exactly one block; "Keys and their order never change" (L438) — keep `main_state`, add no keys.
- Never edit accepted ADRs (`specs/decisions/45-*`).
- Rule-only mentions of `main`/`master` in other agents, commands and agent.md stay as they are.
- English, plain short sentences.

## Gaps found during implementation
