# File Impact — UI/UX Modernization

Authoritative per-file inventory for the UI/UX modernization, verified against the repository on
2026-10-03. Line counts are measured, not estimated.

Use this file instead of the Phase 6 file list in `design-06`, which omits three files and gives
two incorrect paths (constraint `M10`, `M18`).

Phase and lane assignments are authoritative for dispatch. See
[`impact-analysis.md`](./impact-analysis.md) for overlap flags and forbidden-file lists.

---

## Apps — `apps/web`

### Entry, providers, pages

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/main.tsx` | 47 | 1, 2, 5, 7 | — | Add `MantineProvider` (+ modals/notifications); add Mantine + PostCSS stylesheet imports in the correct order; add calendar partial imports | **OVERLAP-2** |
| `src/App.tsx` | 57 | 2, 6 | 6-C | Dev-mode provider tree; login/dashboard switch | **OVERLAP-8** |
| `src/pages/DashboardPage.tsx` | **567** | 4, 5, 6 | single owner | StatusBar host (5 regions), calendar state, slot-click handler, modal state, preview orchestration. **Mandatory split** | **OVERLAP-1**, `M9`, `M15` |
| `src/pages/LoginPage.tsx` | 72 | 6 | 6-C | Mantine form controls | — |

### Styles

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/styles/design-tokens.css` | 84 | 2, 7 | — | Restructure into three tiers; **remove the `*`/`body` reset** (it belongs in `global.css`) | **OVERLAP-5**, `M2` |
| `src/styles/global.css` | 42 | 2, 7 | — | Receive the global reset; breakpoint media queries aligned with Mantine; drop the line-1 duplicate `@import "./design-tokens.css"`; prune dead responsive classes in Phase 7 | **OVERLAP-6**, `M2`, `M4` |
| `src/styles/CalendarStyles.css` | 181 | 5, 7 | — | Full re-skin against `.brk-calendar-google`; remove `!important`; **split into partials before it exceeds 400 lines** | **OVERLAP-7**, `M1`, `M6`, `M17` |

### Shell

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/components/MainLayout.tsx` | 57 | 4 | — | `AppShell` layout with **explicit bounded height** | `M3` |
| `src/components/Header.tsx` | 113 | 4 | — | Brand, date-range title, `Today`. Date-step + view switcher **move out** to `CalendarToolbar`; view switcher stays real `<button>`s | `B1`, `M15` |
| `src/components/Sidebar.tsx` | 109 | 4 | — | Mantine markup; exactly one `<aside>` instance at every viewport | `M7` |

### Calendar

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/components/CalendarView.tsx` | 373 | 5 | — | Replace `eventDidMount` DOM mutation with an `eventContent` renderer; add the `CalendarToolbar`; **do not** add `businessHours` | `M5`, `B3`, `M16` |

### Task feature

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/features/tasks/TaskPanel.tsx` | 60 | 6 | 6-A | Mantine panel. `Tasks ({tasks.length})` must stay **one contiguous text node**; mount exactly once | `M7` |
| `src/features/tasks/TaskModal.tsx` | 342 | 6 | 6-A | Mantine layout, but **native `<select>` nested in the "Duration (minutes)" `<label>`** and native `#task-name` input must be preserved | `B2`, `R-01` |
| `src/features/tasks/TaskCard.tsx` | 69 | 6 | 6-A | Mantine card | `M10` (missing from design-06) |

### Block feature

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/features/blocks/BlockPanel.tsx` | 54 | 6 | 6-A | Mantine panel | — |
| `src/features/blocks/BlockModal.tsx` | 168 | 6 | 6-A | Mantine modal; native selects preserved if any are asserted (none are) | — |
| `src/features/blocks/BlockCard.tsx` | 65 | 6 | 6-A | Mantine card | `M10` (missing from design-06) |

### Preview feature

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/features/preview/PreviewToolbar.tsx` | 75 | 6 | 6-B | Mantine toolbar; **sole owner of the visible `Preview` label** | `R-23` |
| `src/features/preview/SaveButton.tsx` | 50 | 6 | 6-B | Mantine button; copy `Save to Calendar` unchanged | — |
| `src/features/preview/CalculateWeekButton.tsx` | 49 | 6 | 6-B | Mantine button; copy `Calculate Week` unchanged | — |
| `src/features/preview/AllocationFailurePanel.tsx` | 64 | 6 | 6-B | Mantine panel; must **not** introduce the string `Preview` | `R-23` |
| `src/features/preview/DeleteEventDialog.tsx` | 70 | 6 | 6-B | Mantine modal | — |

### Completion feature

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/features/completion/ConflictDialog.tsx` | 86 | 6 | 6-B | Mantine modal with `keepMounted={false}`; must unmount when closed. Contains "preview" copy | `M8`, `R-02` |
| `src/features/completion/CompletionAction.tsx` | 42 | 6 | 6-B | Mantine action row | `M10` (missing from design-06) |
| `src/features/completion/TokenRetryModal.tsx` | 115 | 6 | 6-B | Mantine modal; consumes `@brkroutnxdle/ui` `Modal` | `M18` (**path is `features/completion/`**, not `components/`) |
| `src/features/completion/RefreshButton.tsx` | 40 | 6 | 6-B | Mantine button | `M18` (**path is `features/completion/`**, not `components/`) |

### Shared feature

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/features/shared/CreateActionModal.tsx` | 274 | 6 | 6-B | Mantine layout, but **`<input id="task-name">`, `<select id="task-duration">` and both submit buttons must be preserved**; `button[type="submit"]:has-text("Create")` must still resolve | `B2`, `R-01` |

