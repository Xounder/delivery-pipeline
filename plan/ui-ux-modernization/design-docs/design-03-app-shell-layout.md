# Design 03 — App Shell, Header, Sidebar and Responsive Layout

## Context

### Problem Statement

The shell is three flexbox divs with inline styles:

- `MainLayout.tsx` — `display:flex; flexDirection:column; height:100vh; background:"#EFF8F7"`,
  then a flex row containing `Sidebar` and a bare `<main>`.
- `Header.tsx` — three hardcoded style objects; prev/next are `<button>‹</button>` with **no
  accessible name**; the view switcher is three hand-styled buttons with no `aria-pressed`;
  the date title is an `<h2>`.
- `Sidebar.tsx` — a `<div style={{position:"relative"}}>` wrapper around a `<aside>` whose
  width animates between `280px` and `0px`; the collapse toggle is absolutely positioned and
  flips `right` between `0px` and `280px`; three accordion buttons hold `TaskPanel`,
  `BlockPanel` and a settings action.
- `DashboardPage.tsx` — a toolbar row plus five stacked conditional banners, all inline-styled.

Result: no visual hierarchy, no elevation model, no keyboard/AT affordances, and a responsive
layer that exists in CSS but has no matching class names.

### Goals

- Replace the ad-hoc flex shell with a CSS-grid app shell with named areas.
- Make the header a proper application/tool bar: brand, date navigation, view switcher, actions.
- Make the sidebar a resizable/collapsible rail with real `<aside>` semantics preserved.
- Replace the five stacked banners with a single, non-layout-shifting status region.
- Make the responsive layer real (breakpoints at 62em and 36em).
- Zero changes to E2E-asserted copy and zero changes to DOM elements the tests reach.

### Constraints

- **`<aside>` must remain in the DOM** — `TASK-02.spec.ts:82` does
  `page.locator("aside").locator("text=Sidebar Test Task")`.
- `"Tasks ({n})"` must stay a **single contiguous text node** inside one element. If it is
  split (e.g. `Tasks` in a `<span>` and `(1)` in a `<Badge>`), `text=Tasks (1)` resolves to
  two elements and Playwright strict mode fails.
- **No new visible text may contain** `"Preview"`, `"New Task"`, `"Edit Task"`, `"Tasks ("`,
  `"Create in this slot"`, `"Loading calendar events"`, `"Schedule saved successfully"`,
  `"Failed to sync calendar events"` — the new chrome must not create a second match for any
  of these, which would also trip Playwright strict mode.
- `TaskPanel`, `BlockPanel` and `onSettingsClick` keep their current props and behaviour.

---

## Options Considered

### Option A (Selected): Mantine `AppShell` + `Drawer` composition, grid-based

#### Description

- `MainLayout` becomes a Mantine `AppShell` with `header` and `navbar` slots. Mantine's
  `AppShell` uses CSS grid internally (`grid-template-areas`), which replaces the manual
  flex nesting and gives correct `height: 100dvh` / overflow handling for free.
- `Header` becomes a Mantine `Group` inside `AppShell.Header` containing `ActionIcon`
  (chevrons), a `Button` for Today, a `Title`/`Text` date label, a `SegmentedControl` for
  Day/Week/Month, and right-aligned actions (color-scheme toggle, settings).
- `Sidebar` becomes `AppShell.Navbar` with an accordion of sections. Collapsed state uses
  `AppShell` `collapsed: { mobile: !opened }` and a width transition; the `<aside>` element is
  preserved by rendering `AppShell.Navbar` with `component="aside"`.
- `DashboardPage` banners collapse into a single `StatusBar` region rendered above the
  calendar, using `Alert` (success / error / loading) with `role="status"`.

#### Advantages

- Grid + overflow handling solved by the library rather than by hand-tuned flex.
- `AppShell.Navbar` accepts a polymorphic `component`, so `<aside>` is preserved natively.
- `SegmentedControl` ships `aria-pressed`-style semantics and keyboard arrows for free.
- `Drawer` on mobile gives a proper off-canvas sidebar below 62em, which the current CSS
  only half-implements.

#### Disadvantages

