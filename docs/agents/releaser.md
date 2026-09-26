# `releaser`

Release step of the SDD multi-agent flow (`.claude/agents/releaser.md`, model `haiku`, tools: `Bash`). Run it manually when the release is ready. It never merges and never fixes anything: in run 1 it runs the project checks and reports failures.

**Input:** optional version and optional next version, e.g. `Release 0.1.0` or `Release, next patch`. Without a version the highest `origin/release/*` is used. The next version is `major`, `minor` (default) or `patch` bump, or an explicit `X.Y.Z`.

The agent works in two runs; it finds out which one from the state of the release PR:
1. **Before the merge:** it stops if the tag already exists, if PRs into `release/X.Y.Z` are still open, or if there is nothing new over `main`. Then it runs the project checks on the up-to-date release branch in the current working tree: it stops on uncommitted changes, switches to `release/X.Y.Z` if needed (with a warning) and fast-forwards it from `origin`. The commands come from `## Checks` in `specs/tech-stack.md` (one per line, in run order), else from the `package.json` scripts `typecheck`, `lint`, `test` (with `npm ci` or the pnpm/yarn equivalent first if `node_modules` is missing); with none found it warns and goes on. `*:prod` scripts, `--fix`, `--write` and migrations are never run. Any failed check stops the run and the release PR is not opened: the calling session fixes the failure with the human, then runs the releaser again. If all checks pass, it builds release notes from the PRs merged into the release branch, grouped by type (`feat`, `fix`, `docs`, `refactor`, `chore`, other), and opens the PR `release/X.Y.Z` → `main` with them (or refreshes the body of the open PR). Ends with `status: awaiting_merge`. The human merges the PR.
2. **After the merge** (no checks): it creates the tag `vX.Y.Z` on the merge commit and a GitHub Release with the PR body as notes, fast-forwards the local `main`, and creates and pushes `release/<next>` from `origin/main`.

Re-running is safe: an existing PR, tag, GitHub Release or next release branch is skipped. It never moves or overwrites a tag.

**Output:**

```
RELEASER_RESULT
status: ok
version: 0.1.0
release: release/0.1.0
pr: 12
pr_url: https://github.com/Beefeater84/sdd-spec-kit/pull/12
pr_state: merged
tag: v0.1.0
tag_state: created
gh_release: created
release_url: https://github.com/Beefeater84/sdd-spec-kit/releases/tag/v0.1.0
main_state: updated
next_release: release/0.2.0
next_branch_state: created
checks: -
warnings: -
hint: -
```

After run 1 `status: awaiting_merge`, `checks` lists every check and `hint` says which PR to merge. `status: partial` means something was kept on purpose (see `warnings`). On failure: `status: error`, `version`, `reason`, `checks` (`-` if the checks did not run), `hint`.

A run 1 stopped by a failed check:

```
RELEASER_RESULT
status: error
version: 0.2.0
reason: check check 3 failed: npm test
checks:
  - check 1 — pass — npm ci; ok
  - check 2 — pass — npm run lint; ok
  - check 3 — fail — npm test; test/store.test.js "rejects past due dates"; last lines: 1 failed, 8 passed | Tests: 9 | npm ERR! Test failed
  - check 4 — skipped — npm run build; not run after a failure
hint: fix the failure on release/0.2.0 with the calling session, then run releaser again
```

Each `checks` line is `<name> — pass|fail|skipped — <command>; <detail>`. Commands from `## Checks` are named `check <n>`; `package.json` scripts by their name. A kind the project does not have is `<name> — skipped — not configured`.
