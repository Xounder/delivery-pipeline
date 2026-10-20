# Tasks Index — UI/UX Modernization

15 implementation tasks covering the approved 7-phase Mantine UI/UX modernization, dispatched to
`senior-frontend`. **No backend task exists.**

Source of truth: [`../design-docs/`](../design-docs/) and [`../planning/`](../planning/).
Structure of each task file: [`.opencode/template/task-template.md`](../../../template/task-template.md).

---

## Overview

| # | Task | Phase | Owner | Files | Gate |
| --- | --- | --- | --- | --- | --- |
| 1 | [TASK-01 — Dependencies, lockfile, PostCSS](./TASK-01-dependencies-and-postcss.md) | 1 | senior-frontend | 4 | install + `build apps/web` |
| 2 | [TASK-02 — Design tokens, global reset, breakpoints](./TASK-02-design-tokens-and-reset.md) | 2 | senior-frontend | 2 | `build apps/web` |
| 3 | [TASK-03 — Mantine theme and provider tree](./TASK-03-mantine-theme-and-providers.md) | 2 | senior-frontend | 5 | `build apps/web` |
| 4 | [TASK-04 — `packages/ui` Mantine wrappers](./TASK-04-packages-ui-wrappers.md) | 3 | senior-frontend | 7 | ui → web → e2e |
| 5 | [TASK-05 — AppShell layout, Header, Sidebar](./TASK-05-app-shell-layout.md) | 4 | senior-frontend | 3 | ui → web → e2e |
| 6 | [TASK-06 — `StatusBar` + `DashboardPage` split](./TASK-06-statusbar-and-dashboard-split.md) | 4 | senior-frontend | 7 | ui → web → e2e |
| 7 | [TASK-07 — `CalendarToolbar` + Header ownership](./TASK-07-calendar-toolbar-and-ownership.md) | 5 | senior-frontend | 4 | ui → web → e2e |
| 8 | [TASK-08 — `eventContent` renderer](./TASK-08-event-content-renderer.md) | 5 | senior-frontend | 2 | ui → web → e2e |
| 9 | [TASK-09 — Calendar stylesheet re-skin + partials](./TASK-09-calendar-stylesheet-reskin.md) | 5 | senior-frontend | 5 | ui → web → e2e |
| 10 | [TASK-10 — Lane 6-A: tasks + blocks](./TASK-10-lane-6a-tasks-and-blocks.md) | 6-A | senior-frontend | 6 | ui → web → e2e |
| 11 | [TASK-11 — Lane 6-B: preview + completion + shared](./TASK-11-lane-6b-preview-completion-shared.md) | 6-B | senior-frontend | 10 | ui → web → e2e |
| 12 | [TASK-12 — Lane 6-C: settings, dev, errors, login](./TASK-12-lane-6c-settings-dev-errors-login.md) | 6-C | senior-frontend | 7 | ui → web → e2e |
| 13 | [TASK-13 — Post-merge reconciliation and gate](./TASK-13-post-merge-validation.md) | 6 exit | senior-frontend | 1 report | ui → web → e2e, merged tree |
| 14 | [TASK-14 — Dead-CSS cleanup, final imports](./TASK-14-dead-css-cleanup.md) | 7 | senior-frontend | 7 | ui → web → e2e |
| 15 | [TASK-15 — Final line-count, contrast, bundle audit](./TASK-15-final-audit.md) | 7 | senior-frontend | 1 report | ui → web → e2e, final |

Supporting document: [`dependencies.md`](./dependencies.md) — full graph, overlap map, ownership table
and the 32-constraint coverage matrix.

---

## Execution Order

Phases 1–5 are **strictly sequential**; Phase 6 is the only parallel phase; Phase 7 runs after the
merge. This is forced by `P-7` and by eight of the ten overlap flags, which are cross-phase write
targets.

