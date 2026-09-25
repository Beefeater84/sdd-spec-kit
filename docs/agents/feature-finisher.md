# `feature-finisher`

Last step of the SDD multi-agent flow (`.claude/agents/feature-finisher.md`, model `haiku`, tools: `Bash`). Runs after the feature PR is merged. It does not run tests.

**Input:** task id and task branch, e.g. `Finish task 8, branch feat/8-create-orchestrator`. Optional `tasks`: the sub-issues delivered in the PR; by default they are read from the PR commits `<type>(#<n>): ...`.

The agent checks that the branch belongs to the task and its PR is merged; otherwise it changes nothing. Then it:
1. closes each delivered sub-issue with a comment `Done in #<pr>` (needed because `Closes #N` does not fire on merges into `release/*`);
2. closes the task itself, but an epic only when all its sub-issues are closed; otherwise the epic stays open for the next delivery (`issue: kept_open`). A task that is itself a sub-issue never closes its parent epic: the agent only reports the epic's progress;
3. sets the board Status to Done for everything it closed;
4. fast-forwards the local latest `release/*` from `origin` (switches to it only if you are on the task branch);
5. deletes the task branch locally and in `origin`, but only if its tip is exactly what was merged.

Re-running is safe: completed steps are skipped.

**Output:**

```
FEATURE_FINISHER_RESULT
status: ok
id: 1
pr: 3
branch: feat/1-feature-starter-agent
base: release/0.1.0
tasks: -
issue: already_closed
sub_issues: -
epic: -
board: already_done
release: release/0.1.0
release_state: up_to_date
local_branch: absent
remote_branch: deleted
warnings: -
```

`status: partial` means something was kept on purpose (see `warnings`). On failure: `status: error`, `id`, `reason`, `hint`.
