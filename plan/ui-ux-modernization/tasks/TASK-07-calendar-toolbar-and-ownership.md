# TASK-07 — Calendar Chrome: `CalendarToolbar`, Theme Map and Header Ownership Move

## Task Information

### ID

TASK-07

### Title

Add `CalendarToolbar` + `calendarTheme`, move date navigation and the view switcher out of `Header`, render the toolbar from `DashboardPage`

### Phase

Phase 5a — Calendar chrome and control ownership

### Owner

senior-frontend

### Status

Pending

---

## Description

`headerToolbar={false}` means the calendar renders as a bare grid with no title, no navigation and
no context — the user's core complaint. This task adds a real React toolbar above the grid and fixes
control ownership: the date step and the view switcher move **out of** the `Header` and **into** the
`CalendarToolbar`, so exactly one owner exists for each control.

The grid itself is not touched in this task; TASK-08 replaces the event DOM and TASK-09 re-skins the
stylesheet.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/components/calendar/CalendarToolbar.tsx` | **create** |
| 2 | `apps/web/src/components/calendar/calendarTheme.ts` | **create** |
| 3 | `apps/web/src/components/Header.tsx` | modify — remove date-step + view switcher |
| 4 | `apps/web/src/pages/DashboardPage.tsx` | modify — render `<CalendarToolbar />` above `CalendarView` |

### File count

4 (2 created, 2 modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/components/CalendarView.tsx` — TASK-08 owns it; **do not** pass any new FullCalendar prop from this task
- `apps/web/src/styles/**` — TASK-09 owns `CalendarStyles.css` and the partials
- `apps/web/src/components/MainLayout.tsx`, `Sidebar.tsx`, `StatusBar.tsx`
- `apps/web/src/features/**`, `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `apps/web/src/theme/**` — TASK-03 owns it
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)

---

## Constraints Satisfied

- `B1` — the view switcher in its **new** home MUST still be real `<button>` elements
- `B3` — **no** `businessHours` prop may be introduced on the FullCalendar instance
- `M7` — the toolbar must not add any copy that duplicates the task panel
- `M15` — the Header / CalendarToolbar ownership split is applied
- `M1` — the toolbar's own wrapper class MUST use the real calendar scope class
- `P-9` — every file under 400 lines
- `R-16`, `R-23` — no duplicated/orphaned controls, no copy collisions

---

## Implementation Steps

1. **Create `apps/web/src/components/calendar/calendarTheme.ts`.** A single exported map from
   `--brk-*` semantic tokens to FullCalendar `--fc-*` custom properties, used by TASK-09:
   `--fc-border-color`, `--fc-page-bg-color`, `--fc-neutral-bg-color`, `--fc-neutral-text-color`,
   `--fc-today-bg-color`, `--fc-now-indicator-color`, `--fc-non-business-color`,
   `--fc-highlight-color`, `--fc-event-bg-color`, `--fc-event-border-color`,
   `--fc-event-text-color`, `--fc-event-resizer-thickness`, `--fc-small-font-size`,
   `--fc-daygrid-event-dot-width`, `--fc-list-event-hover-bg-color`. Export it as a documented
   constant so the CSS author and the TS author agree on one list; it is a documentation/typing
   artefact — the actual declarations live in the stylesheet (TASK-09). Also export the business-hours
   label helper used by the toolbar badge (e.g. `"09:00 – 18:00"`), derived from
   `settings.workStartHour` / `workEndHour`. Keep it free of JSX and under 200 lines.
2. **Create `apps/web/src/components/calendar/CalendarToolbar.tsx`.** Renders, left to right:
   - `ActionIcon` with `aria-label="Previous period"` calling `onPrev`;
   - `Button` labelled exactly **`Today`** calling `onToday`;
   - `ActionIcon` with `aria-label="Next period"` calling `onNext`;
   - the date title as `Text component="h2"` — the format logic (`toLocaleDateString("en-US", …)`,
     with the day segment only in `ViewType.Day`) must stay identical to what `Header` used;
   - `Badge` showing the business-hours label (no asserted string);
   - `Badge color="yellow"` reading `Unsaved changes`, rendered **only** when `hasUnsavedChanges`;
   - the view switcher — **`B1` mandatory**: three `Button` elements inside a flex container with
     segmented styling, one per view, each rendering **visible text exactly** `Day`, `Week`, `Month`,
     with `aria-pressed` bound to `currentView`. **`SegmentedControl` MUST NOT be used** — it emits
     `<input type="radio">` + `<label>` and no `<button>`, which fails
     `button:has-text("Day")` / `button:has-text("Week")`.
   **Copy rule (strict).** The toolbar may introduce only: `Today`, `Previous period`,
   `Next period`, `Calendar view`, `Unsaved changes`, and the business-hours label. It **must not**
   introduce any string containing `Preview`, `New Task`, `Edit Task`, `Tasks (`,
   `Create in this slot`, `Loading calendar events`, `Schedule saved successfully`, or
   `Failed to sync calendar events`.
   Props: `{ currentDate, currentView, onPrev, onNext, onToday, onViewChange, hasUnsavedChanges,
   settings }` — all supplied by `DashboardPage`.
3. **Modify `Header.tsx`** — apply the `M15` ownership table:
   | Control | Owner |
   | --- | --- |
   | Brand, date-range title, `Today` | `AppShell.Header` |
   | `‹` / `›` date step, `Day`/`Week`/`Month` | `CalendarToolbar` |

   **Remove** the `‹`/`›` `ActionIcon`s and the three view-switcher buttons from `Header`.
   **Keep** the colour-scheme toggle, the `Settings` button, and the `MainLayout` prop surface
   intact. After the edit, `HeaderProps` may narrow — but `MainLayoutProps` must keep the same
   fields so `DashboardPage` keeps compiling; `MainLayout` (TASK-05's file) **must not be edited in
   this task**, so keep `HeaderProps` source-compatible with what `MainLayout` passes. If `Header`
   no longer needs `onPrev`/`onNext`/`onViewChange`, leave the props declared and unused rather than
   changing `MainLayout` — TASK-05's contract wins.
   Exactly **one** `Today` control may exist in the document; it lives in the toolbar.
4. **Modify `DashboardPage.tsx`** — render `<CalendarToolbar … />` inside `AppShell.Main`'s column,
   above the `CalendarView`, directly under the action row and the `StatusBar`. Wire it from the
   extracted `useCalendarNavigation()` plus `settings` and the existing
   `hasUnsavedChanges` state. **Do not** re-inline logic that TASK-06 extracted (`M9`, `P-8`): if a
   new hook is genuinely needed, add it to `pages/dashboard/` rather than growing this file. Keep
   `DashboardPage.tsx` at or below its TASK-06 size.
5. **Do not** add `businessHours`, `dayMaxEvents`, `slotEventOverlap`, or any other new prop to the
   `<FullCalendar>` instance from this task (`B3`).

---

## Acceptance Criteria

- [ ] `CalendarToolbar.tsx` and `calendarTheme.ts` exist under `apps/web/src/components/calendar/`
- [ ] `Day`, `Week`, `Month` are real `<button>` elements with exactly those visible labels
- [ ] The string `SegmentedControl` does not appear in any created or modified file
- [ ] `Header.tsx` renders **no** date-step button and **no** view switcher
- [ ] Exactly **one** `Today` control exists in the document
- [ ] `‹` / `›` expose the accessible names `Previous period` / `Next period`
- [ ] `Unsaved changes` badge appears only when `hasUnsavedChanges` is true
- [ ] No `businessHours` prop was added to `<FullCalendar>`
- [ ] No new visible copy matching any forbidden string
- [ ] `DashboardPage.tsx` is **not larger** than after TASK-06 and stays ≤ 350 lines
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-06

### Dependency Notes

TASK-06 must land first: the toolbar is rendered from `DashboardPage`, and TASK-06 establishes the
navigation hook and the modal/status composition this task must not disturb.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **View switcher disappears from the page** — it was removed from `Header` but the toolbar was not
  mounted (or renders only when a preview is active). It must be unconditional.
- **`button:has-text("Week")` matches nothing** — `SegmentedControl` used, or the buttons render
  icons without visible text labels.
- **Duplicate `Today` buttons** — `Header` kept its `Today` while the toolbar added one. Exactly one
  may exist.
- **Two date-step pairs** — `Header` kept `‹`/`›`. Remove them; `M15` gives the toolbar sole
  ownership.
- **Toolbar doubles the toolbar-row height** — `headerToolbar={false}` stays; do not re-enable
  FullCalendar's own header.
- **`hasUnsavedChanges` badge flickers** — it must read the existing state, not a copy held in the
  toolbar.
- **Business-hours label with `settings.workEndHour === 24`** — pad to two digits and clamp; do not
  render `24:00` → `2400`. Reuse the existing `padStart(2, "0")` logic from `DashboardPage`.
- **`MainLayout` no longer needs removed props** — the task forbids editing it. Keep the declared
  props and ignore unused ones; a later task may tidy them.
- **Calendar height chain** — the toolbar adds a row above the grid inside a fixed-height column. If
  the page starts scrolling instead of the calendar scrolling, the flex child needs
  `min-height: 0` — verify, do not "fix" it by shrinking the toolbar.
- **Regression on `Tasks (N)` uniqueness** — the toolbar must not introduce any text containing
  `Tasks (`.

---

## Testing

### Unit Tests

- [ ] None — no component test runner exists and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, in particular `TASK-02.spec.ts:105,108` which click `button:has-text("Day")` then `button:has-text("Week")`

### Manual Validation

- [ ] Toolbar shows date title, `Today`, step buttons, view switcher, hours badge
- [ ] Switching Day → Week → Month updates the grid without a full-page reload
- [ ] Only one `Today` exists; no `‹`/`›` in the header

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Selector drift here is invisible to `build` (`M12`); `test apps/web` is the only guard
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/impact-analysis.md` — M15, B1, B3, M1](../planning/impact-analysis.md#material-corrections-m1m18)
- [`planning/dispatch-plan.md` — Phase 5 Gate](../planning/dispatch-plan.md#phase-5-gate--calendar)
- [`design-docs/design-04-calendar-ux.md` — CalendarToolbar + business hours](../design-docs/design-04-calendar-ux.md)
- `apps/web/src/components/Header.tsx`, `apps/web/src/pages/DashboardPage.tsx`