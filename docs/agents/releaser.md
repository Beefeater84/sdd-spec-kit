# `releaser`

Release step of the SDD multi-agent flow (`.claude/agents/releaser.md`, model `haiku`, tools: `Bash`). Run it manually when the release is ready. It never merges and never fixes anything: in run 1 it runs the project checks and reports failures.

**Input:** optional version and optional bump, e.g. `Release 0.2.0` or `Release, bump patch`. The agent always releases the permanent branch `staging` into `main`. The version is taken, in this order: from the title `Release vX.Y.Z` of the open PR `staging` → `main` (a different explicit version stops the run); from the last merged release PR whose tag or GitHub Release is still missing (this is run 2); from the explicit version; else from the latest `vX.Y.Z` tag in `origin` raised by the bump `major`, `minor` (default) or `patch`. With no tags the version is `0.1.0` (with a warning). A new version must be higher than the latest tag.

The agent works in two runs; it finds out which one from the state of the release PR:
1. **Before the merge:** it stops if the tag already exists, if PRs into `staging` are still open, or if `staging` has nothing new over `main`. Then it runs the project checks on the up-to-date `staging` in the current working tree: it stops on uncommitted changes, switches to `staging` if needed (with a warning) and fast-forwards it from `origin`. The commands come from `## Checks` in `specs/tech-stack.md` (one per line, in run order), else from the `package.json` scripts `typecheck`, `lint`, `test` (with `npm ci` or the pnpm/yarn equivalent first if `node_modules` is missing); with none found it warns and goes on. `*:prod` scripts, `--fix`, `--write` and migrations are never run. Any failed check stops the run and the release PR is not opened: the calling session fixes the failure with the human, then runs the releaser again. If all checks pass, it builds release notes from the PRs merged into `staging` whose merge commit is not in `main` yet (`staging` keeps the PRs of all past releases), grouped by type (`feat`, `fix`, `docs`, `refactor`, `chore`, other), and opens the PR `staging` → `main` titled `Release vX.Y.Z` with them (or refreshes the body of the open PR). Ends with `status: awaiting_merge`. The human merges the PR with a merge commit, so `staging` stays an ancestor of `main`.
2. **After the merge** (no checks): it creates the tag `vX.Y.Z` on the merge commit and a GitHub Release with the PR body as notes, and fast-forwards the local `main`.

Re-running is safe: an existing PR, tag or GitHub Release is skipped. It never moves or overwrites a tag and never creates branches.

**Output:**

```
RELEASER_RESULT
status: ok
version: 0.1.0
release: staging
pr: 12
pr_url: https://github.com/Beefeater84/sdd-spec-kit/pull/12
pr_state: merged
tag: v0.1.0
tag_state: created
gh_release: created
release_url: https://github.com/Beefeater84/sdd-spec-kit/releases/tag/v0.1.0
main_state: updated
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
reason: check 3 failed: npm test
checks:
  - check 1 — pass — npm ci; ok
  - check 2 — pass — npm run lint; ok
  - check 3 — fail — npm test; test/store.test.js "rejects past due dates"; last lines: 1 failed, 8 passed | Tests: 9 | npm ERR! Test failed
  - check 4 — skipped — npm run build; not run after a failure
hint: fix the failure on staging with the calling session, then run releaser again
```

Each `checks` line is `<name> — pass|fail|skipped — <command>; <detail>`. Commands from `## Checks` are named `check <n>`; `package.json` scripts by their name. A kind the project does not have is `<name> — skipped — not configured`.
