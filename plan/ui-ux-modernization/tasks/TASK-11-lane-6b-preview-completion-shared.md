# TASK-11 — Phase 6 Lane 6-B: Preview, Completion and Shared Feature Migration

## Task Information

### ID

TASK-11

### Title

Migrate `features/preview/**`, `features/completion/**` and `features/shared/**` to Mantine; keep `Preview` single-owner and `ConflictDialog` unmounted

### Phase

Phase 6 lane 6-B — Preview + Completion + Shared (≈865 lines, 10 files)

### Owner

senior-frontend

### Status

Pending

---

## Description

Ten files migrate to Mantine. This lane carries the two most copy-sensitive constraints in the
delivery:

- `M8` / `R-02` / `R-23` — `ConflictDialog` contains "preview" copy in several places, including a
  discard action. It **must not remain in the DOM when closed**, or `text=Preview` resolves to more
  than one element. `PreviewToolbar.tsx` is the **sole owner** of the visible `Preview` label.
- `B2` — `CreateActionModal.tsx` keeps `<input id="task-name">`, `<select id="task-duration">` and
  both submit-button locators resolvable.

`TokenRetryModal.tsx` and `RefreshButton.tsx` live under `features/completion/`, **not** `components/`
(`M18`).

**Parallel dispatch:** runs concurrently with TASK-10 and TASK-12. Zero shared files.

---

## Files This Task May Create / Modify

| # | File | Lines now | Action |
| --- | --- | --- | --- |
| 1 | `apps/web/src/features/preview/PreviewToolbar.tsx` | 75 | modify — Mantine toolbar; **sole owner of `Preview`** |
| 2 | `apps/web/src/features/preview/SaveButton.tsx` | 50 | modify — copy `Save to Calendar` unchanged |
| 3 | `apps/web/src/features/preview/CalculateWeekButton.tsx` | 49 | modify — copy `Calculate Week` unchanged |
| 4 | `apps/web/src/features/preview/AllocationFailurePanel.tsx` | 64 | modify — must not introduce `Preview` |
| 5 | `apps/web/src/features/preview/DeleteEventDialog.tsx` | 70 | modify — Mantine modal |
| 6 | `apps/web/src/features/completion/ConflictDialog.tsx` | 86 | modify — `keepMounted={false}`; unmount when closed |
| 7 | `apps/web/src/features/completion/CompletionAction.tsx` | 42 | modify — Mantine action row (`M10`) |
| 8 | `apps/web/src/features/completion/TokenRetryModal.tsx` | 115 | modify — consumes `@brkroutnxdle/ui` `Modal` (`M18`) |
| 9 | `apps/web/src/features/completion/RefreshButton.tsx` | 40 | modify — Mantine button (`M18`) |
| 10 | `apps/web/src/features/shared/CreateActionModal.tsx` | 274 | modify — **native controls preserved** |

### File count

10 modified, 0 created

### Do Not Touch — forbidden for lane 6-B

