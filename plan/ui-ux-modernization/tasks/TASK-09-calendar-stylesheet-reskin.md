# TASK-09 — Calendar Stylesheet Re-skin and Split

## Task Information

### ID

TASK-09

### Title

Rewrite `CalendarStyles.css` as a `--fc-*` custom-property layer scoped to `.brk-calendar-google`, drop all 17 `!important` blocks, add CSS-only workday tint and split into partials

### Phase

Phase 5c — Calendar presentation layer

### Owner

senior-frontend

### Status

Pending

---

## Description

`CalendarStyles.css` (181 lines) contains 17 rule blocks, every one of them `!important`, fighting
FullCalendar's runtime-injected stylesheet. This task replaces that specificity war with the
documented `--fc-*` custom-property mechanism, retargets every selector at the class the code
actually renders, applies the contrast-corrected dark palette, and delivers the workday tint through
CSS (`B3` defers the `businessHours` prop).

The re-skin will exceed 400 lines as a single file, so the split is planned up front (`M17`).

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/styles/CalendarStyles.css` | modify — layout/grid only |
| 2 | `apps/web/src/styles/calendar/calendar-tokens.css` | **create** — `--fc-*` variable layer |
| 3 | `apps/web/src/styles/calendar/calendar-events.css` | **create** — event variants + event content |
| 4 | `apps/web/src/styles/calendar/calendar-dark.css` | **create** — dark-tier overrides |
| 5 | `apps/web/src/main.tsx` | modify — append the partial imports after `CalendarStyles.css` |

### File count

5 (3 created, 2 modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/components/CalendarView.tsx`, `components/calendar/EventContent.tsx`,
  `CalendarToolbar.tsx`, `calendarTheme.ts` — TASK-07 / TASK-08 own them
- `apps/web/src/styles/design-tokens.css`, `global.css` — TASK-02 owns them; consume `--brk-*`
  read-only
- `apps/web/src/components/Header.tsx`, `MainLayout.tsx`, `Sidebar.tsx`, `StatusBar.tsx`
- `apps/web/src/pages/DashboardPage.tsx` — **OVERLAP-1** (`P-8`)
- `apps/web/src/features/**`, `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `apps/web/index.html`, `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)

---

## Constraints Satisfied

- `B3` — workday tint via CSS only; **no** `businessHours` prop
- `B4` — the whole event palette is contrast-checked in both tiers before this task exits
- `M1` — every selector is scoped to **`.brk-calendar-google`**, not `.brk-calendar`
- `M5` — target the project-owned classes TASK-08 emits; no `.fc-event-title` / `.fc-event-time`
- `M6` — theme via custom properties; **no** new `!important`
- `M16` — do not add competing sticky-header CSS
- `M17` — split into partials before the file exceeds 400 lines
- `M13` — partials load after the token sheets and after Mantine
- `P-9` — every stylesheet under 400 lines

---

## Implementation Steps

1. **Create `styles/calendar/calendar-tokens.css`** — the `--fc-*` variable layer, scoped to
   `.brk-calendar-google` (the real wrapper class rendered by `CalendarView.tsx`; `M1`). Declare:
   `--fc-border-color`, `--fc-page-bg-color`, `--fc-neutral-bg-color`, `--fc-neutral-text-color`,
   `--fc-today-bg-color`, `--fc-non-business-color`, `--fc-now-indicator-color`,
   `--fc-highlight-color`, `--fc-event-bg-color`, `--fc-event-border-color`,
   `--fc-event-text-color`, `--fc-event-resizer-thickness`, `--fc-small-font-size`,
   `--fc-daygrid-event-dot-width`, `--fc-list-event-hover-bg-color`.
   Every value MUST be a `var(--brk-*)` reference — no literal hexes except where a literal is the
   only option, and none of those may be the hardcoded Google reds `#ea4335` currently at lines 41,
   45 and 46 (replace with `var(--brk-now-indicator)`).
   **Workday tint (`B3`)** — add a CSS-only workday tint using the existing FullCalendar day-column
   classes. Preferred: `.brk-calendar-google .fc-col-header-cell[data-day]` / `.fc-daygrid-day`
   classes combined with a `:not()` selector for the weekend day numbers, applied to
   `.fc-timegrid-col[data-date]` lanes. Keep the existing weekend tint
   (`.fc-day-sat`, `.fc-day-sun`) and make the workday tint a distinct, subtler surface so the
   distinction is visible without touching the time slots. **Do not** use `.fc-non-business`, which
   is the class `businessHours` produces — the whole point of `B3` is to avoid it.
