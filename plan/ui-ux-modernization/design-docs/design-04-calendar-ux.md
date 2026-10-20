# Design 04 — Calendar UX: `--fc-*` Theming, Custom Toolbar, `eventContent`

## Context

### Problem Statement

This is the user's core complaint: *"the calendar part looks like just a table — it should be
turned into a well-crafted UX/UI."* Four concrete defects:

1. **`CalendarStyles.css` fights the library.** 17 rule blocks, every one of them
   `!important`, fighting FullCalendar's runtime-injected stylesheet instead of using the
   `--fc-*` custom properties FullCalendar v6 provides for exactly this purpose.
2. **No information hierarchy.** `headerToolbar={false}` means FullCalendar renders only the
   grid. There is no title, no context, no legend, no summary — the page looks like a bare
   `<table>` with sticky headers.
3. **Imperative DOM mutation.** `CalendarView.tsx:278-332` builds a delete button with
   `document.createElement`, a hardcoded `rgba(0,0,0,0.3)` circle and 13 inline style
   assignments inside `eventDidMount`, then removes it in `eventWillUnmount`. It is
   unreachable by React, unstyled by tokens, and untestable.
4. **Dead and missing state styles.** `CalendarStyles.css:157` styles `.brk-completed`, but
   `CalendarView.tsx:163-192` never emits that class — `isCompleted` is passed in
   `extendedProps` and then ignored. Completed events are visually identical to active ones.

### Goals

- Replace all 17 `!important` blocks with a declarative `--fc-*` custom-property layer.
- Add a custom React toolbar above the grid that creates visual hierarchy.
- Replace the imperative delete button with a real `eventContent` renderer.
- Emit the missing `brk-completed` / `is-past` classes so the CSS actually applies.
- Business-hours shading and sticky day headers.
- Preserve every FullCalendar DOM node and class the Playwright suite depends on.

### Constraints — **Hard, non-negotiable (E2E suite)**

`apps/web/e2e/TASK-01.spec.ts` and `TASK-02.spec.ts` must pass unmodified. Therefore:

| Assertion | Location | Constraint |
|---|---|---|
| `.fc-timegrid-slots` | TASK-01:142, TASK-02:137 | Slot container must exist |
| `.fc-timegrid-slot:has-text('14:00')` | TASK-01:146, TASK-02:132 | A slot labelled `14:00` must exist and be clickable |
| `.fc-timegrid-body` | TASK-01:151, TASK-02:137 | Used for mouse-position calculations → must keep real geometry |
| `.fc-event:has-text('…')` | TASK-02:87,128,161,180,204 | **Event title text must be inside the `.fc-event` element** |
| `.fc-event` count | TASK-02:100,111 | One `.fc-event` per calendar event — no extra `.fc-event` elements |
| `aside` | TASK-02:82 | See `design-03` |
| Exact copy | both specs | See `design-03` StatusBar table |

**The library must not restyle FullCalendar's internals with Mantine components.** FullCalendar
renders its own DOM; Mantine only provides the toolbar and the event chrome around it.

---

## Options Considered

### Option A (Selected): `--fc-*` variable layer + custom React toolbar + `eventContent`

#### Description

Keep `@fullcalendar/react` and all four plugins. Replace `CalendarStyles.css` with a
`--fc-*` custom-property block scoped to `.brk-calendar`, add a React `CalendarToolbar`
above the grid, and supply an `eventContent` render function that produces the event chrome
(time, title, state) plus an accessible delete action.

#### Advantages

- Uses FullCalendar's documented override mechanism; no specificity war, no `!important`.
- FullCalendar's internal DOM is completely untouched → E2E suite safe by construction.
- The toolbar is ordinary React + Mantine, so it gets `SegmentedControl`, `ActionIcon`,
  `Badge`, tooltips and keyboard support for free.
- `eventContent` puts the delete action in the React tree: it can use `Tooltip`, be
  keyboard-focusable, carry an `aria-label`, and honour `prefers-reduced-motion`.
- Smallest diff of the three options and reversible file-by-file.

#### Disadvantages

- FullCalendar's `eventContent` receives the whole event, so the render function must be
  defensive about `extendedProps` shape.
