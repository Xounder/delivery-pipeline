# Failure Recovery

After every agent execution or dispatch outcome, including cancellation and infrastructure failure:

1. Save pipeline state.
2. Save summary.
3. Save errors.
4. Save concerns.
5. Save history entry.

Examples of what to record:
- **errors**: build failures, runtime crashes, test failures, compilation errors
- **concerns**: shell restrictions, ambiguous specifications, tool limitations, environment issues

### Resume Metadata Accuracy

After updating pipeline state, verify that `resume.last_completed_agent` and `resume.last_completed_step` match the actual completed work. These fields must be updated when a step finishes — never leave them pointing to a pending step.

Also reconcile duplicated task lists, step status, current step/agent, pause and dispatch flags, and resume/next-action text. An ended dispatch must not leave an agent marked active. Retain earlier errors in history with their resolution; distinguish them from current blockers so recovery does not repeat already corrected work.

### Evidence-Based Recovery

File existence, a partial implementation, or an agent's success claim alone does not establish completion. Inspect the actual output/diff, task acceptance evidence, and validation results before choosing the next action:

- Missing or incomplete implementation: record partial work and re-dispatch its owner with the remaining scope.
- Implementation present but validation or the required response missing: request those missing results without repeating completed implementation.
- Implementation and its validation complete but QA pending: record awaiting QA rather than task completion.
- Task approved by the required QA under the recorded acceptance scope: mark complete and resume the next unmet dependency.

The orchestrator inspects evidence and manages state; implementation agents and QA execute build/test validation. A required gate remains blocking unless satisfied or explicitly revised by the user. Record skips and waivers as decisions, not as passing checks.

### Cancellation and Infrastructure Blocks

Record the outcome, observed error, actual timestamp, affected task, partial artifacts, and retry count before proceeding. A user cancellation is not permission to immediately relaunch the same work. A quota block or missing capability may require external resolution; persist that blocker rather than repeatedly issuing an unchanged dispatch. For retryable failures, follow the named-agent retry limit in the delivery skill; never silently change agent type or model to bypass it.

An authorized retry receives the original task plus the current partial-result inventory, user restrictions, package-manager/recovery constraints, and remaining acceptance checks. Inspect artifacts before editing so retries preserve existing work.

### Git Diff Tool Limitations

The `get-git-diff` tool may fail with `undefined..undefined` because its default arguments are not applied at runtime. Use `bash` with explicit refs as a fallback:

```text
git diff HEAD~1 -- apps/web/
```

Specify both `baseRef` and `headRef` explicitly when calling the tool.

---

If execution stops unexpectedly:

```text
delivery-pipeline:resume
```

must continue from the latest persisted state.

Completed work must never be repeated.

### Abandoned Pipeline Detection

If a pipeline step remains `running` for more than 24 hours without state updates, it should be treated as potentially abandoned. Before resuming:

1. Verify the step's agent actually produced output (check for target files, git diff, build artifacts).
2. If output exists, classify it using Evidence-Based Recovery above. Do not mark a step complete merely because files were created.
3. If no output exists, re-dispatch only when the prior outcome and current constraints permit it, passing the original task and updated context.
4. If retryable re-dispatch fails repeatedly (3 attempts), mark the step as `failed`, update pipeline status, and report the error. Preserve externally blocked or user-cancelled work accurately rather than manufacturing a success or automatic retry.

This prevents stale `running` states from blocking pipeline recovery.
