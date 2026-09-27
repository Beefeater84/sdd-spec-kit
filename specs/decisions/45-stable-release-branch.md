# Permanent staging branch, versions from tags

Status: accepted
Date: 2026-09-27
Issue: #45
Affects: #43; releaser, feature-starter, feature-finisher, pr-opener, validator, README INSTALL

## Context

The kit used a versioned `release/X.Y.Z` branch as the base and target of every feature branch and PR. Each release moved to a new `release/*` branch (`releaser` created the next one from `main` after tagging), which forced preprod to reconnect to a new branch after every release, and required agents to look up "the latest `release/*`" everywhere a base or target branch was needed.

## Decision

- One permanent branch `staging` replaces versioned `release/*`. Feature branches are created from `staging` in `origin` and merged back into it. Never branch from or target `main`/`master` — they hold released code only. Preprod deploys from `staging` and is never reconnected.
- Released versions are tags, not branches: `releaser` tags `vX.Y.Z` on the release PR's merge commit in `main` and publishes a GitHub Release with notes. The version is decided at release time: latest `v*` tag + bump (`major` / `minor` default / `patch`), or an explicit `X.Y.Z`; no tags yet → `0.1.0`. No `VERSION` file, no milestone. While a release is being prepared, the version is visible in the open PR `Release vX.Y.Z` from `staging` into `main`.
- Release notes are limited to PRs whose merge commit is in `origin/main..origin/staging` (i.e. only what actually shipped since the last release), and the release PR itself is merged with a merge commit so that range stays accurate.
- Kit install: the default install target is the latest tag; for unreleased code, the ref is `staging` (instead of `release/0.2.0`).
- Migration for a project that already has the kit (`README.md` INSTALL step 4):
  - `origin/staging` exists: use it, no migration needed.
  - Only `release/*` branches exist: propose to the human to create `staging` from the latest `release/*` (`git push origin origin/release/<x.y.z>:refs/heads/staging`), retarget open PRs onto it (`gh pr edit <n> --base staging`), reconnect preprod to `staging`, and later delete the old `release/*` branches. Only after a yes.
  - Neither exists: propose to create `staging` from the default branch and push it. Only after a yes.
  - Then create `chore/install-sdd-kit` from `origin/staging`.
- On a kit update, the Branching section of the target's `CLAUDE.md` is replaced with the new text (not skipped as "already there") whenever it still mentions `release/*`.

## Consequences

- No release branches are created or deleted anymore; the branch name (`staging`) is fixed in the kit. A project-level override of this name may come with a later task (#43).
- Preprod is connected to `staging` once and stays connected across releases.
- `releaser`, `feature-starter`, `feature-finisher`, `pr-opener` and `validator` all use `staging` as the fixed base/target instead of resolving "the latest `release/*`".
- Hotfixes committed directly into `main` (bypassing `staging`) are outside this scheme; the kit does not cover them.
- Existing projects on the kit must migrate once, as described above; until they do, their agents keep working against `release/*` per their installed version.