- Custom `eventContent` disables FullCalendar's default title/time rendering — we own the
  markup and must keep the title text present for `has-text()`.

#### Effort Estimate

Medium

---

### Option B (Rejected): Adopt `@mantine/schedule` (new in Mantine 9)

**Rejected — blocked.** `@mantine/schedule` requires Mantine 9, which requires **React 19.2+**
(this project is React 18.2). It would also replace the FullCalendar DOM that the E2E suite
asserts on, breaking `.fc-timegrid-slots`, `.fc-event`, etc. Both are hard blockers. Recorded
here so the option is not re-proposed without a React 19 upgrade *and* an E2E rewrite.

### Option C (Rejected): FullCalendar Premium resource-timeline

**Rejected.** Paid licence, and the resource-timeline view renders an entirely different DOM,
which breaks every `.fc-*` assertion in the suite.

### Option D (Rejected): Hand-build the calendar grid from scratch

**Rejected.** Maximum control, but it destroys drag/resize/select, the `.fc-timegrid-slot`
geometry the tests compute mouse coordinates against, and is far beyond the approved scope.

---

## Decision

### Selected Option

Option A.

### Reasoning

It is the only option that fixes the reported problem (no visual hierarchy) without
invalidating the acceptance criteria. The `!important` removal is a genuine maintainability
win: FullCalendar v6 exposes `--fc-border-color`, `--fc-page-bg-color`, `--fc-neutral-bg-color`,
`--fc-neutral-text-color`, `--fc-today-bg-color`, `--fc-now-indicator-color`,
`--fc-non-business-color`, `--fc-highlight-color`, `--fc-event-bg-color`,
`--fc-event-border-color`, `--fc-event-text-color`, `--fc-event-resizer-thickness`,
`--fc-small-font-size`, `--fc-button-*`, `--fc-daygrid-event-dot-*` and `--fc-list-event-*`,
all intended to be set as custom properties. Because Mantine's CSS is static (loaded at build
time) and this block loads after it, the declaration order is deterministic.

### Trade-offs Accepted

- We own the event markup, including any regression in how times render.
- `--fc-*` covers appearance only; layout-level overrides (sticky headers, event min-height)
  still need 2–3 scoped rules without `!important`, relying on `.brk-calendar` scoping plus
  source order.

---

## Impact

### Documentation Affected

- `design-02-theme-token-architecture.md` (token names consumed here)
- `design-07-risks-and-e2e-register.md`

### Architecture Affected

- `apps/web/src/components/CalendarView.tsx` — `eventContent` added; `eventDidMount` /
  `eventWillUnmount` delete-button logic **removed**; `classNames` extended
- `apps/web/src/styles/CalendarStyles.css` — rewritten as a `--fc-*` layer
- **new** `apps/web/src/components/calendar/CalendarToolbar.tsx`
- **new** `apps/web/src/components/calendar/EventContent.tsx`
- **new** `apps/web/src/components/calendar/calendarTheme.ts` — maps `--brk-*` → `--fc-*`
- `apps/web/src/pages/DashboardPage.tsx` — renders `CalendarToolbar` above `CalendarView`

---

## Specification

### 1. `--fc-*` variable layer

`CalendarStyles.css` is rewritten to declare variables on `.brk-calendar` (the existing
wrapper class is retained as the scope root), and to contain **only** these rules:

```
.brk-calendar {
  --fc-border-color:            var(--brk-border-subtle);
  --fc-page-bg-color:           var(--brk-bg-raised);
  --fc-neutral-bg-color:        var(--brk-bg-sunken);
  --fc-neutral-text-color:      var(--brk-text-muted);
  --fc-today-bg-color:          var(--brk-today-tint);
  --fc-non-business-color:      var(--brk-bg-subtle);
  --fc-now-indicator-color:     var(--brk-now-indicator);
  --fc-highlight-color:         color-mix(in srgb, var(--brk-accent-brand) 18%, transparent);
  --fc-event-bg-color:          var(--brk-event-generated);
  --fc-event-border-color:      var(--brk-event-generated);
  --fc-event-text-color:        var(--brk-text-on-brand);
  --fc-event-resizer-thickness: 8px;
  --fc-small-font-size:         0.8125rem;
  --fc-daygrid-event-dot-width: 8px;
  --fc-list-event-hover-bg-color: var(--brk-bg-sunken);
}
```