| Order | Task | What it delivers |
| --- | --- | --- |
| 1 | TASK-01 | Mantine `8.3.18` + Tabler + PostCSS installed; `postcss.config.cjs`; lockfile regenerated |
| 2 | TASK-02 | Three-tier `--brk-*` tokens, global reset in the right file, breakpoint values fixed |
| 3 | TASK-03 | `MantineProvider`, `ThemeProvider`, `ColorSchemeScript`, Mantine-first import order |
| 4 | TASK-04 | Four `packages/ui` wrappers rebuilt on Mantine with an unchanged public API |
| 5 | TASK-05 | `AppShell` with explicit bounded height, single `<aside>`, Mantine Header/Sidebar |
| 6 | TASK-06 | `StatusBar` absorbing all five status regions; 567-line `DashboardPage` split to ≤ 350 |
| 7 | TASK-07 | `CalendarToolbar` with real view-switcher buttons; Header hands over nav ownership |
| 8 | TASK-08 | React `eventContent` renderer replaces `eventDidMount` DOM mutation |
| 9 | TASK-09 | `--fc-*` layer, zero `!important`, CSS-only workday tint, three CSS partials |
| 10 | TASK-10 ∥ TASK-11 ∥ TASK-12 | 23 feature files migrated across three file-disjoint lanes |
| 11 | TASK-13 | Ten cross-lane reconciliation checks + the consolidated full gate |
| 12 | TASK-14 | Dead responsive/tokens/calendar rules removed; final import order |
| 13 | TASK-15 | Line-count, contrast, bundle delta and the final acceptance gate |

---

## Dependency Graph

```text
01 → 02 → 03 → 04 → 05 → 06 → 07 → 08 → 09
                                              │
                        ┌─────────────────────┼─────────────────────┐
                        ▼                     ▼                     ▼
                      10 (6-A)              11 (6-B)              12 (6-C)
                        └─────────────────────┼─────────────────────┘
                                              ▼
                                             13 → 14 → 15
```