2. **Create `styles/calendar/calendar-events.css`** — event variants and the new event content.
   - Targets **only** the project-owned classes TASK-08 emits: `.brk-event-content`,
     `.brk-event-time`, `.brk-event-title`, `.brk-event-check`, `.brk-event-delete-btn`.
   - Remove every `.fc-event-title` and `.fc-event-time` rule — FullCalendar no longer renders those
     internals under `eventContent` (`M5`).
   - Keep the variant rules with their existing class names and **no** `!important`:
     `.fc-event.brk-generated`, `.brk-external`, `.brk-preview`, `.brk-block`,
     `.brk-completed` (now actually emitted), `.is-past`, `.brk-event-delete-btn`.
   - **Preserve** the two load-bearing layout rules from lines 171–181: `.fc-timegrid-event { min-height: 100% }`
     and `overflow: visible` on `.fc-event` and `.fc-event-main` — restated without `!important`.
     These are what let the delete button escape FullCalendar's clipping.
3. **Create `styles/calendar/calendar-dark.css`** — dark-tier overrides inside a
   `[data-mantine-color-scheme="dark"] .brk-calendar-google { … }` block: event fills, grid lines,
   today tint, all-day row, now indicator. **`B4` mandatory:** every event fill must reach ≥ 4.5:1
   against its text and ≥ 3:1 for non-text. `#3CC4BD` must **not** be paired with white text
   (≈2.1:1). Cross-check the values chosen in TASK-02's token file and reuse them.
4. **Rewrite `styles/CalendarStyles.css`** as the layout/grid partial: grid lines, day-header type,
   time gutter, today cell, weekend tint, all-day slot, week numbers, and the `min-height` /
   `overflow` pair. Every selector scoped to `.brk-calendar-google`. **Zero `!important`**
   declarations in this file (down from 17 blocks across the original sheet). If a rule genuinely
   needs to beat FullCalendar's runtime injection, express it as a custom property or raise scoping
   specificity — never `!important` (`M6`).
5. **`M16` — sticky headers.** FullCalendar's timegrid plugin already applies sticky positioning in
   JS via `.fc-sticky`. **Do not** add custom sticky CSS for the day-header row until the built-in
   behaviour has been observed. If a rule is genuinely required, scope it away from `.fc-sticky` so
   it cannot fight the library.
6. **Modify `apps/web/src/main.tsx`** — append the three partial imports **after**
   `./styles/CalendarStyles.css`, preserving the order: `calendar-tokens.css`,
   `calendar-events.css`, `calendar-dark.css`. The full order becomes:
    `@mantine/core/styles.css` → `@mantine/notifications/styles.css`
   → `design-tokens.css` → `global.css` → `CalendarStyles.css` → the three partials (`M13`).
    TASK-03 owns the initial imports; only append. Installed `@mantine/modals` does not export `styles.css`.
7. **Verify no `!important` remains** across `CalendarStyles.css` and the partials, and that no
   `.brk-calendar` (without `-google`) selector exists anywhere.

---

## Acceptance Criteria

