# TASK-13 — Post-Merge Reconciliation and Consolidated Validation

## Task Information

### ID

TASK-13

### Title

Run the full gate on the merged Phase 6 tree, reconcile cross-lane copy/duplicate findings, and route any fix back to its owning lane

### Phase

Phase 6 exit gate — validation-only

### Owner

senior-frontend

### Status

Pending

---

## Description

The three Phase 6 lanes are file-disjoint, so the merge is expected to be conflict-free. This task
verifies that expectation and runs the full gate **once more on the consolidated tree** before Phase 7
starts (`R-25`).

It is a **validation task**. It does not implement new features and it does not edit another lane's
files. Its only permitted source edits are trivial, non-behavioural reconciliation items (see step 4)
that no lane owner is actively working on; anything else is routed back to the owning lane.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `.opencode/plan/ui-ux-modernization/tasks/TASK-13-merge-validation-report.md` | **create** — validation record |

### File count

1 created (a report), 0 source files modified

### Allowed Conditional Edits

Only after explicit confirmation, and only when the owning lane is closed:

- `apps/web/src/styles/calendar/*.css` — duplicate rule de-duplication, **no** new declarations
- `apps/web/src/main.tsx` — import-order tidy only (**OVERLAP-2**; all phases now closed)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)
- `pnpm-lock.yaml` — regenerated only via `pnpm install` (**OVERLAP-4**)
- `packages/ui/dist/**` — generated only (**OVERLAP-10**)
- `apps/web/src/pages/DashboardPage.tsx` — **OVERLAP-1**, TASK-06's agent
- Any feature file belonging to TASK-10 / TASK-11 / TASK-12
- `apps/api/**`, `packages/shared/**`, `packages/backend/**`

---

## Constraints Satisfied

- `R-25` — consolidated gate after merge
- `M14` — fresh dev server required before any trusted `test apps/web` result
- `M12` — build is not evidence that E2E specs still typecheck
- `P-5`, `P-6`, `P-8` — no spec or config edits
- `M7`, `M8`, `B1`, `B2` — cross-lane copy and DOM-contract re-verification

---

## Implementation Steps

1. **Verify the merge was clean.** Diff the consolidated tree against each lane's reported file list.
   Confirm the three lane file sets are still disjoint and that no lane touched a file outside its
   list. A merge conflict here is a **lane-boundary violation**, not a Git problem — record the
   offending file and which lane claimed it.
2. **Cross-lane reconciliation checks** (these are the failures that only appear after merging):
   | Check | Failure mode | Owner to fix |
   | --- | --- | --- |
   | Exactly one `<aside>` | sidebar rendered twice | TASK-05 owner |
   | Exactly one `Tasks (N)` panel | `TaskPanel` mounted twice | TASK-10 |
   | Exactly one visible `Preview` | stale `ConflictDialog` copy mounted | TASK-11 |
   | No duplicate modal copy | a `keepMounted` modal left over | TASK-10 / TASK-11 |
   | Exactly one `Today` control | toolbar + header duplication | TASK-07 |
   | No `!important` in calendar CSS | re-introduced while patching | TASK-09 |
   | No `.brk-calendar` (without `-google`) selector | design path copied | TASK-09 |
   | No `businessHours` prop on FullCalendar | added by a lane | TASK-08 |
   | `DashboardPage.tsx` ≤ 350 lines | a lane re-inlined logic | TASK-06 |
   | No source file > 400 lines | a rewrite grew past the cap | owning lane |
3. **Run the full gate on the consolidated tree, in order:**
   1. `run-package-command build packages/ui` → expect **PASS**
   2. `run-package-command build apps/web` → expect **PASS**
   3. `run-package-command test apps/web` → expect **9/9 PASS**
   Before step 3, ensure no stale dev server is bound to the dev port (`M14`). Record the run
   conditions in the report — a run on a reused server is **invalid**.
4. **Route, do not absorb.** For every failure, identify the owning lane and record it in
   `TASK-13-merge-validation-report.md` with the failing assertion, the file, and the lane. Only
   apply the two allowed conditional edits above (duplicate CSS rules, import-order tidy) and only
   with confirmation; every other fix goes back to its lane.
