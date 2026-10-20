# TASK-08 — `eventContent` Renderer and Class-Name Emission

## Task Information

### ID

TASK-08

### Title

Replace the `eventDidMount` DOM-mutation delete button with an `eventContent` React renderer; emit `brk-completed` / `is-past` and project-owned event classes

### Phase

Phase 5b — Calendar event rendering

### Owner

senior-frontend

### Status

Pending

---

## Description

`CalendarView.tsx` builds a delete button with `document.createElement` and thirteen inline style
assignments inside `eventDidMount`, then removes it in `eventWillUnmount`. The button is unreachable
by React, unstyled by tokens and untestable. This task moves it into a real React `eventContent`
renderer and fixes the class emission — `brk-completed` and `is-past` are computed but never emitted,
so completed events look identical to active ones.

**This is the highest-risk single edit in the delivery** (`R-03`).

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/components/CalendarView.tsx` | modify — `eventContent`, class emission, delete `eventDidMount`/`eventWillUnmount` |
| 2 | `apps/web/src/components/calendar/EventContent.tsx` | **create** |

### File count

2 (1 created, 1 modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/styles/**` — TASK-09 re-skins the stylesheet; **this task must not edit any CSS**
- `apps/web/src/components/Header.tsx`, `MainLayout.tsx`, `Sidebar.tsx`, `StatusBar.tsx`
- `apps/web/src/components/calendar/CalendarToolbar.tsx`, `calendarTheme.ts` — TASK-07 owns them
- `apps/web/src/pages/DashboardPage.tsx` — **OVERLAP-1**, single owner is TASK-06's agent (`P-8`)
- `apps/web/src/features/**`, `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)

---

## Constraints Satisfied

- `B3` — **no** `businessHours` prop may be added
- `M5` — `eventContent` must keep the title inside `.fc-event`, emit project-owned classes, cover
  `dayGridMonth` **and** both `timeGrid` views, render all event kinds, stop delete-click propagation
- `M16` — no competing sticky-header CSS (not applicable here, but no layout CSS may be added)
- `M7` — no new visible copy
- `P-9` — both files under 400 lines; `CalendarView.tsx` must **shrink**
- `R-03`, `R-21` — event DOM preservation, portal/z-index safety

---

## Implementation Steps

1. **Create `apps/web/src/components/calendar/EventContent.tsx`.** A pure presentational component
   receiving `{ arg: EventContentArg; showDelete: boolean; onDelete: (id, title) => void }`.
   Markup contract — this exact class set is what TASK-09's stylesheet targets, so do not rename it:
   ```tsx
   <div className="brk-event-content">
     <span className="brk-event-time">{arg.timeText}</span>
     <span className="brk-event-title">{event.title}</span>
     {p.isCompleted && <IconCheck className="brk-event-check" aria-hidden />}
     {showDelete && (
       <Tooltip label="Delete event" withArrow>
         <ActionIcon
           className="brk-event-delete-btn"
           size="xs" variant="subtle"
           aria-label={`Delete ${event.title}`}
           onClick={(e) => { e.stopPropagation(); onDelete(event.id, event.title); }}
         >
           <IconX size={12} />
         </ActionIcon>
       </Tooltip>
     )}
   </div>
   ```
   - **`event.title` MUST be rendered inside the `.fc-event` element.** `eventContent` replaces
     FullCalendar's children, so `.fc-event` still wraps this markup — the title text must appear
     inside it. This is the single most important line of the task.
   - The delete button MUST call `e.stopPropagation()` **and** preserve the old double-click guard
     (`e.detail > 1 → return`), so a double-click on the button does not fire `eventClick`.
   - The `.brk-event-delete-btn` class name is **preserved** even though the element is now
     React-owned, so existing CSS selectors keep matching.
2. **Modify `CalendarView.tsx`:**
   - Delete `handleEventDidMount` and `handleEventWillUnmount` in full, and remove both props from
     the `<FullCalendar>` JSX.
   - Add `eventContent={(arg) => <EventContent … />}` and compute `showDelete` in the callback:
     `Boolean(p.isGenerated || p.isBlock || p.isPreview) && !isPast`, where `isPast` falls back to a
     runtime end-time comparison (`endStr ?? startStr + 30 min`) for recurring blocks whose
     `isPast` is not precomputed — mirroring the current `eventDidMount` logic exactly.
   - Do **not** add `businessHours` (`B3`).
   - **Extend `classNames`** so the stylesheet can actually apply its existing state rules:
     - task events → `["brk-generated" | "brk-external", e.isCompleted && "brk-completed", isPast && "is-past"]`
     - preview events → `["brk-preview", isPast && "is-past"]`
     - block events → `["brk-block", isPast && "is-past"]`
     Filter falsy entries out of the arrays.
   - Leave the wrapper `<div className="brk-calendar-google" …>` **unchanged** — TASK-09 targets it
     (`M1`).
   - Keep every existing FullCalendar prop, the `VIEW_MAP` / `REVERSE_VIEW_MAP` maps, the
     `isSyncingRef` guard, `handleDatesSet`, `handleEventAllow`, `handleExternalDrop` and all
     drag/drop wiring exactly as they are.
   - Month view (`dayGridMonth`) has no time text; `arg.timeText` is simply empty there — render the
     span conditionally on `arg.view.type.startsWith("timeGrid")` so chips are not left with an empty
     block.
3. **Reduce the file below its current 373 lines** by moving the two `useMemo` event mappers and the
   new render callback into small helpers inside `components/calendar/` if that helps. `CalendarView.tsx`
   must finish under 400 lines (`P-9`).

---

## Acceptance Criteria

- [ ] `eventDidMount` and `eventWillUnmount` are **gone** from `CalendarView.tsx`
- [ ] `eventContent` is supplied and renders `.brk-event-content`, `.brk-event-time`, `.brk-event-title`, optionally `.brk-event-check`, optionally `.brk-event-delete-btn`
- [ ] The event title text is inside the `.fc-event` element for all five event kinds
- [ ] Exactly one `.fc-event` element exists per calendar event (no extra elements added)
- [ ] `brk-completed` is emitted for completed events and `is-past` for past events
- [ ] Existing class names `brk-generated`, `brk-external`, `brk-preview`, `brk-block` are preserved
- [ ] The delete button calls `stopPropagation()` and honours the `detail > 1` guard
- [ ] No `businessHours` prop on the FullCalendar instance
- [ ] `.brk-calendar-google` wrapper class unchanged
- [ ] `CalendarView.tsx` is under 400 lines and smaller than before
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-07

### Dependency Notes

TASK-07 lands first so the toolbar and control ownership are stable before the grid's internals
change. TASK-09 depends on this task and targets the exact class names defined in step 1.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **`.fc-event` count doubles** — the renderer wrapped the content in an extra `.fc-event`-classed
  element, or FullCalendar's default content was not actually replaced. `eventContent` **replaces**
  children; it must never add a second event element (`TASK-02.spec.ts:100,111` asserts exactly 1).
- **`Sidebar Test Task` not found inside `.fc-event`** — the title is rendered outside the event
  element, or as `aria-label` only. It must be visible text inside `.fc-event`.
- **Delete click also opens the edit modal** — `stopPropagation()` missing.
- **Double-click on delete fires twice** — the `e.detail > 1` guard was dropped.
- **Delete button absent on past events** — the runtime `isPast` fallback was not reproduced for
  recurring blocks, which have no precomputed `isPast`.
- **Month view chips broken** — `arg.timeText` is empty in `dayGridMonth`; render the time span
  conditionally.
- **Title truncated to nothing in narrow time slots** — the chip needs `min-height` / `overflow`
  rules; those live in the stylesheet (TASK-09), so verify the text is present in the DOM rather than
  visually.
- **`eventClick` no longer fires for the title** — `EventContent` must not intercept clicks on the
  title itself; only the delete button stops propagation.
- **Interim visual regression is expected** — between this task and TASK-09 the old
  `.fc-event-title` / `.fc-event-time` rules no longer match, so events render unstyled. This is
  expected and must be resolved by TASK-09, which lands immediately after. Record it in the report.
- **400-line ceiling** — `CalendarView.tsx` must end below 400 lines.

---

## Testing

### Unit Tests

- [ ] None — no component test runner exists and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, in particular the five `.fc-event:has-text('…')` assertions (`TASK-02.spec.ts:87,128,161,180,204`) and the `.fc-timegrid-slot:has-text('14:00')` click

### Manual Validation

- [ ] Delete button appears on generated/block/preview events and not on past or external events
- [ ] Clicking delete removes the event and does not open the edit modal
- [ ] Completed events differ visually from active ones (after TASK-09)
- [ ] Month view shows title-only chips with no empty time row

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Do not add FullCalendar drag/select automation; the suite already covers deterministic interactions
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

- [`planning/impact-analysis.md` — M5, B3](../planning/impact-analysis.md#material-corrections-m1m18)
- [`planning/risk-register.md` — R-03](../planning/risk-register.md#critical-and-high-risks)
- [`design-docs/design-04-calendar-ux.md` — eventContent](../design-docs/design-04-calendar-ux.md)
- `apps/web/src/components/CalendarView.tsx`