The two hardcoded Google reds (`#ea4335` at `CalendarStyles.css:41,43`) are replaced by
`--brk-now-indicator`, which is the point of the token layer.

Event-variant rules keep their existing class names and drop every `!important`:

```
.brk-calendar .fc-event.brk-generated { background: …; border-left: 4px solid var(--brk-event-generated); }
.brk-calendar .fc-event.brk-external  { … var(--brk-event-external) … }
.brk-calendar .fc-event.brk-preview   { … dashed border, var(--brk-event-preview) … }
.brk-calendar .fc-event.brk-block     { … coral stripe, pointer-events: none … }
.brk-calendar .fc-event.brk-completed { opacity: .55; }        /* now actually emitted */
.brk-calendar .fc-event.is-past      { opacity: .45; filter: saturate(.6); }
.brk-calendar .fc-event .brk-event-delete-btn { /* styles for the React-rendered button */ }
```

`.fc-timegrid-event { min-height: 100% }` and the `overflow: visible` pair from
`CalendarStyles.css:171-181` are kept (they are load-bearing for the delete button) but
restated without `!important`.

### 2. Sticky day headers

```
.brk-calendar .fc-timegrid-col-header { position: sticky; top: 0; z-index: var(--z-sticky); background: var(--brk-bg-raised); }
```

Sticky positioning is safe here because the calendar's scroll container is
`DashboardPage.tsx:476` (`overflow:auto`) and FullCalendar's own sticky time-gutter already
uses the same technique. QA must verify the sticky time gutter (`fc-timegrid-axis`) still
overlaps correctly.

### 3. Business-hours shading

Pass `businessHours={{ daysOfWeek: settings.workDays, startTime, endTime }}` to FullCalendar
using the existing `settings.workStartHour` / `workEndHour`. Combined with the existing
`slotMinTime` / `slotMaxTime` (which already clamp the visible range to the working window),
`--fc-non-business-color` shades non-working **days**.

> **Constraint:** `settings.workDays` must include the day under test and the working window
> must contain `14:00`, otherwise `businessHours` de-emphasises the `.fc-timegrid-slot:has-text('14:00')`
> the specs click. `highlightOverlaps: false` is set so events outside business hours are not
> visually flagged. This is the single highest-risk change in the calendar work and is
> registered as **R-CAL-01**.

### 4. Custom React toolbar (`CalendarToolbar`)

Replaces the current `Header` responsibility for calendar navigation (the app `Header`
retains brand/actions only). Rendered by `DashboardPage` **above** `CalendarView`, inside the
`AppShell.Main` column. `headerToolbar={false}` stays.

```
<Group justify="space-between" wrap="nowrap" className="brk-calendar-toolbar">
  <Group gap="xs">
    <ActionIcon aria-label="Previous period" onClick={onPrev}><IconChevronLeft/></ActionIcon>
    <Button variant="default" onClick={onToday}>Today</Button>
    <ActionIcon aria-label="Next period" onClick={onNext}><IconChevronRight/></ActionIcon>
    <Text component="h2">{dateTitle}</Text>
  </Group>
  <Group gap="sm">
    <Badge variant="light">{businessHoursLabel}</Badge>
    {hasUnsavedChanges && <Badge color="yellow" variant="light">Unsaved changes</Badge>}
    <SegmentedControl aria-label="Calendar view" data={VIEWS} value={currentView} onChange={onViewChange} />
  </Group>
</Group>
```

**Copy rule (strict):** the toolbar introduces the strings `"Today"`, `"Previous period"`,
`"Next period"`, `"Calendar view"`, `"Unsaved changes"` and a business-hours label. It must
**not** introduce any string containing `"Preview"`, `"New Task"`, `"Edit Task"`, `"Tasks ("`,
`"Create in this slot"`, `"Loading calendar events"`, `"Schedule saved successfully"` or
`"Failed to sync calendar events"`. The business-hours badge therefore reads e.g.
`"09:00 – 18:00"` and **must not** be labelled with the word "Preview" in any tier or locale.