- `AppShell` controls padding/border defaults, so the visual result needs explicit overrides
  to match the existing flat aesthetic.
- AppShell introduces its own `config` prop for breakpoints; must be pinned so it agrees with
  the `design-02` breakpoints.

#### Effort Estimate

Medium

---

### Option B (Rejected): Keep the hand-rolled flex shell, restyle with tokens

**Rejected because** it keeps the structural problems (no off-canvas behaviour, absolute
positioned toggle, dead responsive CSS) and only changes colours. The approved Option C scope
explicitly includes the app shell.

### Option C (Rejected): Rebuild the shell on Mantine `Grid`/`Flex` primitives directly

**Rejected because** it re-implements `AppShell`'s header/navbar/main coordination by hand,
which is the exact flexbox nesting the redesign is trying to escape.

---

## Decision

### Selected Option

Option A — Mantine `AppShell` + `Drawer`, `SegmentedControl`, `Alert`/`StatusBar`.

### Reasoning

`AppShell` is the only library primitive that solves the specific defect (a flexbox column of a
header plus a flex row of sidebar plus main, with no responsive story). Its polymorphic
`component` prop is what lets us keep `<aside>` for the E2E suite — that constraint alone
disqualifies anything that would require a wrapper element change.

### Trade-offs Accepted

- Visual output depends on overriding several `AppShell` defaults; accepted in exchange for
  correct structure.
- Breakpoints must be declared in two places (Mantine `AppShell` config and CSS); both are
  pinned to the same values and cross-referenced in code comments.

---

## Impact

### Documentation Affected

- `design-05-component-inventory.md` (new shell primitives)
- `design-06-migration-plan.md`

### Architecture Affected

- `apps/web/src/components/MainLayout.tsx` — rewritten as `AppShell`
- `apps/web/src/components/Header.tsx` — rewritten on Mantine primitives
- `apps/web/src/components/Sidebar.tsx` — rewritten as `AppShell.Navbar` (`component="aside"`)
- `apps/web/src/pages/DashboardPage.tsx` — banners replaced by `StatusBar`
- `apps/web/src/styles/global.css` — dead `@media` blocks removed
- **new** `apps/web/src/components/StatusBar.tsx`

---

## Shell Specification

### Breakpoints (pinned, single source in `design-02`)

| Name | Width | Behaviour |
|---|---|---|
| `sm` | `>= 36em` (576px) | Navbar collapses to overlay `Drawer` |
| `md` | `>= 62em` (992px) | Navbar becomes a static, collapsible rail |
| `lg` | `>= 75em` (1200px) | Navbar width grows 280px → 320px |

Mantine `AppShell` is configured with exactly these `breakpoints` so its internal media
queries match the CSS layer.

### `MainLayout` (new shape)

```
<AppShell
  header={{ height: 64 }}
  navbar={{ width: 280, breakpoint: 'sm', collapsed: { mobile: !mobileNavOpen } }}
  padding={0}
  transitionDuration={200}
  transitionTimingFunction="ease"
>
  <AppShell.Header className="brk-shell-header"> ...Header... </AppShell.Header>
  <AppShell.Navbar component="aside" className="brk-shell-navbar"> ...Sidebar... </AppShell.Navbar>
  <AppShell.Main className="brk-shell-main">{children}</AppShell.Main>
</AppShell>
```

- `padding={0}` — the calendar manages its own gutter; `AppShell` default padding would
  inset the FullCalendar grid and break the `.fc-timegrid-body` height chain.
- `transitionDuration={200}` maps to `--motion-base`; all transitions must be disabled under
  `@media (prefers-reduced-motion: reduce)`.

### `Header` (new shape)

Left to right:

1. **Nav cluster** — `ActionIcon` chevron-left / chevron-right. Both receive
   `aria-label="Previous period"` / `aria-label="Next period"` (today they have **no**
   accessible name at all — an a11y defect this redesign fixes).
2. **`Button` "Today"** — copy preserved verbatim.
3. **Date title** — `Text component="h2"` with the existing `toLocaleDateString("en-US", …)`
   logic untouched, styled with `--brk-text-brand` and `--font-h2`.
