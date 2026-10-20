# Constraint Index — UI/UX Modernization

Canonical cross-phase index of all mandatory constraints discovered while validating the approved
design against the actual repository.

**Full actionable text:**

- `B1`–`B4` → [`feasibility.md`](./feasibility.md#blocking-corrections-b1b4)
- `M1`–`M18` → [`impact-analysis.md`](./impact-analysis.md#material-corrections-m1m18)

These IDs are stable. The Tech Lead should reference them directly from task files, e.g.
"Task 4.2 MUST satisfy `B2`".

---

## Blocking Constraints (must be built in from the first task)

| ID | Title | Phase | Enforced at | Risk |
| --- | --- | --- | --- | --- |
| `B1` | No Mantine `SegmentedControl` — Day/Week/Month must be real `<button>`s | 4, 5 | `build` + `test apps/web` | `R-01` |
| `B2` | Native `<select>` / `<input>` preserved in `TaskModal` + `CreateActionModal` | 6-A, 6-B | `test apps/web` | `R-01` |
| `B3` | Defer FullCalendar `businessHours`; workday tint via CSS only | 5 | `test apps/web` | `R-06` |
| `B4` | Fix dark event contrast — `#3CC4BD` + white text fails (≈2.1:1) | 2, 5 | review + `test apps/web` | `R-08` |

---

## Material Constraints

| ID | Title | Phase | Enforced at | Risk |
| --- | --- | --- | --- | --- |
| `M1` | Calendar CSS must target `.brk-calendar-google`, not `.brk-calendar` | 5 | review | `R-09` |
| `M2` | Global reset is in `design-tokens.css`; tokens are imported **twice** | 2, 7 | review | `R-10` |
| `M3` | AppShell root needs an explicit bounded height (`min-height` default breaks calendar) | 4 | `test apps/web` | `R-04` |
| `M4` | Mantine `sm` = `48em`, design says `36em` — align theme + PostCSS media queries | 2, 4 | review + `test apps/web` | `R-07` |
| `M5` | `eventContent` replaces FullCalendar internals — update title/time selectors, cover dayGrid + all event kinds | 5 | `test apps/web` | `R-03` |
| `M6` | FullCalendar injects CSS at runtime — theme via custom properties, not `!important` | 5 | review | `R-09` |
| `M7` | Exactly one `<aside>` and one `Tasks (N)` panel at every viewport | 4 | `test apps/web` | `R-02` |
| `M8` | `ConflictDialog` must unmount when closed (`keepMounted={false}`) | 6-B | `test apps/web` | `R-02` |
| `M9` | Split `DashboardPage.tsx` (567 lines); **single owner** across phases 4/5/6 | 4, 5, 6 | dispatch | `R-13` |
| `M10` | design-06 omits `TaskCard.tsx`, `BlockCard.tsx`, `CompletionAction.tsx` | 6 | review | `R-14` |
| `M11` | `SettingsModal` renders 400+ timezone options; Mantine `Select` is not virtualized | 6-C | review | `R-15` |
| `M12` | `apps/web/tsconfig.json` excludes `e2e` — the build cannot detect selector drift | all | dispatch | `R-12` |
| `M13` | PostCSS pipeline belongs to `apps/web`; import Mantine styles first | 1, 2 | `build apps/web` | `R-10` |
| `M14` | `reuseExistingServer: true` — a stale dev server invalidates a Playwright run | all | `test apps/web` | `R-11` |
| `M15` | **Five** status regions (not four); CalendarToolbar owns date-nav + view switcher | 4 | `test apps/web` | `R-16` |
| `M16` | FullCalendar already implements sticky day headers — do not add competing CSS | 5 | review | `R-17` |
| `M17` | `CalendarStyles.css` (181 lines) will breach 400 lines — plan the split up front | 5 | review | `R-18` |
| `M18` | `TokenRetryModal.tsx` / `RefreshButton.tsx` live in `features/completion/`, not `components/` | 6-B | review | `R-14` |

---

## Process Constraints (apply to every phase)

| ID | Rule |
| --- | --- |
| `P-1` | Validation order is always `build packages/ui` → `build apps/web` → `test apps/web` |
| `P-2` | `pnpm install` must be run after **every** dependency-manifest change to regenerate `pnpm-lock.yaml` |
| `P-3` | The PostCSS config must be `apps/web/postcss.config.cjs` (`.js` fails under `"type": "module"`) |
| `P-4` | **No lint gate.** `apps/web` has no `lint` script; do not add one and do not report lint results |
| `P-5` | Never modify `apps/web/e2e/TASK-01.spec.ts` or `TASK-02.spec.ts` |
| `P-6` | Never modify tsconfig / eslint / vitest / package scripts to make an implementation or test pass |
| `P-7` | Phases 1–5 are strictly sequential; phase 6 is the only parallel phase |
| `P-8` | `DashboardPage.tsx` has one owner for the whole delivery; never two concurrent writers |
| `P-9` | No file may exceed 400 lines — including the calendar stylesheet (`M17`) and `DashboardPage.tsx` (`M9`) |
| `P-10` | If a pre-existing test fails, fix the implementation; the test is the acceptance criterion |

---

## Coverage Check

| Category | Count | IDs |
| --- | --- | --- |
| Blocking | 4 | `B1` `B2` `B3` `B4` |
| Material | 18 | `M1` … `M18` |
| Process | 10 | `P-1` … `P-10` |
| **Total** | **32** | all documented in this planning set |

Every ID appears in full actionable text in `feasibility.md` (blocking) or `impact-analysis.md`
(material), and every ID has a corresponding entry in [`risk-register.md`](./risk-register.md).