This resolves the duplication concern: the current `Header` already owns navigation, so the
toolbar **replaces** it rather than being added alongside — exactly one "Today" button will
exist in the DOM.

### 5. `eventContent` renderer

Replaces the `eventDidMount` DOM injection entirely.

```tsx
const renderEventContent = (arg: EventContentArg) => {
  const { event } = arg;
  const p = event.extendedProps as Partial<CalendarEvent> & { isBlock?: boolean };
  const showDelete = Boolean(p.isGenerated || p.isBlock || p.isPreview) && !isPast(event);
  return (
    <div className="brk-event-content">
      <span className="brk-event-time">{arg.timeText}</span>
      <span className="brk-event-title">{event.title}</span>
      {p.isCompleted && <IconCheck className="brk-event-check" aria-hidden />}
      {showDelete && (
        <Tooltip label="Delete event" withArrow>
          <ActionIcon
            className="brk-event-delete-btn"
            size="xs" variant="subtle" color="gray"
            aria-label={`Delete ${event.title}`}
            onClick={(e) => { e.stopPropagation(); onDeleteRequest(event.id, event.title); }}
          ><IconX size={12} /></ActionIcon>
        </Tooltip>
      )}
    </div>
  );
};
```

Contract with the E2E suite:

- `event.title` **must** be rendered inside the `.fc-event` element — `.fc-event:has-text('Sidebar Test Task')`
  depends on it. This is the single most important line of the renderer.
- Exactly one `.fc-event` per event; `eventContent` does not add elements, it replaces
  children, so `.fc-event` counts are unchanged.
- `.brk-event-delete-btn` class name is preserved even though the element is now React-owned,
  so the existing CSS selectors in `CalendarStyles.css` keep working.
- The `e.detail > 1` double-click guard from the old code (`CalendarView.tsx:319-320`) is
  preserved on the React `onClick`.
- `eventWillUnmount` is deleted — React owns cleanup.

### 6. Class-name emission

`CalendarView.tsx:163-192` currently emits only `brk-generated` / `brk-external` /
`brk-preview` / `brk-block`. Extend to:

```
taskEvents:  ["brk-generated"|"brk-external", p.isCompleted && "brk-completed", isPast && "is-past"]
previewMapped: ["brk-preview", isPast && "is-past"]
blockMapped:  ["brk-block", isPast && "is-past"]
```

---

## Risks

| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R-CAL-01 | `businessHours` de-emphasises or removes the `14:00` slot the specs click | Medium | **High** | Configure from `settings`; add an explicit guard so the working window always covers the configured work hours; QA runs both specs in every colour scheme |
| R-CAL-02 | `eventContent` drops the title text → all 5 `.fc-event:has-text(...)` assertions fail | Medium | **High** | Title rendered as a dedicated `<span className="brk-event-title">`; verified by the existing suite with no test modification |
| R-CAL-03 | Removing `!important` from `CalendarStyles.css` lets FullCalendar defaults win | Medium | Medium | Layered under `.brk-calendar` scoping + declared after `@mantine/core/styles.css`; visual diff reviewed in QA |
| R-CAL-04 | Sticky headers break the sticky time gutter geometry | Low | Medium | QA screenshot check of the time-axis overlap; fallback is `position: relative` on `.fc-timegrid-axis` |
| R-CAL-05 | `eventContent` breaks month view (`dayGridMonth`) chip layout | Medium | Low | Month view needs no time text; render `arg.timeText` conditionally on `view.type.startsWith('timeGrid')` |
| R-CAL-06 | Mantine's CSS reset alters FullCalendar's box model | Low | Medium | `box-sizing: border-box` matches; verify `.fc-timegrid-body` height chain in QA |

---

## Open Questions

- Should the toolbar show a week summary (e.g. "12 events · 4h 30m scheduled")? It would be a
  strong hierarchy win but adds a derived-data concern. Default: out of scope for this
  delivery; listed as a follow-up enhancement.
- Business-hours badge copy — confirm `"09:00 – 18:00"` is acceptable, or prefer a label such
  as `"Working hours"`. Neither collides with any E2E-asserted string.