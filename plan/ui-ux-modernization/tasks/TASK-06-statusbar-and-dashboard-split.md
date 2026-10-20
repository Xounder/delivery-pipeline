# TASK-06 — `StatusBar` and the Mandatory `DashboardPage.tsx` Split

## Task Information

### ID

TASK-06

### Title

Fold all five inline status regions into a `StatusBar` and split the 567-line `DashboardPage.tsx` into prop-stable children

### Phase

Phase 4b — App shell (StatusBar host + page decomposition)

### Owner

senior-frontend

### Status

Pending

---

## Description

`DashboardPage.tsx` is 567 lines — already over the project's 400-line rule — and it is required by
Phases 4, 5 **and** 6. It also contains **five** stacked inline status regions. This task performs
the mandatory split and establishes the stable prop interfaces that TASK-07 and the Phase 6 lanes
route their changes through.

**This task's agent is the named single owner of `DashboardPage.tsx` for the entire delivery**
(`M9`, `P-8`).

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/components/StatusBar.tsx` | **create** |
| 2 | `apps/web/src/pages/DashboardPage.tsx` | modify — mandatory split |
| 3 | `apps/web/src/pages/dashboard/DashboardModals.tsx` | **create** |
| 4 | `apps/web/src/pages/dashboard/useCalendarNavigation.ts` | **create** |
| 5 | `apps/web/src/pages/dashboard/useDashboardModals.ts` | **create** |
| 6 | `apps/web/src/pages/dashboard/useExternalDrop.ts` | **create** |
| 7 | `apps/web/src/pages/dashboard/dashboardTypes.ts` | **create** |

### File count

7 (5 created, 2 modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/components/MainLayout.tsx`, `Header.tsx`, `Sidebar.tsx` — TASK-05 owns them
- `apps/web/src/styles/**` — TASK-02 / TASK-09
- `apps/web/src/features/**` — Phase 6 lanes own them; **this task must not restyle any feature file**
- `apps/web/src/contexts/**`, `apps/web/src/hooks/**` — unchanged
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)

---

## Constraints Satisfied

- `M9` — `DashboardPage.tsx` split into focused children; single owner for the whole delivery
- `M15` — **five** status regions, not four; all five fold into the `StatusBar`
- `M7` — no new visible copy that duplicates the task panel
- `P-8` — exactly one owner of `DashboardPage.tsx`; never two concurrent writers
- `P-9` — every resulting file under 400 lines
- `R-13`, `R-16`, `R-23` — page churn, control ownership, copy uniqueness

---

## Implementation Steps

1. **Create `apps/web/src/components/StatusBar.tsx`.** One region, `role="status"`,
   `aria-live="polite"`, rendered above the calendar. It renders **at most one** message at a time,
   by strict priority:

   | Priority | Condition | Exact copy | Presentation |
   | --- | --- | --- | --- |
   | 1 | `saveState.status === "error"` | `Error: {message}` | Mantine `Alert color="red"` |
   | 2 | `calendarQuery.isError` | `Failed to sync calendar events: {message}` | Mantine `Alert color="red"` |
   | 3 | `saveState.status === "success"` | `Schedule saved successfully!` | Mantine `Alert color="green"` |
   | 4 | `calendarQuery.isLoading` | `Loading calendar events...` | `Skeleton` + `Text` |
   | 5 | `isPreviewActive && failurePanelOpen` | *(allocation failures)* | `Paper` hosting `AllocationFailurePanel` |

   Each alert keeps its inline **`Retry`** `Button` with the existing behaviour
   (retry save on save-error; re-open the token modal on an auth-token-expired calendar error,
   otherwise refetch). Copy is **verbatim**; no wording may be reworded.
2. **Create `apps/web/src/pages/dashboard/dashboardTypes.ts`** — the shared, exported prop/state
   types (`DashboardStatusState`, `DashboardCalendarNav`, `DashboardModalState`,
   `DashboardConflictState`, …). These are the **stable interfaces** that TASK-07 and the Phase 6
   lanes consume instead of editing `DashboardPage.tsx`.
3. **Create `useCalendarNavigation.ts`** — owns `currentView`, `currentDate`, `handlePrev`,
   `handleNext`, `handleToday`. Move the existing logic verbatim, including the
   day/week/month-aware stepping. Export a typed `{ currentView, currentDate, onPrev, onNext,
   onToday, setCurrentView, setCurrentDate }` object.
4. **Create `useDashboardModals.ts`** — the single modal-state owner: `settingsOpen`,
   `selectedSlot`, `editingTask`, `editingBlock`, `creatingBlockPrefill`, `tokenModalOpen` plus
   their setters and the derived open/close handlers. No JSX.
5. **Create `useExternalDrop.ts`** — move `handleExternalDrop` verbatim (including the manual local
   `YYYY-MM-DDTHH:mm:00` string building and the `editing.canDropAt` guard). Depends on `settings`,
   `editing`, `setEvents`, `createBlock`. No JSX.
6. **Create `DashboardModals.tsx`** — render, in this order: `CreateActionModal` (when
   `selectedSlot`), `BlockModal` (create-prefill), `TaskModal` (edit), `BlockModal` (edit),
   `SettingsModal`, `DeleteEventDialog`, `ConflictDialog`, `TokenRetryModal`. All props are passed
   from `DashboardPage`; this component adds **no** state and **no** new copy.
