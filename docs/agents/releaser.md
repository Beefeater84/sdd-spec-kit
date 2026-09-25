# `releaser`

Release step of the SDD multi-agent flow (`.claude/agents/releaser.md`, model `haiku`, tools: `Bash`). Run it manually when the release is ready. It never merges and does not run tests.

**Input:** optional version and optional next version, e.g. `Release 0.1.0` or `Release, next patch`. Without a version the highest `origin/release/*` is used. The next version is `major`, `minor` (default) or `patch` bump, or an explicit `X.Y.Z`.

The agent works in two runs; it finds out which one from the state of the release PR:
1. **Before the merge:** it stops if the tag already exists, if PRs into `release/X.Y.Z` are still open, or if there is nothing new over `main`. Otherwise it builds release notes from the PRs merged into the release branch, grouped by type (`feat`, `fix`, `docs`, `refactor`, `chore`, other), and opens the PR `release/X.Y.Z` → `main` with them (or refreshes the body of the open PR). Ends with `status: awaiting_merge`. The human merges the PR.
2. **After the merge:** it creates the tag `vX.Y.Z` on the merge commit and a GitHub Release with the PR body as notes, fast-forwards the local `main`, and creates and pushes `release/<next>` from `origin/main`.

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
warnings: -
hint: -
```

After run 1 `status: awaiting_merge` and `hint` says which PR to merge. `status: partial` means something was kept on purpose (see `warnings`). On failure: `status: error`, `version`, `reason`, `hint`.
