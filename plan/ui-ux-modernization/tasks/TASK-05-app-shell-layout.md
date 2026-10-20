# TASK-05 — App Shell: `AppShell`, Header and Sidebar on Mantine

## Task Information

### ID

TASK-05

### Title

Migrate `MainLayout`, `Header` and `Sidebar` to Mantine `AppShell` with an explicitly bounded height and exactly one `<aside>`

### Phase

Phase 4a — App shell

### Owner

senior-frontend

### Status

Pending

---

## Description

Replace the hand-rolled flex shell with a Mantine `AppShell` grid. Two defects in the current shell
must be fixed rather than reproduced: `AppShell`'s default `min-height` breaks the calendar's
`height="100%"` chain, and any navbar-plus-drawer arrangement that mounts the sidebar twice produces
a duplicate `<aside>` and a duplicate `Tasks (N)` panel.

This task deliberately **leaves the date-step buttons and the `Day`/`Week`/`Month` view switcher in
the `Header`**. TASK-07 moves them into `CalendarToolbar` once the calendar chrome exists — moving
them now would leave the page with no view switcher and break the E2E suite at this gate.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/components/MainLayout.tsx` | modify — `AppShell` layout |
| 2 | `apps/web/src/components/Header.tsx` | modify — Mantine primitives, keep all controls |
| 3 | `apps/web/src/components/Sidebar.tsx` | modify — Mantine `Accordion`, single `<aside>` |

### File count

3 (all modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/pages/DashboardPage.tsx` — TASK-06 owns it; **OVERLAP-1**
- `apps/web/src/styles/**` — TASK-02 / TASK-09 own the stylesheets
- `apps/web/src/theme/**` — TASK-03 owns it
- `apps/web/src/features/**`, `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `apps/web/src/components/StatusBar.tsx` — TASK-06 creates it
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)

---

## Constraints Satisfied

- `B1` — `Day` / `Week` / `Month` MUST remain real `<button>` elements; `SegmentedControl` is forbidden
- `M3` — the `AppShell` root MUST have an explicitly bounded height
- `M7` — exactly one `<aside>` and exactly one task panel at every viewport
- `M4` — the `AppShell` `breakpoint` value MUST come from `src/theme/breakpoints.ts`
- `M15` (partial) — ownership is unchanged in this task; `Today`, date title and the view switcher
  all stay in the `Header` until TASK-07
- `P-9` — all three files stay under 400 lines
- `R-04`, `R-16`, `R-22`, `R-23` — height chain, control ownership, polymorphic typing, copy uniqueness

---

## Implementation Steps

1. **`MainLayout.tsx` → `AppShell`.** Compose:
   ```tsx
   <AppShell
     header={{ height: 64 }}
     navbar={{ width: 280, breakpoint: "sm", collapsed: { mobile: !mobileNavOpen } }}
     padding={0}
     transitionDuration={200}
     transitionTimingFunction="ease"
   >
   ```
   - Import `breakpoint`-relevant values from `src/theme/breakpoints.ts` — **do not hardcode**
     `sm`/`md` numbers (`M4`).
   - Keep the `MainLayoutProps` interface **unchanged** (it is consumed by `DashboardPage.tsx`, which
     this task must not touch). Keep `Header`, `Sidebar` and `<main>` in the same order.
2. **`M3` — bound the shell height explicitly.** `AppShell` defaults to `min-height: 100dvh`, which
   imposes no ceiling and breaks `<FullCalendar height="100%">`. Apply on the `AppShell` root:
   `height: 100dvh` with an explicit `height: 100vh` fallback declaration before it, and
   `overflow: hidden`. On the scroll container that hosts the calendar, apply
   `min-height: 0` and `overflow: auto`. The current code's bounded box is exactly what makes the
   calendar viewport-sized; it must be reproduced deliberately, not inherited.
3. **`Sidebar.tsx`.**
   - Render the sidebar content inside a **plain `<aside>` element** (guaranteed to compile and to
     satisfy `page.locator("aside")`). Do **not** rely on `AppShell.Navbar component="aside"` —
     that polymorphic typing is unverified (`R-22`). Place the `<aside>` **inside**
     `AppShell.Navbar`.
   - **Do not add a Mantine `Drawer`.** A drawer holding a second `<Sidebar>` instance would mount
     two `<aside>` elements and two `Tasks (N)` panels and fail `TASK-02` strict mode (`M7`,
     `R-23`). Instead use `AppShell.Navbar` with `collapsed: { mobile: !mobileNavOpen }` so exactly
     one sidebar instance exists at every viewport.
   - Replace the three hand-rolled section buttons with a Mantine `Accordion` (`multiple={false}`)
     covering the same three sections and the **same copy**: `Tasks`, `Blocked Slots`, `Settings`.
     The `Settings` control must still call `onSettingsClick()`.
    - Mantine 8.3.18 `AccordionPanel` does not accept a `keepMounted` prop. Keep the accordion
      section containing draggable tasks open while dragging; verify drag behavior with the panel expanded.
   - Move the collapse toggle out of its absolutely-positioned floating position into an
     `ActionIcon` with `aria-label="Toggle sidebar"` and `aria-expanded`.
   - Keep `SidebarProps` (`isCollapsed`, `onToggle`, `onSettingsClick`) and the `TaskPanel` /
     `BlockPanel` usage unchanged.
4. **`Header.tsx`.** Keep the same five controls and the same copy:
   - `‹` / `›` date-step buttons → Mantine `ActionIcon` with `aria-label="Previous period"` /
     `"Next period"` (today they have no accessible name at all — that is the a11y defect being
     fixed). The visible glyph may become a chevron icon; the accessible name must not change.
   - `Today` → `Button` with the label exactly `Today`.
   - Date title → keep the existing `toLocaleDateString("en-US", …)` logic untouched, rendered as
     `Text component="h2"`.
   - `Day` / `Week` / `Month` view switcher → **`B1` mandatory.** Build it from a flex container of
     three Mantine `Button` (or `UnstyledButton`) elements with segmented styling, **one per view**,
     each rendering **visible text exactly** `Day`, `Week`, `Month`. Add `aria-pressed` for state.
     **`SegmentedControl` MUST NOT appear** — it emits `<input type="radio">` + `<label>` and no
     `<button>` at all, which fails `button:has-text("Day")` and `button:has-text("Week")`.
     Keep the `VIEW_LABELS` and `VIEWS` constants unchanged.
   - `Settings` → `Button` with the label exactly `Settings`, calling `onSettingsClick`.
   - Add an `ActionIcon` colour-scheme toggle with `aria-label="Toggle color scheme"` using
     `useMantineColorScheme` from `@mantine/core`.
   - Style with `--brk-*` tokens only; remove every hardcoded hex and inline `style` object.
   - **Add no new visible copy.** No string may contain `Preview`, `New Task`, `Edit Task`,
     `Tasks (`, `Create in this slot`, `Loading calendar events`, `Schedule saved successfully`, or
     `Failed to sync calendar events` (`R-23`).
   - `HeaderProps` is consumed by `MainLayout`, so keep it source-compatible.

---

## Acceptance Criteria

- [ ] `MainLayout` renders `AppShell` / `AppShell.Header` / `AppShell.Navbar` / `AppShell.Main` with `padding={0}`
- [ ] The `AppShell` root has an explicit bounded height (`100vh` fallback + `100dvh`) and `overflow: hidden`; the scroller has `min-height: 0` + `overflow: auto`
- [ ] Exactly **one** `<aside>` element exists in the document at 1440×900
- [ ] Exactly **one** `Tasks (N)` panel is mounted; its label is a single contiguous text node inside one element
- [ ] `Day`, `Week` and `Month` are real `<button>` elements with exactly those visible labels
- [ ] The string `SegmentedControl` does not appear in any of the three files
- [ ] `‹`/`›` have accessible names `Previous period` / `Next period`
- [ ] Accordion copy is exactly `Tasks`, `Blocked Slots`, `Settings`; draggable tasks work with their panel expanded
- [ ] All three files reference `--brk-*` tokens; no hardcoded hex remains in them
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**
- [ ] All three files under 400 lines

---

## Dependencies

### Required Tasks

- TASK-04

### Dependency Notes

TASK-04 first because the shell may consume `@brkroutnxdle/ui` where convenient. TASK-06 depends on
this task and rewrites `DashboardPage.tsx`, which still imports `MainLayout` with the unchanged prop
interface — so this task must not change `MainLayoutProps`.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **Calendar taller than the viewport / page scrolls instead of the grid** — the classic `M3`
  symptom. The definite parent height is missing; re-apply steps 2 and re-check that
  `DashboardPage`'s calendar wrapper still carries `flex: 1; overflow: auto; minHeight: 0`.
- **Two `<aside>` elements** — a `Drawer` or a duplicated `<Sidebar>` was added. `TASK-02` then
  fails on the `aside` locator or on `Tasks (1)` strict mode.
- **Zero `<aside>`** — `AppShell.Navbar` was replaced by a `div`. Wrap the sidebar content in a
  literal `<aside>`.
- **`Tasks (1)` resolves to two elements** — the label was split (`Tasks` in one element, `1` in a
  `Badge`). It must be one expression inside one element.
- **`button:has-text("Week")` matches nothing** — a `SegmentedControl` slipped in, or the labels are
  icons without visible text. Both break `B1`.
- **`button:has-text("Day")` clicks `Today` instead** — Playwright's `has-text` is a case-insensitive
  substring match, so `Today` contains `day`. This does not fail the suite (the assertion is an
  event count), but the view switcher must still be three distinct real buttons for the feature to
  work. Keep the order stable and do not add any other button whose text contains `Day`/`Week`.
- **Task drag stops working** — check that the Tasks accordion section is expanded before dragging.
- **Toolbar/control duplication** — do **not** add date navigation or a view switcher anywhere else in
  this task; TASK-07 owns that move.
- **Hardcoded hex left behind** — grep the three files for `#` before finishing.
- **Breakpoint drift** — `breakpoint: "sm"` must resolve through `theme/breakpoints.ts`; never
  hardcode `48em` here.
- **400-line ceiling** — if a file approaches 400 lines, extract the header control cluster into a
  sibling component in the same directory (still owned by this task).

---

## Testing

### Unit Tests

- [ ] None — no unit test runner exists for `apps/web` components and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, including the single-`aside` and `Tasks (N)` assertions in `TASK-02.spec.ts` and the `Day`/`Week` switch in `TASK-02.spec.ts:105,108`

### Manual Validation

- [ ] Calendar fills the viewport and the **page** does not scroll at 1440×900
- [ ] Sidebar collapse/expand works and the rail does not shift content
- [ ] Colour-scheme toggle flips `data-mantine-color-scheme` on `<html>`
- [ ] `Today`, `Settings`, `Day`, `Week`, `Month` all render with exact copy

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] The existing suite is the only acceptance gate; `build` cannot detect selector drift (`M12`)
- [ ] Report pass/fail per spec file (`TASK-01.spec.ts`, `TASK-02.spec.ts`)

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/impact-analysis.md` — M3, M7, M15](../planning/impact-analysis.md#material-corrections-m1m18)
- [`planning/dispatch-plan.md` — Phase 4 Gate](../planning/dispatch-plan.md#phase-4-gate--app-shell)
- [`design-docs/design-03-app-shell-layout.md`](../design-docs/design-03-app-shell-layout.md)
- `apps/web/src/components/MainLayout.tsx`, `Header.tsx`, `Sidebar.tsx`
