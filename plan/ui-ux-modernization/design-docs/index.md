# Design Docs — UI/UX Modernization

Pipeline context: `ui-ux-modernization`
Created by: Solution Designer
Status: **Approved** (Option C — component library approach, dark-mode tier included, single delivery)

---

## Overview

The application renders its entire UI from ~100 inline `style={{}}` objects with hardcoded hex
values across ~25 TSX files. A token layer (`design-tokens.css`) and a FullCalendar override
sheet (`CalendarStyles.css`) already exist but are almost entirely bypassed, and the responsive
rules in `global.css` target class names that do not exist in any component. The result is a
flat interface whose calendar reads as a bare table.

The approved solution adopts **Mantine v8** as a component library, keeps `design-tokens.css`
as the single source of truth with new semantic light/dark tiers, rebuilds the app shell on
`AppShell`, and replaces the calendar's `!important` override sheet with a declarative
`--fc-*` variable layer plus a custom React toolbar and an `eventContent` renderer.

**FullCalendar is retained.** Its DOM is asserted directly by two Playwright specs that are
treated as immutable acceptance criteria.

---

## Design Decision List

| # | Document | Decision | Status |
|---|---|---|---|
| 01 | [design-01-library-selection.md](./design-01-library-selection.md) | Adopt **Mantine v8.3.18** over MUI v6 | Selected |
| 02 | [design-02-theme-token-architecture.md](./design-02-theme-token-architecture.md) | 3-tier tokens (primitive → semantic light → semantic dark); Mantine theme derived from primitives | Selected |
| 03 | [design-03-app-shell-layout.md](./design-03-app-shell-layout.md) | Mantine `AppShell` + `Drawer`; `SegmentedControl`; `StatusBar`; real responsive breakpoints | Selected |
| 04 | [design-04-calendar-ux.md](./design-04-calendar-ux.md) | `--fc-*` variable layer; custom React toolbar; `eventContent` replaces imperative DOM injection | Selected |
| 05 | [design-05-component-inventory.md](./design-05-component-inventory.md) | `packages/ui` becomes a thin brand wrapper layer over Mantine | Selected |
| 06 | [design-06-migration-plan.md](./design-06-migration-plan.md) | 7 sequential phases, 3 parallel lanes in Phase 6, explicit overlap flags | Selected |
| 07 | [design-07-risks-and-e2e-register.md](./design-07-risks-and-e2e-register.md) | 18 general risks + 9 copy assertions + 7 DOM assertions | Selected |

---

## Selected Options Summary

### Library

**`@mantine/core@8.3.18`** + `@mantine/hooks@8.3.18` + `@mantine/modals@8.3.18` +
`@mantine/notifications@8.3.18` + `@tabler/icons-react@3.48.0`.
Dev: `postcss`, `postcss-preset-mantine`, `postcss-simple-vars`.

Chosen over MUI v6 for four reasons:

1. **React 18 compatibility (hard constraint).** Mantine **9.x requires React 19.2+**
   (peer bumped to `^19.2.0` in 9.0.0, released 2026-03-31); this project is on React 18.2.
   Mantine 8 is the newest line supporting React 18. MUI v6 is worse — it is superseded by
   **two** majors (v9 shipped 2026-04-08).
2. **FullCalendar interop.** Mantine v7+ ships **static CSS**; FullCalendar v6 injects its own
   CSS at runtime. Static-then-runtime ordering is deterministic, so a scoped `--fc-*` block
   wins reliably without `!important`. Emotion's runtime injection is non-deterministic and
   would put the calendar work at risk.
3. **No styling runtime.** No Emotion, no SSR/`jsxImportSource` friction under Vite 5, and
   ~67 kB gzip vs ~112 kB for MUI on the same benchmark.
4. **Dark mode is first-class** via `data-mantine-color-scheme`, which maps directly onto the
   required `[data-theme]` semantic tier.

Explicitly rejected: `@mantine/schedule` (needs Mantine 9 + React 19.2, and replaces the
FullCalendar DOM the specs assert on), FullCalendar Premium (paid + different DOM),
MUI v6 (see above).