### Misc components

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/components/SettingsModal.tsx` | 263 | 6 | 6-C | Mantine controls permitted here. 400+ timezone options — do not swap to a non-virtualized `Select` without filtering | `M11` |
| `src/components/DevModePanel.tsx` | 101 | 6 | 6-C | Mantine modal | — |
| `src/components/DevModeBadge.tsx` | 38 | 6 | 6-C | Mantine badge | — |
| `src/components/ErrorFallback.tsx` | 65 | 6 | 6-C | Mantine; consumes `@brkroutnxdle/ui` `Button` | — |
| `src/components/ErrorBoundary.tsx` | 37 | 6 | 6-C | Mantine boundary | — |

---

## Package — `packages/ui`

| File | Lines | Phase | Lane | Change | Flags |
| --- | --- | --- | --- | --- | --- |
| `src/index.ts` | 5 | 3 | — | Export surface; only Phase 3 may change it | **OVERLAP-9** |
| `src/components/Button.tsx` | — | 3 | — | Rebuild on Mantine; preserve `variant`/`size` prop names | `R-05` |
| `src/components/Modal.tsx` | — | 3 | — | Rebuild on Mantine; map `isOpen` → `opened` internally, keep `isOpen` public | `R-05` |
| `src/components/FAB.tsx` | — | 3 | — | Rebuild on Mantine. **No consumers** — optional cleanup | `R-14` |
| `src/components/BottomSheet.tsx` | — | 3 | — | Rebuild on Mantine. **No consumers** — optional cleanup | `R-14` |
| `package.json` | — | 1, 3 | — | Add `@mantine/core@8.3.18` as peer + dev | **OVERLAP-3** |
| `dist/**` | — | 3 | — | **Generated.** Never hand-edited; rebuild before every web build | **OVERLAP-10**, `R-05` |

Current exports: `Button`, `ButtonVariant`, `ButtonSize`, `Modal`, `FAB`, `BottomSheet`.
Only **two** app files consume this package: `ErrorFallback.tsx` (lane 6-C) and
`TokenRetryModal.tsx` (lane 6-B).

---

## Configuration and manifests

| File | Phase | Change | Flags |
| --- | --- | --- | --- |
| `apps/web/package.json` | 1 | Mantine + Tabler deps, PostCSS dev deps. **No `lint` script — do not add one** | **OVERLAP-3**, `P-4` |
| `apps/web/postcss.config.cjs` | 1 | **New file.** Must be `.cjs` because `apps/web` is `"type": "module"` | `P-3`, `R-10` |
| `pnpm-lock.yaml` | 1, 3 | Regenerated by `pnpm install` after every manifest change | **OVERLAP-4**, `P-2` |
| `apps/web/vite.config.ts` | 1 | Only if PostCSS wiring needs explicit config | `M13` |
| `apps/web/playwright.config.ts` | — | **Unchanged.** Note `reuseExistingServer: true`, `fullyParallel: false`, `retries: 1`, `1440x900` | `M14`, `R-11` |
| `apps/web/tsconfig.json` | — | **Unchanged.** Note `e2e` is excluded from the program | `M12`, `R-12`, `P-6` |
| `package.json` (root) | — | **Unchanged.** `pnpm -r build` is topological but targeted builds are not | `R-05` |

---

## Files that must NOT be modified

| Path | Reason |
| --- | --- |
| `apps/web/e2e/TASK-01.spec.ts` | Immutable acceptance criteria (`P-5`) |
| `apps/web/e2e/TASK-02.spec.ts` | Immutable acceptance criteria (`P-5`) |
| `apps/web/tsconfig.json` | Excluded from modification (`P-6`) |
| `apps/web/playwright.config.ts` | Excluded from modification (`P-6`) |
| `apps/web/package.json` scripts | No lint script may be added (`P-4`, `P-6`) |
| `packages/backend/**` | Out of scope |
| `packages/shared/**` | Out of scope |
| `packages/ui/dist/**` | Generated artifact (`OVERLAP-10`) |

---

## Totals

| Metric | Value |
| --- | --- |
| `apps/web` source files touched | 30 |
| Total `apps/web` lines in scope | ≈ 3,150 |
| New files created | 1 (`apps/web/postcss.config.cjs`) + calendar CSS partials (`M17`) |
| Files over 400 lines requiring a split | 1 (`DashboardPage.tsx`, 567) + 1 imminent (`CalendarStyles.css`, 181 → will exceed) |
| Phase 6 lane balance | 6-A ≈ 758 / 6-B ≈ 865 / 6-C ≈ 633 lines |
| Files with cross-phase overlap | 10 (`OVERLAP-1` … `OVERLAP-10`) |
| Files needing exclusive ownership | 1 (`DashboardPage.tsx`) |
| Files modified by exactly one phase 6 lane | 23 |
