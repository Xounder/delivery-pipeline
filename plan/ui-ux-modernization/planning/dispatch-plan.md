# Dispatch Plan — UI/UX Modernization

Parallel-dispatch specification: phase ordering, the definitive Phase 6 lane split, per-lane
ownership and forbidden-file lists, the `DashboardPage.tsx` single-owner rule, and the validation
gate for every phase.

Related: [`index.md`](./index.md) (overview),
[`impact-analysis.md`](./impact-analysis.md) (overlap flags + material corrections `M1`–`M18`),
[`file-impact.md`](./file-impact.md) (per-file inventory),
[`feasibility.md`](./feasibility.md) (blocking corrections `B1`–`B4`).

---

## Dispatch Order

| # | Phase | Parallelism | Depends on |
| --- | --- | --- | --- |
| 1 | Dependencies, lockfile, PostCSS pipeline | **strictly sequential** | — |
| 2 | Design tokens, theme, Mantine providers, dark mode | **strictly sequential** | 1 |
| 3 | `packages/ui` Mantine wrapper components | **strictly sequential** | 2 |
| 4 | App shell: AppShell, Header, Sidebar, StatusBar | **strictly sequential** | 3 |
| 5 | Calendar: tokens, `eventContent`, grid, weekend tint | **strictly sequential** | 4 |
| 6 | Feature component migration | **3 parallel lanes** | 5 |
| 7 | Dead-CSS cleanup, bundle check, final validation | **strictly sequential** | 6 merged |

**Phases 1–5 must not be parallelised.** Eight of the ten overlap flags
(`OVERLAP-2` … `OVERLAP-9`) are cross-phase write targets, so any concurrent dispatch inside
phases 1–5 produces merge conflicts in `main.tsx`, the package manifests, the lockfile or the
token stylesheets.

**Phase 6 is the only parallel phase. Phase 7 runs only after all three lanes merge.**

---

## Validation Gate (phases 3–7)

Run in this exact order, every time:

1. `run-package-command build packages/ui`
2. `run-package-command build apps/web`
3. `run-package-command test apps/web`

**Why step 1 is mandatory.** `packages/ui` is `tsc`-only and emits declarations to `dist`.
`apps/web` resolves `@brkroutnxdle/ui` through that `dist`. A targeted `apps/web` build does **not**
rebuild the package, so skipping step 1 makes `apps/web` typecheck against stale declarations and
report misleading errors (`R-05`).

**Playwright preconditions.** `apps/web/playwright.config.ts` sets
`webServer: pnpm dev` with **`reuseExistingServer: true`**. Before each run, confirm no stale dev
server is bound to the dev port. A reused server can make a broken build look green or a fixed bug
look broken. With `fullyParallel: false` and `retries: 1`, one contaminated run can block a phase
sign-off. A run on a reused server is **invalid** (`M14`, `R-11`).

**No lint gate.** `apps/web/package.json` has no `lint` script. Do not add one, do not run one, and
do not report a lint result. Validation is build + Playwright only (`P-4`).

**Phase 1/2 exception.** Phases 1 and 2 do not change `packages/ui`, so their gate is
`build apps/web` only (plus a successful `pnpm install` in Phase 1).

---

## Phase 1 Gate — Dependencies, lockfile, PostCSS

- `pnpm install` completes with no peer-dependency errors and updates `pnpm-lock.yaml` (`P-2`).
- `apps/web/postcss.config.cjs` exists and loads (`.cjs` is mandatory — `apps/web` is
  `"type": "module"`, so `postcss.config.js` would be parsed as ESM and fail) (`P-3`).
- `run-package-command build apps/web` → **PASS**.

## Phase 2 Gate — Tokens, theme, providers, dark mode

- `MantineProvider` present in `src/main.tsx`; colour-scheme source chosen and wired.
- Global reset present in `global.css` and absent from `design-tokens.css` (`M2`).
- Exactly **one** token import survives (`M2`).
- Mantine breakpoints and `global.css` media queries numerically identical (`M4`).
- Event + brand palette contrast-checked in both tiers (`B4`).
- `run-package-command build apps/web` → **PASS**.

## Phase 3 Gate — `packages/ui` wrappers