### Theme

`design-tokens.css` becomes Tier 1 primitives (existing `--clr-*` names and values preserved),
Tier 2 semantic light (`--brk-*`), Tier 3 semantic dark scoped to
`[data-mantine-color-scheme="dark"]`. Components reference only `--brk-*`. The Mantine `teal`
tuple is built from the existing brand hexes so `teal[4]=#3CC4BD`, `teal[5]=#2BA8A2`,
`teal[6]=#1E8C86`, `teal[0]=#E8F6F5` — no brand colour changes.

### Calendar

`CalendarStyles.css` is rewritten from 17 `!important` blocks into a `--fc-*` custom-property
layer on `.brk-calendar`, plus a small number of scoped rules. A new `CalendarToolbar`
replaces `headerToolbar={false}` with real React chrome (date title, prev/next, Today,
`SegmentedControl`, unsaved-changes badge, business-hours badge). An `eventContent` renderer
replaces the `document.createElement` delete button in `eventDidMount` and emits the currently
dead `brk-completed` / `is-past` classes. Adds sticky day headers and `businessHours` shading.

### Shell

`MainLayout` → `AppShell` (CSS grid, `padding={0}`), `Header` → Mantine primitives with
`aria-label`s added, `Sidebar` → `AppShell.Navbar component="aside"` + `Accordion`, and the
four stacked inline banners in `DashboardPage` collapse into one `StatusBar` with
`role="status"`.

---

## Reference Mapping

### Source files read to produce these designs

| File | Used for |
|---|---|
| `apps/web/package.json` | React 18.2, FullCalendar v6, dependency baseline |
| `apps/web/vite.config.ts` | 15-line config, `/api` proxy on 5173 |
| `apps/web/node_modules/vite/package.json` | Installed Vite 5.4.21 |
| `apps/web/src/main.tsx` | Provider tree, current stylesheet import order |
| `apps/web/src/styles/design-tokens.css` | Existing Tier-1 primitive inventory |
| `apps/web/src/styles/global.css` | Dead responsive selectors, `.sr-only` |
| `apps/web/src/styles/CalendarStyles.css` | 17 `!important` blocks; dead `.brk-completed` |
| `apps/web/src/components/MainLayout.tsx` | Flex shell, hardcoded `#EFF8F7` |
| `apps/web/src/components/Header.tsx` | Nav controls with no accessible names |
| `apps/web/src/components/Sidebar.tsx` | `<aside>` preservation, collapse toggle |
| `apps/web/src/components/CalendarView.tsx` | `headerToolbar={false}`, `eventDidMount` DOM injection, `classNames` |
| `apps/web/src/pages/DashboardPage.tsx` | Four inline banners, calendar prop wiring |
| `apps/web/src/features/tasks/TaskPanel.tsx` | `Tasks ({tasks.length})` copy node |
| `apps/web/src/features/tasks/TaskModal.tsx` | `New Task` / `Edit Task` copy node |
| `apps/web/src/features/preview/PreviewToolbar.tsx` | `Preview: {n} events` copy node |
| `apps/web/src/features/completion/ConflictDialog.tsx` | `Discard Preview — Start fresh` copy node |
| `packages/ui/package.json`, `packages/ui/src/index.ts` | 4 primitives, `tsc`-only build |
| `apps/web/e2e/TASK-01.spec.ts`, `TASK-02.spec.ts` | Immutable acceptance criteria |
| `apps/web/playwright.config.ts` | Chromium, 1440×900, port 5173 |

### Existing documentation

No architecture docs, ADRs, feature specs or `docs-catalog.md` exist under `.opencode/` at the
time of writing, so these designs are self-contained and reference no prior decisions.

### Templates

All documents follow `.opencode/template/design-template.md`
(Context → Options Considered → Decision → Impact → Risks → Open Questions), extended with
per-topic sections where the approved scope required them (token tables, calendar variable
specification, component API, migration phases, E2E register).