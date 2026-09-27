# Context: #45 Stable release branch name with tracked release versions

## Code map

- `.claude/agents/releaser.md:3` frontmatter description — rewrite fully (release/X.Y.Z, next release/* branch).
- `releaser.md:8` role; `:10` two-run model (keep); `:16-24` hard rules — `:19` "only branch you create is next release/*" → creates no branch, never deletes/changes `staging`, `main`, `master`; `:20` re-run list mentions next branch; `:24` "switch only to `<release>` in Step 5".
- `releaser.md:26-29` Input: `version` (e.g. `0.1.0` / `release/0.1.0`, default latest origin/release/*), `next` (major/minor/patch/X.Y.Z, default minor) → `version` optional explicit X.Y.Z (strip leading `v`), `bump` major/minor/patch default minor.
- `releaser.md:31-69` Step 1 repo + fetch --prune + pick latest origin/release/* (`:46-50` for-each-ref | sed | grep -Ex X.Y.Z | sort -V | tail -1), `:53` STOP none, `:58` X.Y.Z check, `:63` `<release>`/`<tag>`, `:66` origin/<release> exists.
- `releaser.md:71-100` Step 2 next version: `:84-89` `IFS=. read -r MA MI PA` + case bump; `:95` higher-than via `sort -V`; `:100` `<next_release>`.
- `releaser.md:102-113` Step 3 release PR lookup: `gh pr list --head "<release>" --base main --state all --json number,state,url,mergeCommit` → MERGED → Step 7; OPEN → Steps 4,5,6 "PR exists"; none/CLOSED → "no PR". With `--head staging` there are many PRs over time: must pick the OPEN one (version from title `Release vX.Y.Z`), else the most recent MERGED one whose tag does not exist yet → run 2, else none → new version. Add `title` to json.
- `releaser.md:115-140` Step 4 checks before merge: `:120-121` peeled tag lookup `git ls-remote --tags origin "refs/tags/<tag>*" | awk`; `:129` open PRs into `<release>`; `:137` `rev-list --count origin/main..origin/<release>` STOP on 0.
- `releaser.md:142-237` Step 5 project checks: clean tree, `git switch "<release>"` + warning, `merge --ff-only origin/<release>`, `## Checks` awk, forbidden filter; `:235` hint "fix the failure on <release>".
- `releaser.md:239-287` Step 6 notes + PR: `:245` `gh pr list --base "<release>" --state merged --limit 500 --json number,title,headRefName,mergedAt` + jq grouping (`:246-257`, heading `## Release v<version>`); must additionally filter by `mergeCommit.oid` ∈ `git rev-list origin/main..origin/staging`. `:267` `gh pr create --base main --head "<release>" --title "Release <tag>"`; `:275-285` body refresh; `:287` run-1 result.
- `releaser.md:289-330` Step 7 tag + GitHub Release; `:332-367` Step 8 ff local main (`:354` mentions `<release>` after Step 5); `:369-406` Step 9 next release branch — delete.
- `releaser.md:408-449` Output: finished keys status, version, release, pr, pr_url, pr_state, tag, tag_state, gh_release, release_url, main_state, next_release, next_branch_state, checks, warnings, hint → drop next_release, next_branch_state. Error keys: status, version, reason, checks, hint. `:447` "Keys and their order never change."
- `.claude/agents/feature-starter.md:3,8,16,17,21` description/role/hard rules about release/*; `:91-135` Step 4 base: clean tree (`:93-99`, keep), pick highest origin release/* (`:101-113`), highest local release/* + 3 STOPs (`:115-135`); `:137-159` Step 5 ls-remote + `rev-list --left-right --count "origin/<base>...<base>"` ahead/diverged (keep, works with staging); `:173` `git switch --no-track -c "<branch>" "origin/<base>"`; result keys `:230-241` (unchanged).
- `.claude/agents/feature-finisher.md:3,17,21` description/hard rules; `:185-245` Step 6 update local release branch: `:187-198` pick latest origin/release/* + fallback (`release: -`, warning `no release/* branch in origin`, go to Step 7); `:200-245` ff logic (keep, `<release>` = `staging`); `:255` Step 7 refers to Step 6; result `:306-307` `release`, `release_state` (keep keys).
- `.claude/agents/pr-opener.md:3,17` description/hard rule; `:29` input `base` e.g. `release/0.2.0`; `:45-49` base check `grep -Ex 'release/v?[0-9]+(\.[0-9]+)*'` → exact `staging`; `:153` "`Closes` does not fire on merges into `release/*`" → into `staging` (not the default branch).
- `.claude/agents/validator.md:3` description "base release/* branch"; `:20` input `base` "the release branch it targets".
- `docs/agents/releaser.md:5,8,9,11` prose; `:15-33` example (`release: release/0.1.0`, `next_release`, `next_branch_state`); `:35` variants; `:39-50` error example hint on `release/0.2.0`.
- `docs/agents/feature-starter.md:9,11,22`; `docs/agents/feature-finisher.md:8,11,24,31`; `docs/agents/pr-opener.md:5,7,19`.
- `agent.md:24-25` Branching rules.
- `README.md:11` ref examples; `:23-27` INSTALL step 2 version (latest tag, else `main`); `:37-41` step 4 branch (release/* lookup, propose `release/0.1.0`); `:60-67` update mode; `:97-102` Branching block copied into target CLAUDE.md (step 7 "add each section only if not there yet"); `:107` step 8 `--base release/<x.y.z>`; `:123,127,131,133` agents section; `:153,161` flow.
- `.claude/commands/create-sdd-feature.md:10-11` unit of delivery / Branches rule.
- `.claude/commands/sdd-init-legacy.md:136` "PR into the project's working branch (e.g. the latest `release/*`)".

## Patterns to follow

- Agent steps — like `.claude/agents/releaser.md`: `## Step N. Title`, bash block, "It prints X: …", `STOP with \`reason: …\` and \`hint: …\``, `SKIP with \`key: value\``, "Record", "add warning", "Go to Step N". Placeholders `<angle>`, defined as "`<x>` = `…`".
- docs/agents pages — like `docs/agents/releaser.md`: one-line header, **Input**, behavior, "Re-running is safe", **Output** example block mirroring result keys.
- ADR — `.claude/templates/sdd/adr.md`: title, `Status`, `Date`, `Issue`, `Affects`, Context / Decision / Consequences.

## Conventions

- English in agents and docs; short sentences written for haiku; every command written out; every branch ends in go on / SKIP / STOP.
- Frontmatter `description` must match behavior (Claude Code routes by it).
- Renumber steps and fix every "Step N" reference when steps are removed.
- Result keys: change the agent's Output and its docs/agents example together.
- The Branching text is identical in `agent.md`, README INSTALL step 7 block, and `create-sdd-feature.md` Rules (in meaning).
- `Refs`, not `Closes`: `Closes` fires only on merges into the default branch; `staging` is not it.
- Release PR is merged with a merge commit, so `staging` stays an ancestor of `main`.
- Do not change `specs/features/*` (except this folder) and `docs/analysis/8-create-sdd-feature.md`.

## Gaps found during implementation