- Wrapper public API unchanged (`Button`/`ButtonVariant`/`ButtonSize`/`Modal`/`FAB`/`BottomSheet`;
  `Modal` keeps `isOpen`, not Mantine's `opened`).
- `@mantine/core@8.3.18` added as **peer + dev**, not a hard runtime dependency.
- `run-package-command build packages/ui` → **PASS**, **then**
  `run-package-command build apps/web` → **PASS**.

## Phase 4 Gate — App shell

- AppShell root has an **explicit bounded height** (`M3`).
- Exactly one `<aside>` and exactly one `Tasks (N)` panel at every viewport (`M7`).
- View switcher renders real `<button>`s labelled `Day` / `Week` / `Month` (`B1`).
- All **five** status regions folded into `StatusBar`; Header and CalendarToolbar ownership
  respected (`M15`).
- `DashboardPage.tsx` split, under 400 lines, with stable extracted prop interfaces (`M9`).
- Full gate (ui → web → e2e) → **PASS**.

## Phase 5 Gate — Calendar

- `CalendarStyles.css` scoped to `.brk-calendar-google` (`M1`).
- No `businessHours` prop on the FullCalendar instance (`B3`).
- `eventContent` renderer covers dayGrid + timeGrid and all event kinds; title stays inside
  `.fc-event`; no `.fc-event-title` / `.fc-event-time` selectors remain (`M5`).
- Calendar stylesheet split into partials before exceeding 400 lines (`M17`).
- `.fc-timegrid-slots`, `.fc-timegrid-body`, `.fc-timegrid-slot` (`14:00`) all still present.
- Full gate (ui → web → e2e) → **PASS**.

## Phase 6 Gate — Feature migration

- Each lane passes the full gate **individually**.
- After all three lanes merge, the full gate runs **once more** on the consolidated tree
  (`R-25`).
- All three lanes' forbidden-file lists respected (below).

## Phase 7 Gate — Cleanup and final validation

- Dead responsive rules removed from `global.css`.
- Bundle delta recorded against the 524.63 kB / 156.62 kB baseline (reported, not enforced — no
  budget script exists).
- No source file above 400 lines anywhere in `apps/web` or `packages/ui`.
- Full gate (ui → web → e2e) → **PASS** on a fresh dev server.

---

## `DashboardPage.tsx` Single-Owner Rule

`apps/web/src/pages/DashboardPage.tsx` is **567 lines** — already over the project's 400-line rule —
and is required by phases 4, 5 **and** 6. It has **exactly one owning lane for the entire
delivery**. Two lanes must never edit it concurrently (`M9`, `P-8`).

**Recommended arrangement**

1. **Phase 4 owner performs the mandatory split.** Extract focused children with stable prop
   interfaces: the five status regions feeding `StatusBar`, calendar view state + view switching,
   the preview orchestration handlers, the slot-click handler, and modal state ownership.
2. **Phase 5 and Phase 6 route changes through those interfaces** rather than editing
   `DashboardPage.tsx` directly. New needs become new props or new extracted children owned by the
   Phase 4 owner.
3. If the Tech Lead prefers strict lane isolation, designate a single "DashboardPage owner" lane
   that holds the file exclusively across phases 4–7 and gate every other lane's dashboard changes
   behind it.

**Forbidden:** concurrent writes to `DashboardPage.tsx`; leaving it above 400 lines after Phase 4;
letting Phase 5 or 6 re-inline extracted logic to "avoid touching" it.

---

## Phase 6 — Definitive Lane Split

Three lanes, **file-disjoint**, run concurrently after Phase 5. Each lane owns its files
exclusively. The forbidden lists are binding — a lane that edits a file outside its list invalidates
the parallel dispatch and its gate result.

Per-file line counts: [`file-impact.md`](./file-impact.md).

### Lane 6-A — Tasks + Blocks (≈758 lines, 6 files)

| File | Lines |
| --- | --- |
| `apps/web/src/features/tasks/TaskPanel.tsx` | 60 |
| `apps/web/src/features/tasks/TaskModal.tsx` | 342 |
| `apps/web/src/features/tasks/TaskCard.tsx` | 69 |
| `apps/web/src/features/blocks/BlockPanel.tsx` | 54 |
| `apps/web/src/features/blocks/BlockModal.tsx` | 168 |
| `apps/web/src/features/blocks/BlockCard.tsx` | 65 |

**Lane constraints**

- `B2` — `TaskModal` **must** keep a native `<select>` nested inside its
  `<label>` (the "Duration (minutes)" control) and a native input for `#task-name`. Mantine
  `Select` / `NumberInput` / `TextInput` are forbidden in this file.
- `M7` / `R-23` — `TaskPanel` mounts exactly once; `Tasks ({tasks.length})` stays a **single
  contiguous text node**; no new visible copy containing `Tasks (`.
- `M10` — `TaskCard.tsx` and `BlockCard.tsx` are in scope (absent from design-06).

**Forbidden for 6-A:** `features/preview/**`, `features/completion/**`, `features/shared/**`,
`components/**`, `pages/**`, `styles/**`.

### Lane 6-B — Preview + Completion + Shared (≈865 lines, 10 files)

| File | Lines |
| --- | --- |
| `apps/web/src/features/preview/PreviewToolbar.tsx` | 75 |
| `apps/web/src/features/preview/SaveButton.tsx` | 50 |
| `apps/web/src/features/preview/CalculateWeekButton.tsx` | 49 |
| `apps/web/src/features/preview/AllocationFailurePanel.tsx` | 64 |
| `apps/web/src/features/preview/DeleteEventDialog.tsx` | 70 |
| `apps/web/src/features/completion/ConflictDialog.tsx` | 86 |
| `apps/web/src/features/completion/CompletionAction.tsx` | 42 |
| `apps/web/src/features/completion/TokenRetryModal.tsx` | 115 |
| `apps/web/src/features/completion/RefreshButton.tsx` | 40 |
| `apps/web/src/features/shared/CreateActionModal.tsx` | 274 |

**Lane constraints**

- `B2` — `CreateActionModal` **must** keep `<input id="task-name">`,
  `<select id="task-duration">`, `button[type="submit"]:has-text("Create Task")` and
  `button[type="submit"]:has-text("Create")`. Only one of the two create buttons may be visible at
  a time, as today.
- `M8` / `R-02` / `R-23` — `ConflictDialog` **must** unmount when closed
  (`keepMounted={false}`); it contains "preview" copy that must not remain in the DOM.
  `PreviewToolbar.tsx` is the sole owner of the visible `Preview` label; no other file in this
  lane may introduce that string.
- `M18` — `TokenRetryModal.tsx` and `RefreshButton.tsx` live in `features/completion/`, **not**
  `components/`.
- `M10` — `CompletionAction.tsx` is in scope.
- `OVERLAP-9` — `packages/ui/src/index.ts` is **read-only** for this lane.

**Forbidden for 6-B:** `features/tasks/**`, `features/blocks/**`, `components/**`, `pages/**`,
`styles/**`.

### Lane 6-C — Settings + Dev mode + Errors + Login (≈633 lines, 7 files)

| File | Lines |
| --- | --- |
| `apps/web/src/components/SettingsModal.tsx` | 263 |
| `apps/web/src/components/DevModePanel.tsx` | 101 |
| `apps/web/src/components/DevModeBadge.tsx` | 38 |
| `apps/web/src/components/ErrorFallback.tsx` | 65 |
| `apps/web/src/components/ErrorBoundary.tsx` | 37 |
| `apps/web/src/pages/LoginPage.tsx` | 72 |
| `apps/web/src/App.tsx` | 57 |

**Lane constraints**

- `M11` — `SettingsModal` builds 400+ timezone options from
  `Intl.supportedValuesOf("timeZone")`. Mantine `Select` is **not** virtualized: keep the native
  `<select>` for the timezone field or add search filtering before switching. Mantine inputs are
  otherwise permitted here (no spec asserts `SettingsModal`).
- `OVERLAP-8` — `App.tsx` may already carry Phase 2 provider changes. **Rebase; do not revert
  them.**
- `OVERLAP-9` — `packages/ui/src/index.ts` is **read-only** for this lane.
- `ErrorFallback.tsx` and `TokenRetryModal.tsx` are the **only two** consumers of
  `@brkroutnxdle/ui`, so this lane's usage must stay compatible with the Phase 3 wrapper API.

**Forbidden for 6-C:** `features/**`, `components/CalendarView.tsx`, `components/MainLayout.tsx`,
`components/Header.tsx`, `components/Sidebar.tsx`, `styles/**`.

### Lane merge rule

1. Each lane passes `build packages/ui` → `build apps/web` → `test apps/web` individually.
2. Lanes 6-A, 6-B and 6-C touch **zero** shared files, so the merge is expected to be conflict-free.
3. After all three merge, run the full gate **once more** on the consolidated tree before Phase 7
   starts (`R-25`).
4. If a merge conflict does occur, the conflict is a lane-boundary violation — re-check the
   forbidden lists before proceeding.

---

## Dispatch Checklist

- [ ] Phases 1–5 dispatched strictly in order, one phase at a time.
- [ ] `pnpm install` run after every dependency-manifest change; `pnpm-lock.yaml` committed each time.
- [ ] `apps/web/postcss.config.cjs` created (not `.js`).
- [ ] No lint gate attempted at any phase.
- [ ] `DashboardPage.tsx` owner named before Phase 4 starts.
- [ ] Phase 4 completes the `DashboardPage.tsx` split before Phase 5 begins.
- [ ] Phase 6 dispatched as exactly 3 lanes with the forbidden lists attached to each task.
- [ ] Dev port confirmed free before every `test apps/web`.
- [ ] Consolidated post-merge gate run before Phase 7.
- [ ] `apps/web/e2e/**` untouched throughout (`P-5`).