5. **Write the report** at
   `.opencode/plan/ui-ux-modernization/tasks/TASK-13-merge-validation-report.md`, containing:
   - the per-lane file lists as merged;
   - each reconciliation check and its result;
   - the three validation results with observed bundle sizes;
   - an explicit list of findings routed back to lanes, or "none".
6. **Do not start Phase 7** until the gate is green and all cross-lane findings are resolved.

---

## Acceptance Criteria

- [ ] Lane file sets are confirmed disjoint in the merged tree; no file is owned by two lanes
- [ ] All ten reconciliation checks executed and recorded
- [ ] `build packages/ui` → **PASS** on the consolidated tree
- [ ] `build apps/web` → **PASS** on the consolidated tree
- [ ] `test apps/web` → **9/9 PASS** on a **fresh** dev server, with run conditions documented
- [ ] Bundle output recorded (JS raw/gzip, CSS raw/gzip) for the Phase 7 baseline comparison
- [ ] Report written to `TASK-13-merge-validation-report.md`
- [ ] No feature, shell, calendar, context or hook file was modified by this task
- [ ] No E2E spec, config or manifest was modified
- [ ] Every unresolved finding is attributed to a named owning lane

---

## Dependencies

### Required Tasks

- TASK-10
- TASK-11
- TASK-12

### Dependency Notes

All three Phase 6 lanes must be merged before this task runs. Phase 7 (TASK-14, TASK-15) depends on
this task's green result.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **Green build, red E2E** — expected risk class. `apps/web/tsconfig.json` excludes `e2e`, so the
  build cannot see selector drift introduced by `B1`, `B2`, `M5`, `M7` or `M8` (`M12`). The Playwright
  run is the only real gate.
- **False green from a stale dev server** — `reuseExistingServer: true` makes Playwright silently
  reuse a server started in an earlier phase, exercising an old bundle. Both false passes and false
  failures are possible; with `fullyParallel: false` and `retries: 1`, one contaminated run can
  invalidate a phase sign-off (`M14`). Report the port state, not just the result.
- **False failure that was already fixed** — same root cause, opposite direction. Re-verify against a
  fresh bundle before routing a fix to a lane.
- **Lane claims a file outside its list** — a lane-boundary violation. Record it; do not silently
  accept the merged state.
- **`packages/ui/dist` is stale** — a targeted `apps/web` build does not rebuild the package, so the
  web app can typecheck against stale declarations. `build packages/ui` first, always (`R-05`,
  **OVERLAP-10**).
- **Two `Preview` labels from different lanes** — ownership is TASK-11's. Report; TASK-12 must not
  "fix" it by editing a feature file.
- **File count grew past the inventory** — a lane created new files instead of editing existing ones.
  Record the extra files; that itself is a lane-boundary finding.
- **A lane reported a failure it did not route** — treat as an open defect and block Phase 7.
- **Partial merge state** — only two lanes merged. Stop; do not run the consolidated gate.

---

## Testing

### Unit Tests

- [ ] None — validation-only task

### Integration Tests

- [ ] The full gate run on the consolidated tree, with all three results recorded
- [ ] Each reconciliation check re-verified manually or via a targeted locator check

### Manual Validation

- [ ] Cross-check that each lane's reported changes are actually present in the merged tree
- [ ] Confirm no duplicate DOM copies of any asserted string exist at a desktop viewport

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] This task is where cross-lane selector drift surfaces; treat any Playwright failure as a real defect, not a flake, until disproven
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Merge verified disjoint
- [ ] Ten reconciliation checks executed and recorded
- [ ] Full gate green on the consolidated tree with a fresh dev server
- [ ] Report written
- [ ] No routed findings remain open
- [ ] Validation completed

---

## References

- [`planning/dispatch-plan.md` — Phase 6 Gate and lane merge rule](../planning/dispatch-plan.md#phase-6-gate--feature-migration)
- [`planning/risk-register.md` — R-25](../planning/risk-register.md#critical-and-high-risks)
- [`planning/impact-analysis.md` — OVERLAP flags](../planning/impact-analysis.md#file-overlap-flags)