4. **`SegmentedControl`** — `Day` / `Week` / `Month`, `aria-label="Calendar view"`, value bound
   to `currentView`. Emits `onChange` → `onViewChange`. `VIEW_LABELS` and `VIEWS` constants
   are unchanged, so no copy changes.
5. **Right cluster** — `ActionIcon` color-scheme toggle (`aria-label="Toggle color scheme"`)
   and the existing Settings `Button`. Settings copy preserved verbatim.

The header is `position: sticky` within the grid and uses `--brk-bg-raised` with
`--brk-shadow-sm` applied only when the calendar is scrolled (optional enhancement; not
required for correctness).

### `Sidebar` (new shape)

- `AppShell.Navbar` with `component="aside"` — **required**, preserves `TASK-02.spec.ts:82`.
- Sections become `Accordion` (`Tasks`, `Blocked Slots`, `Settings`) — copy preserved verbatim.
  `Accordion` replaces the `openSection: string | null` state with controlled/uncontrolled
  `Accordion` state; only one section open at a time is preserved via `multiple={false}`.
- Collapse toggle moves from an absolutely-positioned floating button to an `ActionIcon`
  inside the header (desktop) / inside the `Drawer` header (mobile), with
  `aria-label="Toggle sidebar"` and `aria-expanded`.
- When collapsed on desktop, the rail narrows to an icon-only 56px column showing section
  icons — this is the visual upgrade over the current `width: 0px` (which fully hides content
  and shifts nothing).
- `TaskPanel` and `BlockPanel` keep their existing props.

### `StatusBar` (new, replaces 4 stacked banners)

One row above the calendar, `role="status"`, `aria-live="polite"`. Priority order — only the
highest-priority state renders at a time:

| Priority | Condition | Copy (verbatim) | Mantine |
|---|---|---|---|
| 1 | `saveState.status === "error"` | `Error: {…}` | `Alert color="red"` |
| 2 | `calendarQuery.isError` | `Failed to sync calendar events: {…}` | `Alert color="red"` |
| 3 | `saveState.status === "success"` | `Schedule saved successfully!` | `Alert color="green"` |
| 4 | `calendarQuery.isLoading` | `Loading calendar events...` | `Skeleton` + `Text` |
| 5 | `isPreviewActive && failurePanelOpen` | (AllocationFailurePanel) | `Paper` |

This preserves all four asserted strings and their visibility semantics while eliminating the
layout shift of stacked banners. Each `Alert` keeps its inline **Retry** `Button`.

> Because the priority table makes these mutually exclusive where they were previously
> stackable, QA must confirm no test depends on two being visible at once. On inspection of
> both specs, none does.

### Responsive behaviour

| Range | Sidebar | Header |
|---|---|---|
| `< 36em` | `Drawer`, opened from a hamburger `ActionIcon` | Segmented control collapses to icons; date title drops to `--font-sm` |
| `36em–62em` | `Drawer`, overlay | Segmented control visible |
| `>= 62em` | Static rail, collapsible | Full header |
| `>= 75em` | 320px rail | Full header |

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| `AppShell` default padding breaks the FullCalendar height chain | Medium | High | `padding={0}` set explicitly; `height:100%` preserved on the calendar wrapper; verified in QA |
| `<aside>` lost → `TASK-02.spec.ts:82` fails | Medium | High | `component="aside"` is a hard requirement, called out in the code comment and the E2E register |
| `Tasks (N)` split across elements → strict-mode failure | Medium | High | Render as one contiguous string node; never wrap the count in a `Badge` |
| Accordion changes panel mount timing → drag source unavailable | Medium | Medium | Mantine 8.3.18 `AccordionPanel` has no `keepMounted` prop; expand Tasks before dragging and verify the drag source remains live while expanded |
| `AppShell` breakpoint mismatch with CSS breakpoints | Low | Medium | Both pinned to the three values above; single constant object exported from `theme/` |

---

## Open Questions

- Should the collapsed rail be icon-only (56px, proposed) or fully hidden (`0px`, current
  behaviour)? Default: icon-only, as it is the more usable modern pattern. Flagged for user
  confirmation; not blocking.