No cycles. Full rationale per edge: [`dependencies.md`](./dependencies.md#task-dependency-graph).

---

## Ownership Mapping

**All 15 tasks are owned by `senior-frontend`.** The backend has no work: no API, schema, service or
migration change exists in this delivery (`apps/api/**`, `packages/backend/**`,
`packages/shared/**`, `packages/domain/**` and `packages/calendar/**` are on every task's forbidden
list).

### Exclusive-ownership assignments inside `senior-frontend`

| Area | Owning tasks | Note |
| --- | --- | --- |
| `apps/web/package.json`, `postcss.config.cjs`, `vite.config.ts` | TASK-01 | Only writer; `scripts` untouched (`P-4`) |
| `packages/ui/package.json` + 5 wrapper sources | TASK-04 | Only writer of `packages/ui/src` (`OVERLAP-9`) |
| `MainLayout.tsx`, `Sidebar.tsx` | TASK-05 | Not writable by any other task |
| `Header.tsx` | TASK-05, then TASK-07 | **Sequential**; TASK-05 must keep `MainLayoutProps` stable |
| `DashboardPage.tsx` | **TASK-06 for the entire delivery** | `OVERLAP-1`, `M9`, `P-8`. TASK-07 routes changes through the extracted hook; lanes 6-A/6-B/6-C are forbidden |
| `pages/dashboard/**`, `StatusBar.tsx` | TASK-06 | New needs become props on the extracted interfaces |
| `CalendarView.tsx`, `components/calendar/EventContent.tsx` | TASK-08 | TASK-07 may not add a `<FullCalendar>` prop |
| `components/calendar/CalendarToolbar.tsx`, `calendarTheme.ts` | TASK-07 | — |
| `styles/CalendarStyles.css`, `styles/calendar/**` | TASK-09, then TASK-14 | **Sequential** |
| `styles/design-tokens.css`, `styles/global.css` | TASK-02, then TASK-14 | **Sequential**; TASK-15 may correct one contrast value |
| `main.tsx` | TASK-03, TASK-09, TASK-14 | **Sequential, append-only** (`OVERLAP-2`) |
| `App.tsx` | TASK-12 | Single owner (`OVERLAP-8`) |
| `features/tasks/**`, `features/blocks/**` | TASK-10 | Lane 6-A exclusive |
| `features/preview/**`, `features/completion/**`, `features/shared/**` | TASK-11 | Lane 6-B exclusive |
| `SettingsModal`, Dev-mode, error, `LoginPage`, `App.tsx` | TASK-12 | Lane 6-C exclusive |

---

## File Overlaps

Ten files are written by more than one task. Every one of them sits in the **sequential** chain, so
the sequence itself is the merge strategy — no concurrent-writer arbitration is needed anywhere in
this delivery except the intra-phase lane discipline in Phase 6.

| Flag | File | Written by | Strategy |
| --- | --- | --- | --- |
| **OVERLAP-1** | `pages/DashboardPage.tsx` | TASK-06, TASK-07 | Single owner for the whole delivery; mandatory split in TASK-06; TASK-07 adds no new logic. Forbidden in all three lanes |
| **OVERLAP-2** | `main.tsx` | TASK-03, TASK-09, TASK-14 | Sequential append: providers + first six imports → calendar partials → duplicate removal only |
| **OVERLAP-3** | `apps/web/package.json` | TASK-01 | Resolved to a single owner; `scripts` never change |
| **OVERLAP-4** | `pnpm-lock.yaml` | TASK-01, TASK-04 | Regenerated only, never hand-edited; both writers are sequential |
| **OVERLAP-5** | `styles/design-tokens.css` | TASK-02, TASK-14, TASK-15* | Restructure → prune → contrast correction only |
| **OVERLAP-6** | `styles/global.css` | TASK-02, TASK-14 | Reset + breakpoints → dead-rule removal |
| **OVERLAP-7** | `styles/CalendarStyles.css` | TASK-09, TASK-14 | Re-skin + split → prune; never merged back (`M17`) |
| **OVERLAP-8** | `App.tsx` | TASK-12 | Resolved to a single owner; rebase, never revert |
| **OVERLAP-9** | `packages/ui/src/index.ts` | TASK-04 (write), TASK-11/12 (read) | Read-only for lanes; export lines unchanged |
| **OVERLAP-10** | `packages/ui/dist/**` | Generated | `build packages/ui` precedes every `apps/web` build |

\* `TASK-15` is conditional and may only correct a contrast value.

**Extra overlaps introduced by this decomposition** (all sequential):
`components/Header.tsx` (TASK-05 → TASK-07); the three `styles/calendar/*.css` partials
(TASK-09 → TASK-14); `pages/dashboard/**` (TASK-06 only).

---

## Parallelization Plan

| Batch | Tasks | Justification |
| --- | --- | --- |
| 1–9 | TASK-01 … TASK-09 (one at a time) | Each depends on the previous through a real data contract — installed packages, token values, theme, wrapper API, shell props, navigation hook, event class names, CSS selectors. `P-7` also mandates sequential execution for phases 1–5 |
| **10** | **TASK-10 ∥ TASK-11 ∥ TASK-12** | **The only parallel batch.** Three file-disjoint lanes (23 files), each with a binding forbidden-file list |
| 11 | TASK-13 | Needs all three lanes merged to run the consolidated gate |
| 12 | TASK-14 | Needs the merged tree validated before cleanup |
| 13 | TASK-15 | Needs cleanup done before the final audit |

### Batch 10 — the parallel phase

| Lane | Task | Files | Lines | Hard constraints |
| --- | --- | --- | --- | --- |
| 6-A | TASK-10 | `features/tasks/*` (3), `features/blocks/*` (3) | ≈758 | `B2` native select/input in `TaskModal`; `M7` single `Tasks (N)` text node; `M10` both card files |
| 6-B | TASK-11 | `features/preview/*` (5), `features/completion/*` (4), `features/shared/*` (1) | ≈865 | `B2` native controls in `CreateActionModal`; `M8` `ConflictDialog` unmounts; `M18` `features/completion/` paths |
| 6-C | TASK-12 | `components/{SettingsModal,DevModePanel,DevModeBadge,ErrorFallback,ErrorBoundary}`, `pages/LoginPage`, `App.tsx` | ≈633 | `M11` 400+ timezone options; `OVERLAP-8` rebase `App.tsx` |

**Conditions for a valid parallel dispatch**

1. Each lane's file list is exclusive. Editing a file outside it invalidates the dispatch.
2. Each lane passes `build packages/ui` → `build apps/web` → `test apps/web` individually.
3. Each lane records the dev-port state before its Playwright run (`M14`).
4. TASK-13 runs the full gate once more on the consolidated tree (`R-25`).
5. A merge conflict in batch 10 means a lane-boundary violation — re-check the forbidden lists.

### Why the other 12 tasks cannot be parallelised

- `main.tsx` is written by three tasks (`OVERLAP-2`).
- The dependency manifests and lockfile are written by two tasks (`OVERLAP-3`, `OVERLAP-4`).
- `design-tokens.css` and `global.css` are each written by two tasks (`OVERLAP-5`, `OVERLAP-6`).
- `CalendarStyles.css` plus three partials are written by two tasks (`OVERLAP-7`).
- `DashboardPage.tsx` has one owner by rule (`OVERLAP-1`, `M9`, `P-8`).
- `App.tsx` sits in phase 6 and needs Phase 2 as a precondition (`OVERLAP-8`).

**Maximum safe parallelism: 3 concurrent tasks.**

---

## Constraint Coverage

All **32** constraint IDs from [`../planning/constraints.md`](../planning/constraints.md) are bound
to at least one task: `B1`–`B4` (4), `M1`–`M18` (18), `P-1`–`P-10` (10). The full ID-by-ID matrix is
in [`dependencies.md`](./dependencies.md#constraint-coverage--all-32-ids).

| Category | IDs | Primary enforcing tasks |
| --- | --- | --- |
| Blocking | `B1` `B2` `B3` `B4` | TASK-04/05/07, TASK-10, TASK-11, TASK-02/03/09 |
| Material | `M1` … `M18` | TASK-02, TASK-03, TASK-05, TASK-06, TASK-07, TASK-08, TASK-09, TASK-10, TASK-11, TASK-12, TASK-14 |
| Process | `P-1` … `P-10` | Every task; the phase structure enforces `P-7` and `P-8` |

---

## Global Rules for Every Task

1. **Validation order:** `build packages/ui` → `build apps/web` → `test apps/web`. No lint gate exists,
   none may be added, none may be reported (`P-1`, `P-4`).
2. **Fresh dev server before every Playwright run.** `reuseExistingServer: true` makes a reused server
   an invalid result (`M14`).
3. **`apps/web/e2e/**` is immutable.** A failing pre-existing test means the implementation is wrong,
   not the test (`P-5`, `P-10`).
4. **Never modify tsconfig, scripts or Playwright config** to make anything pass (`P-6`).
5. **No file may exceed 400 lines**, including the calendar stylesheets and `DashboardPage.tsx`
   (`P-9`, `M9`, `M17`).
6. **No new visible copy** that duplicates the `Tasks (`, `Preview`, `New Task`, `Edit Task`,
   `Create in this slot`, `Loading calendar events`, `Schedule saved successfully` or
   `Failed to sync calendar events` strings (`R-23`, `M7`, `M8`).
7. **`pnpm install` after every manifest change**; never hand-edit `pnpm-lock.yaml` (`P-2`).
8. **Documentation and code comments in English.**
9. **Report using the standard agent response format** — `summary.changes`, `summary.validations`,
   `concerns`, `errors`.

---

## Baseline

Recorded before any task, for comparison in TASK-15.

| Metric | Baseline |
| --- | --- |
| `build packages/ui` | PASS |
| `build apps/web` | PASS — 158 modules transformed |
| JS raw / gzip | 524.63 kB / 156.62 kB |
| CSS raw / gzip | 6.40 kB / 1.75 kB |
| `test apps/web` | PASS — 9/9 |

Mantine is expected to increase the bundle. The delta is **reported, not enforced** — no budget
script exists and none may be added (`P-4`).