- `apps/web/src/features/tasks/**`, `features/blocks/**` (TASK-10)
- `apps/web/src/components/**` — **including** `components/calendar/**` (TASK-05 / TASK-07 / TASK-08)
- `apps/web/src/pages/**` — `DashboardPage.tsx` is **OVERLAP-1**, owned by TASK-06's agent (`P-8`)
- `apps/web/src/styles/**` (TASK-02 / TASK-09 / TASK-14)
- `apps/web/src/main.tsx` (**OVERLAP-2**), `apps/web/src/App.tsx` (TASK-12, **OVERLAP-8**)
- `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `packages/ui/src/**`, `packages/ui/package.json`, `packages/ui/dist/**` — **OVERLAP-9 is read-only
  for this lane**; **OVERLAP-10** is generated and never hand-edited
- `apps/web/e2e/**`, `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-5`, `P-6`)
- `apps/api/**`, `packages/shared/**`, `packages/backend/**`, `packages/domain/**`, `packages/calendar/**`

---

## Constraints Satisfied

- `B2` — `CreateActionModal` native `#task-name` input, native `#task-duration` select, both submit locators
- `M8` — `ConflictDialog` unmounts when closed
- `M10` — `CompletionAction.tsx` is in scope
- `M18` — correct `features/completion/` paths for `TokenRetryModal` / `RefreshButton`
- `M7`, `P-5`, `P-6`, `P-8`, `P-9`, `R-01`, `R-02`, `R-23`

---

## Implementation Steps

1. **`PreviewToolbar.tsx`** — Mantine `Group` / `Paper` / `Segmented`-free layout. It is the
   **sole legitimate owner of the visible `Preview` label**, and `TASK-02.spec.ts` asserts
   `text=Preview` resolves to **exactly one** element and is visible. Keep that label as visible text,
   mounted only when the toolbar is rendered. Do not add a second `Preview` string anywhere in this
   lane.
2. **`ConflictDialog.tsx` — `M8` is the task's critical requirement.** The component currently
   returns `null` when closed, so its "preview" copy never reaches the DOM. After migration it must
   still be absent:
   - keep the early `if (!isOpen) return null;` guard (or an equivalent that prevents render);
   - if a Mantine `Modal` is used, keep the default `keepMounted={false}` — do **not** set
     `keepMounted`;
   - never render the dialog markup from a parent that always mounts it.
   A `keepMounted` modal here leaves several `Preview`-containing strings in the DOM and breaks the
   exactly-one assertion for the rest of the suite.
3. **`CreateActionModal.tsx` (274 lines) — `B2`:**
   - `<input id="task-name">` stays a native input (`fill("#task-name")`);
   - `<select id="task-duration">` stays a native select (`selectOption("#task-duration")`);
   - `button[type="submit"]:has-text("Create Task")` and `button[type="submit"]:has-text("Create")`
     must both still resolve. The two create buttons alternate by mode; **only one may be visible at
     a time**, exactly as today. If both are rendered simultaneously with `display: none`, Playwright's
     visibility filter usually tolerates it, but prefer real conditional rendering.
   - Mantine `TextInput` / `Select` / `NumberInput` are forbidden for these two fields.
   - Keep the file under 400 lines (`P-9`).
4. **`SaveButton.tsx` / `CalculateWeekButton.tsx`** — copy is unchanged and asserted by the suite:
   `Save to Calendar`, `Calculate Week`. Keep them as real `<button>` elements with those exact
   visible labels. Loading / disabled states must keep working.
5. **`AllocationFailurePanel.tsx`** — Mantine panel. It sits inside `StatusBar`'s priority-5 region
   (TASK-06). It **must not** introduce the string `Preview` (`R-23`).
6. **`CompletionAction.tsx`, `DeleteEventDialog.tsx`, `TokenRetryModal.tsx`, `RefreshButton.tsx`** —
   Mantine migration; preserve existing copy and handlers. `TokenRetryModal` consumes
   `@brkroutnxdle/ui`'s `Modal`, whose public prop is `isOpen` (not Mantine's `opened`). Verify the
   Phase 3 wrapper contract before wiring (`OVERLAP-9` is read-only here).
7. **Scan all ten files** for leftover `style={{` inline styles; all styling must move to Mantine
   props or token classes.
8. **No new files** in this lane.

---

## Acceptance Criteria

- [ ] `ConflictDialog` renders nothing when `isOpen` is false — verified by the `text=Preview` exactly-one assertion
- [ ] No Mantine modal in this lane sets `keepMounted`
- [ ] Exactly **one** element matches `text=Preview`, and it is visible
- [ ] `<input id="task-name">` and `<select id="task-duration">` still exist in `CreateActionModal.tsx`
- [ ] `button[type="submit"]:has-text("Create Task")` and `button[type="submit"]:has-text("Create")` both resolve; only one visible at a time
- [ ] `Save to Calendar` and `Calculate Week` are unchanged visible button labels
- [ ] `AllocationFailurePanel.tsx` contains no `Preview` string
- [ ] `TokenRetryModal` / `RefreshButton` remain under `features/completion/`; no duplicate components created under `components/` (`M18`)
- [ ] All ten files remain under 400 lines
- [ ] `packages/ui/src/index.ts` untouched
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-09

### Dependency Notes

May only start after Phase 5 is merged and `packages/ui` has been built. Runs concurrently with
TASK-10 and TASK-12; no intra-phase dependency.

### Runs In Parallel With

- TASK-10 (lane 6-A) — file-disjoint
- TASK-12 (lane 6-C) — file-disjoint

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** on a fresh dev server (`M14`)
4. No lint step (`P-4`)

**Merge rule:** after TASK-10, TASK-11 and TASK-12 all complete, the full gate runs once more on the
consolidated tree (TASK-13).

---

## Edge Cases

- **`text=Preview` resolves to two elements** — `ConflictDialog`'s copy is still mounted
  (`keepMounted`, or the early `return null` guard was dropped while migrating to Mantine). This is
  the single most likely failure in this lane (`M8`).
- **`text=Preview` resolves to zero** — the visible label moved out of `PreviewToolbar` into an
  `aria-label` or a tooltip. It must be visible text.
- **`selectOption("#task-duration")` times out** — Mantine `Select` used instead of a native
  `<select>`, or the `id` moved (`B2`).
- **`fill("#task-name")` succeeds but nothing is created** — the input stopped being controlled, or
  the submit path changed.
- **Both create buttons visible at once** — conditional rendering replaced with CSS hiding; the
  spec's `has-text("Create")` locator may then resolve to the wrong control.
- **`Save to Calendar` / `Calculate Week` label drift** — a trailing space, an ellipsis or an icon
  that replaced the text breaks a `has-text` matcher. Keep the exact strings.
- **Delete-confirmation dialog stays in the DOM** — same unmount rule as `ConflictDialog`; the
  suite asserts other unique strings and stale duplicates cause strict-mode violations.
- **`TokenRetryModal` opens on the wrong prop** — the `@brkroutnxdle/ui` wrapper takes `isOpen`;
  passing Mantine's `opened` type-errors or silently breaks the open path (`R-05`).
- **Duplicate components created under `components/`** — copying the design's wrong paths from
  `M18`. Files stay in `features/completion/`.
- **Stale dev server** — a false 9/9 against another lane's bundle (`M14`). Report run conditions.
- **Cross-lane copy conflict** — if another lane introduces a second `Preview` label, this lane must
  not edit their files; report it for the merge review in TASK-13.

---

## Testing

### Unit Tests

- [ ] None — no component test runner exists in `apps/web` and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, in particular `TASK-02.spec.ts` `text=Preview` uniqueness and visibility, and `TASK-01.spec.ts` preview → save → conflict flows

### Manual Validation

- [ ] Generate a preview; confirm one visible `Preview` label
- [ ] Save to Calendar; confirm `Schedule saved successfully!` still appears
- [ ] Trigger a conflict; confirm the dialog mounts on open and disappears entirely on close
- [ ] Allocation failures render the panel with a working retry

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Do not add time-based waits where a locator assertion is deterministic
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] Lane forbidden lists respected
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/dispatch-plan.md` — Lane 6-B](../planning/dispatch-plan.md#lane-6-b--preview--completion--shared-865-lines-10-files)
- [`planning/file-impact.md` — Preview / Completion / Shared tables](../planning/file-impact.md#preview-feature)
- [`planning/impact-analysis.md` — B2, M8, M10, M18, R-01, R-02, R-23](../planning/impact-analysis.md#material-corrections-m1m18)
- [`design-docs/design-05-component-inventory.md`](../design-docs/design-05-component-inventory.md)
- [`design-docs/design-06-migration-plan.md`](../design-docs/design-06-migration-plan.md)