7. **Rewrite `DashboardPage.tsx`** to compose: context/hook wiring → `useCalendarNavigation()`,
   `useDashboardModals()`, `useExternalDrop()` → `MainLayout` → action row
   (`CalculateWeekButton` / `SaveButton` / `PreviewToolbar` + `RefreshButton`) → `<StatusBar />` →
   calendar scroll container + `CalendarView` → `<DashboardModals />`.
   Delete the five inline banner blocks (currently lines ~362–474) and the modal JSX (lines
   ~505–564). Keep every remaining behaviour identical.
   **Target: ≤ 350 lines.** If the result exceeds 400 lines, extract further (for example the
   action-row cluster into `pages/dashboard/DashboardActionRow.tsx`) before finishing.
8. **Leave a header comment in `DashboardPage.tsx`** recording: this file has exactly one owner for
   the whole delivery; later phases extend `dashboardTypes.ts` / the extracted hooks instead of
   re-inlining logic here (`M9`, `P-8`).

---

## Acceptance Criteria

- [ ] `StatusBar.tsx` renders **at most one** region at a time, in the five-level priority order
- [ ] The five asserted strings are byte-identical: `Schedule saved successfully!`, `Error: {…}`, `Failed to sync calendar events: {…}`, `Loading calendar events...`, plus the allocation-failure panel
- [ ] Every `Retry` action keeps its current behaviour (save retry, token-modal re-open, refetch)
- [ ] `StatusBar` has `role="status"` and `aria-live="polite"`
- [ ] `DashboardPage.tsx` is **≤ 350 lines** (project limit is 400)
- [ ] Every new file under `apps/web/src/pages/dashboard/` is under 400 lines
- [ ] No behaviour changed: view switching, slot-click creation, task/block CRUD, preview generation, save, settings persistence, dev-mode auth all work as before
- [ ] No new visible copy containing `Preview`, `New Task`, `Edit Task`, `Tasks (`, `Create in this slot`, `Loading calendar events`, `Schedule saved successfully`, `Failed to sync calendar events`
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-05

### Dependency Notes

TASK-05 must land first and must leave `MainLayoutProps` unchanged, because this task keeps the
`MainLayout` call site intact. TASK-07 and the Phase 6 lanes depend on the extracted interfaces
produced here.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **Only four regions folded** — the code has **five**; the allocation-failure panel is the one most
  often missed (`M15`).
- **`Loading calendar events...` never disappears** — `isLoading` handling moved or inverted. The
  E2E suite waits for this string to become hidden (`TASK-01.spec.ts:138`).
- **Two statuses visible at once** — priority resolution missing. No spec depends on two being
  visible simultaneously, but mutual exclusivity removes the layout shift.
- **Stale closure after extraction** — `handleCreateTaskInline` and `handleExternalDrop` depend on
  `settings`, `createTask`, `createBlock`, `editing`, `setEvents`. Missing a dependency causes stale
  values, not a crash. Re-check each extracted hook's dependency array against the original.
- **`useEffect` on `saveState.status` moved or dropped** — `setHasUnsavedChanges(false)` after a
  successful save must survive, otherwise the unsaved-changes state is wrong.
- **`crypto.randomUUID()` in `handleExternalDrop`** — must stay; the save-flow filtering test depends
  on pending events carrying a local UUID.
- **Circular imports** — `DashboardModals.tsx` importing from `DashboardPage.tsx` would create a
  cycle. Types live in `dashboardTypes.ts`; keep `DashboardPage` as the only orchestrator.
- **The five status regions were previously stackable** — after folding they are mutually
  exclusive. If a Playwright failure suggests an assertion depended on two at once, fix the
  `StatusBar` priority, never the spec (`P-10`).
- **Concurrency** — another agent editing `DashboardPage.tsx` while this task runs violates `P-8`.
  Only this task's agent writes it.

---

## Testing

### Unit Tests

- [ ] None — no component test runner exists in `apps/web` and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, including `Loading calendar events...` becoming hidden and `Schedule saved successfully!` becoming visible (`TASK-01.spec.ts`)
- [ ] `build apps/web` typechecks every extracted module

### Manual Validation

- [ ] Trigger a save success, a save error, a calendar fetch error and a loading state — each shows exactly one region
- [ ] Allocation failures render the panel, and `Retry` still works on both error regions
- [ ] Create/edit task, create/edit block, slot-click create, save, settings, refresh — all unchanged

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] `build` cannot see spec drift (`M12`); `test apps/web` is the only guard
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `DashboardPage.tsx` split, single owner named, ≤ 350 lines
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/impact-analysis.md` — M9, M15](../planning/impact-analysis.md#material-corrections-m1m18)
- [`planning/dispatch-plan.md` — DashboardPage single-owner rule](../planning/dispatch-plan.md#dashboardpagetsx-single-owner-rule)
- [`design-docs/design-03-app-shell-layout.md` — StatusBar table](../design-docs/design-03-app-shell-layout.md)
- `apps/web/src/pages/DashboardPage.tsx`