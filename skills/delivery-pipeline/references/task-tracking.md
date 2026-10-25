# Task Tracking

Implementation tasks must be tracked in pipeline state.

## Transition and Consistency Rules

- Distinguish implementation in progress, partial implementation, implementation awaiting QA, and a task approved by QA. File creation is evidence of output, not approval.
- Update every representation of a task together, including the top-level task list and the owning step's list. Align the current step/agent and resume instructions with the next actual action.
- Record each dispatch outcome at its actual timestamp, including cancellation, quota blocks, partial reports, and format-only retries. Preserve the completed portions and identify the remaining scope.
- Keep prior failures and their resolution in history. The current status and next action must not imply a resolved defect is still blocking.
- A user-authorized skipped task is recorded as skipped, never completed or approved. Identify which downstream criteria were explicitly revised and which remain mandatory.
- Approval notes must distinguish source/build evidence from runtime and deployed/external checks. Accepted lint baselines retain their failed command result; waived or prohibited checks are not passing checks.
- Before completion, reconcile task/step statuses, active-agent and pause/dispatch flags, and all resume metadata. Record outstanding follow-ups without leaving stale instructions to repeat earlier work.

Example:

```yaml
tasks:

  - id: TASK-01
    title: Create User Endpoint
    owner: senior-backend
    status: completed
    notes: null
    updated_at: 2026-06-11T20:00:00Z

  - id: TASK-02
    title: Create User Screen
    owner: senior-frontend
    status: pending
    notes: Depends on TASK-01
    updated_at: null
```
