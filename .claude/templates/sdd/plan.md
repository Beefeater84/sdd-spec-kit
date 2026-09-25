# Plan: #<id> <title>

Issue: #<id> · Approach: <link to the "## Подход к реализации" comment>
Delivery: #<n>, #<m> <!-- sub-issues in this delivery; "-" for a single task -->
Later: #<k> <!-- sub-issues left for the next delivery of the epic, or "-" -->

<!-- For the human. Short. Says WHAT and WHERE, never HOW:
     no code, no signatures, no step-by-step instructions — the implementer decides that.
     One group = one commit `<type>(#<task>): ...`.
     Epic: one group per sub-issue in the delivery, in the order they build on each other.
     Single task: group by code area, not by layer; every group's task is #<id>.
     One implementer may continue through groups in the same code area. -->

## Group 1: <name>

- Task: #<n> <!-- the sub-issue this group delivers, or #<id> for a single task -->
- Goal: <one sentence>
- Files: `<path>` (create), `<path>` (change)
- Reuse: `<path>` — <what>
- Done when: <observable result, e.g. a test passes>
- Complexity: normal <!-- normal | complex (complex runs on a stronger model) -->

## Group 2: <name>

- Task:
- Goal:
- Files:
- Reuse:
- Done when:
- Complexity: normal
