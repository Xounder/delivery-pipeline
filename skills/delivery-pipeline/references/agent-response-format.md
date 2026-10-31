# Standard Agent Response Format

All agents MUST return:

```yaml
summary:
  changes:
    - item            # File paths created or modified
  validations:
    - item            # Build, lint, typecheck results
concerns:
  - item              # Blockers, deviations, tool limitations
errors:
  - item              # Failures encountered (may be empty)
```

Fields:
- `summary.changes` — a YAML array listing every file created, modified, or deleted with a brief description; use `[]` when there are no changes
- `summary.validations` — a YAML array reporting each check's command or method, package/workdir, result, and execution status
- `concerns` — a YAML array recording issues such as shell restrictions, ambiguous specs, or tool limitations; use `[]` when empty
- `errors` — a YAML array recording failures such as build errors, runtime errors, or unexpected behavior; use `[]` when empty

Errors may be empty.
Concerns may be empty.

Scalar strings are not substitutes for the arrays above. An omitted field is not an empty array. Before returning, verify all four required fields have the documented shape. If only the report format is wrong, the orchestrator requests a corrected report without repeating implementation or validation.

## Shared Validation Contract

Implementation agents, the Tech Lead, and QA use this contract. The orchestrator delegates validation rather than executing these commands itself.

### Select the Project's Actual Commands

1. Identify the affected package directory and available scripts from the current project manifests and workspace configuration. A package may be at the repository root; never assume `apps/web` or `apps/api` exists.
2. Determine the project's package manager from its instructions, `packageManager` field, lockfile, and any recorded recovery decision. Resolve conflicting signals before installing or changing dependencies.
3. Inspect a runner's side effects before using it. If `run-package-command` would install dependencies with a different manager or modify a recovered environment, use the project's existing script through the appropriate shell command instead. Validation is not authorization to install dependencies, regenerate lockfiles, or switch managers.
4. Run the existing, relevant build, lint, typecheck, and test scripts in their package directories. If a script does not exist, report it as NOT APPLICABLE when appropriate; do not claim it passed or add a test framework merely to satisfy a generic instruction. A missing acceptance-critical check remains a gap for the orchestrator to resolve.
5. Respect user constraints for every command and tool, including GUI/headless browsers, CDP automation, servers, and deployment. A prohibited check is NOT VERIFIED, not implicitly waived.

### Record Evidence, Not Just a Green Summary

- Use **PASS**, **FAIL**, **NOT VERIFIED**, or **NOT APPLICABLE** for each check. Include the exact command/method, workdir, exit code when available, and relevant findings or artifact paths.
- A nonzero lint exit is FAIL even when all findings belong to an accepted baseline. Identify existing versus new findings by file and rule, not just the total count; an unchanged count can hide a regression.
- Existing failures may be accepted only within the documented acceptance scope. Report the failures and that decision explicitly; never describe a baselined failure as a passing command.
- If earlier evidence is reused, label it **not rerun** and identify its source and the reviewed revision or unchanged files. Reuse evidence only when relevant source, configuration, dependencies, and artifacts still match. Do not present an earlier agent's command as a newly executed check.
- Do not rerun a successful build solely because another workflow section repeats it. Rerun when relevant changes, failures, or unresolved concerns invalidate the evidence.
- Distinguish source inspection, build validation, runtime verification, and deployed/external validation. A build or typecheck does not prove visual equivalence, animation behavior, accessibility-tree output, or production HTTP responses.

### Exercise Service Boundaries

- For service changes involving pagination, partial failures, cache replacement, or HTTP validators such as ETags, include relevant regression checks against the actual production service. Passing pure-helper tests or finding expected source text alone does not establish correct orchestration across these boundaries.
- Exercise the affected failure and race scenarios: incomplete page sets must not be committed as complete snapshots, and a validator must not refresh cache records it did not validate. Assert observable results, requests, and cache writes rather than duplicating production logic in a test-only implementation.
- Where browser execution is unavailable or restricted, use existing tooling for scoped Node execution with mocked network/storage boundaries when feasible. Restore mocks after each check, avoid dependency or infrastructure changes solely for this purpose, and disclose any remaining acceptance-critical gaps.
- Report pure-helper assertions, static checks, and mocked execution of production services as distinct evidence. Mocked HTTP/storage checks do not certify real browser behavior, CORS, cross-tab storage timing, or deployed responses; preserve the actual execution and approval scope.

### Bound the Verdict

QA returns an explicit verdict and its approval scope. List unexecuted acceptance checks separately. A waiver records the user's decision and its exact scope; it does not change a check's execution result to PASS. If mandatory criteria remain unmet and no applicable revision was approved, return Corrections Needed or report the blocker rather than silently narrowing acceptance.

## Example

Illustrative npm project at the repository root; select commands from the actual project rather than copying these names unconditionally.

```yaml
summary:
  changes:
    - "Created src/components/Button.tsx"
    - "Modified src/main.tsx (added provider)"
  validations:
    - "PASS: npm run typecheck; workdir=.; exit=0; zero TypeScript errors"
    - "PASS: npm run build; workdir=.; exit=0; 142 modules"
    - "NOT VERIFIED: browser interaction checks; execution prohibited by the user"
concerns:
  - "This report provides source/build evidence, not runtime or deployment certification"
errors: []
```