- [ ] Every selector in all four calendar stylesheets is scoped to `.brk-calendar-google`
- [ ] The string `.brk-calendar {` (without `-google`) does not appear
- [ ] Zero `!important` declarations remain in the calendar stylesheets
- [ ] `calendar-tokens.css` declares the full `--fc-*` layer with only `var(--brk-*)` values
- [ ] `.fc-event-title` and `.fc-event-time` selectors are **gone**
- [ ] `.brk-event-title`, `.brk-event-time`, `.brk-event-delete-btn`, `.brk-event-check`, `.brk-event-content` are styled
- [ ] The work-day tint is delivered by CSS on FullCalendar day-column classes; `.fc-non-business` is **not** introduced and no `businessHours` prop exists
- [ ] `#ea4335` does not appear in any calendar stylesheet
- [ ] `calendar-dark.css` has ≥ 4.5:1 contrast for every event fill/text pair (documented in a comment)
- [ ] `min-height: 100%` on `.fc-timegrid-event` and the `overflow: visible` pair are preserved
- [ ] No custom sticky-header CSS competes with `.fc-sticky`
- [ ] All four stylesheets are under 400 lines
- [ ] `main.tsx` import order ends with the three partials, and its first six imports are unchanged
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-08

### Dependency Notes

TASK-08 must land first: this task styles the exact class names the `eventContent` renderer emits.
TASK-02 must have landed earlier to supply the `--brk-*` tokens and the breakpoint/`color-mix`
decisions.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **Whole re-skin silently no-ops** — selectors copied from the design docs target `.brk-calendar`,
  which matches nothing. Always verify against `.brk-calendar-google` (`M1`).
- **FullCalendar defaults win despite no `!important`** — the custom-property layer is missing an
  entry, or the stylesheet loads before FullCalendar's runtime injection. Custom properties inherit
  and win regardless of order; verify the block is on the wrapper element itself.
- **Delete button clipped again** — the `overflow: visible` / `min-height: 100%` pair was dropped
  while splitting. Those two rules are load-bearing.
- **Completed / past states still invisible** — `.brk-completed` / `.is-past` are not emitted by
  `CalendarView.tsx`. That is TASK-08's fix; if still missing, report it rather than editing
  `CalendarView.tsx` (it is not this task's file).
- **Sunday slot no longer clickable** — the workday tint was implemented by shading or overlaying
  the **time slots** instead of the day columns. `TASK-01`/`TASK-02` click the Sunday `14:00` slot;
  it must stay a real, clickable `.fc-timegrid-slot` with no overlay (`B3`).
- **Sticky header jitter** — custom sticky CSS was added on top of `.fc-sticky` (`M16`).
- **Dark tier event text unreadable** — a dark fill was paired with dark text, or the `B4`
  contrast check was skipped. Re-derive from the TASK-02 tokens.
- **CSS bundle doubles** — a partial is imported twice, or the token sheet got imported again here.
- **400-line ceiling** — a partial is approaching the limit; split it further rather than growing
  `CalendarStyles.css`.
- **Interim-styling regression** — if TASK-08 landed without its CSS, events are briefly unstyled.
  This task must close that window.

---

## Testing

### Unit Tests

- [ ] None — stylesheet-only change

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, in particular `.fc-timegrid-slots`, `.fc-timegrid-body`, `.fc-timegrid-slot:has-text('14:00')`, `.fc-event:has-text('…')`

### Manual Validation

- [ ] Calendar is visually re-skinned in both tiers; no hardcoded hex visible in devtools
- [ ] The Sunday `14:00` slot is clickable and unshaded
- [ ] The day-header row does not jitter while scrolling the grid
- [ ] The delete button is not clipped inside an event chip

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] `.fc-timegrid-body` geometry must stay real — `boundingBox()` is used for mouse math
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] Zero `!important`, correct scope, partials split, contrast verified
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/impact-analysis.md` — M1, M5, M6, M16, M17, B3, B4](../planning/impact-analysis.md#material-corrections-m1m18)
- [`planning/dispatch-plan.md` — Phase 5 Gate](../planning/dispatch-plan.md#phase-5-gate--calendar)
- [`design-docs/design-04-calendar-ux.md`](../design-docs/design-04-calendar-ux.md)
- `apps/web/src/styles/CalendarStyles.css`, `apps/web/src/styles/design-tokens.css`
