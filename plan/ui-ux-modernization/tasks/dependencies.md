# Dependencies — UI/UX Modernization

Task dependency graph, definitive execution order, file-ownership map and constraint coverage for the
15 tasks in this folder.

Related: [`index.md`](./index.md) (overview, parallelization plan),
[`../planning/dispatch-plan.md`](../planning/dispatch-plan.md) (phase ordering and lane split),
[`../planning/impact-analysis.md`](../planning/impact-analysis.md) (`OVERLAP-1` … `OVERLAP-10`).

---

## Task Dependency Graph

```text
TASK-01  Dependencies, lockfile, PostCSS
   │
   ▼
TASK-02  Design tokens, global reset, breakpoints
   │
   ▼
TASK-03  Mantine theme + provider tree
   │
   ▼
TASK-04  packages/ui wrappers
   │
   ▼
TASK-05  AppShell layout, Header, Sidebar
   │
   ▼
TASK-06  StatusBar + DashboardPage split
   │
   ▼
TASK-07  CalendarToolbar + Header ownership move
   │
   ▼
TASK-08  eventContent renderer
   │
   ▼
TASK-09  Calendar stylesheet re-skin + partials
   │
   ├──────────────┬──────────────┐
   ▼              ▼              ▼
TASK-10        TASK-11        TASK-12
lane 6-A        lane 6-B       lane 6-C
tasks+blocks    preview+compl.  settings+dev+login
   │              │              │
   └──────────────┼──────────────┘
                  ▼
              TASK-13  Post-merge reconciliation + consolidated gate
                  │
                  ▼
              TASK-14  Dead-CSS cleanup + final import order
                  │
                  ▼
              TASK-15  Line-count, contrast, bundle + final gate
```

### No cycles

Every edge points forward in task ID order. `TASK-13`, `TASK-14` and `TASK-15` have no outgoing
edges.

### Required dependency table

| Task | Depends on | Reason the dependency is unavoidable |
| --- | --- | --- |
| TASK-01 | — | Entry point; nothing can import Mantine first |
| TASK-02 | TASK-01 | Mantine packages must exist before tokens pin `--mantine-color-*` |
| TASK-03 | TASK-02 | Theme derives from Tier-1 primitives; breakpoints must match (`M4`) |
| TASK-04 | TASK-03 | Wrappers sit on the theme; consumers compile only after this |
| TASK-05 | TASK-04 | Shell may consume `@brkroutnxdle/ui` |
| TASK-06 | TASK-05 | Rewrites `DashboardPage.tsx`, which still calls `MainLayout` unchanged |
| TASK-07 | TASK-06 | Toolbar is rendered from `DashboardPage`; navigation hook must exist first |
| TASK-08 | TASK-07 | Grid internals change only after control ownership is settled |
| TASK-09 | TASK-08 | Stylesheet targets the class names `eventContent` emits |
| TASK-10 | TASK-09 | Phase 6 starts after Phase 5 merges |
| TASK-11 | TASK-09 | Phase 6 starts after Phase 5 merges |
| TASK-12 | TASK-09 | Phase 6 starts after Phase 5 merges |
| TASK-13 | TASK-10, TASK-11, TASK-12 | Consolidated gate requires all three lanes merged |
| TASK-14 | TASK-13 | Cleanup must not run before the merged tree is validated |
| TASK-15 | TASK-14 | The audit must run on the cleaned tree |

### Deliberate non-dependencies

| Pair | Why there is no dependency |
| --- | --- |
| TASK-10 ∥ TASK-11 | File-disjoint (`features/tasks` + `features/blocks` vs `features/preview` + `features/completion` + `features/shared`) |
| TASK-10 ∥ TASK-12 | File-disjoint (feature lane vs component/page lane) |
| TASK-11 ∥ TASK-12 | File-disjoint |
| TASK-01 ∥ TASK-04 | Both touch `pnpm-lock.yaml`, but they sit in the same sequential chain; no parallelism is required |
| TASK-03 ∥ TASK-09 | Both touch `main.tsx`, but TASK-09 styles files TASK-08 creates, which requires the provider tree to exist |

---

## Ownership Map

| Task | Owner | Phase | Files | Gate |
| --- | --- | --- | --- | --- |
| TASK-01 | senior-frontend | 1 | 4 | install + `build apps/web` |
| TASK-02 | senior-frontend | 2 | 2 | `build apps/web` |
| TASK-03 | senior-frontend | 2 | 5 | `build apps/web` |
| TASK-04 | senior-frontend | 3 | 7 | ui → web → e2e |
| TASK-05 | senior-frontend | 4 | 3 | ui → web → e2e |
| TASK-06 | senior-frontend | 4 | 7 | ui → web → e2e |
| TASK-07 | senior-frontend | 5 | 4 | ui → web → e2e |
| TASK-08 | senior-frontend | 5 | 2 | ui → web → e2e |
| TASK-09 | senior-frontend | 5 | 5 | ui → web → e2e |
| TASK-10 | senior-frontend | 6-A | 6 | ui → web → e2e |
| TASK-11 | senior-frontend | 6-B | 10 | ui → web → e2e |
| TASK-12 | senior-frontend | 6-C | 7 | ui → web → e2e |
| TASK-13 | senior-frontend | 6 exit | 1 (report) | ui → web → e2e on the merged tree |
| TASK-14 | senior-frontend | 7 | 7 | ui → web → e2e |
| TASK-15 | senior-frontend | 7 | 1 (report) | ui → web → e2e, final |

**Backend:** none. No task touches `apps/api/**`, `packages/backend/**`, `packages/shared/**`,
`packages/domain/**` or `packages/calendar/**`.

### Totals

| Metric | Value |
| --- | --- |
| Tasks | 15 |
| File slots across all tasks | 71 |
| Unique source files | 58 |
| New source files | 16 |
| Generated artifacts (`pnpm-lock.yaml`, `packages/ui/dist/**`) | 2 |
| Validation reports created by TASK-13 / TASK-15 | 2 |
| Files over 400 lines requiring a split | 1 (`DashboardPage.tsx`, 567) |

---

## File Overlaps

Every file written by more than one task. All of them sit in the **sequential** chain, so none of
these overlaps requires concurrent-writer arbitration — the sequence itself is the merge strategy.

| # | File | Written by | Flag | Strategy |
| --- | --- | --- | --- | --- |
| 1 | `apps/web/src/pages/DashboardPage.tsx` | TASK-06, TASK-07 | **OVERLAP-1** | **Sequential, single owner.** TASK-06's agent owns it for the whole delivery and performs the mandatory split; TASK-07 routes its toolbar mount through the extracted hook. Lanes 6-A/6-B/6-C have it on their forbidden lists (`P-8`) |
| 2 | `apps/web/src/main.tsx` | TASK-03, TASK-09, TASK-14 | **OVERLAP-2** | **Sequential, append-only.** TASK-03 writes the provider tree and the first six imports; TASK-09 appends the three calendar partials; TASK-14 removes duplicates only. Each step states the expected full order |
| 3 | `apps/web/package.json` | TASK-01 | **OVERLAP-3** (resolved) | **Single owner.** Only TASK-01 writes the app manifest; no `scripts` change, no `lint` script |
| 4 | `pnpm-lock.yaml` | TASK-01, TASK-04 | **OVERLAP-4** | **Regenerated only.** Never hand-edited; re-run `pnpm install` after each manifest change. Both writers are in the sequential chain |
| 5 | `apps/web/src/styles/design-tokens.css` | TASK-02, TASK-14, TASK-15 (conditional) | **OVERLAP-5** | **Sequential.** TASK-02 restructures; TASK-14 prunes dead declarations; TASK-15 may correct a contrast value only |
| 6 | `apps/web/src/styles/global.css` | TASK-02, TASK-14 | **OVERLAP-6** | **Sequential.** TASK-02 receives the reset and writes breakpoint queries; TASK-14 removes dead responsive rules |
| 7 | `apps/web/src/styles/CalendarStyles.css` | TASK-09, TASK-14 | **OVERLAP-7** | **Sequential.** TASK-09 rewrites and splits per `M17`; TASK-14 prunes superseded rules |
| 8 | `apps/web/src/App.tsx` | TASK-12 | **OVERLAP-8** (resolved) | **Single owner.** The provider tree lives in `main.tsx` (TASK-03), so TASK-12 is the only writer. If a Phase 2 change ever lands here, TASK-12 rebases — it never reverts |
| 9 | `packages/ui/src/index.ts` | TASK-04 (write); TASK-11, TASK-12 (read) | **OVERLAP-9** | **Read-only for lanes.** Only TASK-04 writes; the five export lines stay identical |
| 10 | `packages/ui/dist/**` | TASK-04 build + every gate | **OVERLAP-10** | **Generated.** Never hand-edited; `build packages/ui` precedes every `apps/web` build |

### Additional overlaps introduced by this decomposition

| File | Written by | Strategy |
| --- | --- | --- |
| `apps/web/src/components/Header.tsx` | TASK-05, TASK-07 | **Sequential.** TASK-05 migrates the markup and keeps all controls; TASK-07 removes the date-step and view switcher. TASK-05 must not change `MainLayoutProps`, because TASK-07 is forbidden from editing `MainLayout` |
| `apps/web/src/styles/calendar/calendar-tokens.css`, `calendar-events.css`, `calendar-dark.css` | TASK-09, TASK-14 | **Sequential.** TASK-09 creates; TASK-14 prunes. The split is never merged back (`M17`) |
| `apps/web/src/pages/dashboard/**` | TASK-06 | **Single owner.** Lanes 6-A/6-B/6-C route new dashboard needs through TASK-06's extracted interfaces instead of adding files here |
| `apps/web/src/theme/breakpoints.ts` | TASK-03 (write); TASK-02, TASK-14 (consume) | **Consistency dependency, not a conflict.** TASK-02 writes the media queries first; TASK-03 formalises the values; TASK-14 re-verifies numeric equality (`M4`) |

### Refinements against the planning inventory

| Planning statement | This decomposition |
| --- | --- |
| `file-impact.md` assigns `CalendarView.tsx` both the `eventContent` renderer **and** the `CalendarToolbar` | Split: TASK-08 owns `CalendarView.tsx`; TASK-07 owns `components/calendar/CalendarToolbar.tsx` and mounts it from `DashboardPage`. TASK-07 is explicitly forbidden from adding a prop to `<FullCalendar>` |
| `dispatch-plan.md` lists `StatusBar` under Phase 4 | Split into TASK-05 (shell) and TASK-06 (StatusBar + the mandatory `DashboardPage` split), because those are the same file and the same ownership |
| Phase 2 "may add providers" to `App.tsx` | Not needed — the provider tree mounts in `main.tsx` (TASK-03), so `App.tsx` keeps a single owner (TASK-12) |

---

## Parallelization Plan

| Batch | Tasks | Justification |
| --- | --- | --- |
| 1 | TASK-01 | Entry point; unblocks everything |
| 2 | TASK-02 | Needs the Mantine packages installed |
| 3 | TASK-03 | Needs the token tiers; owns `main.tsx` providers |
| 4 | TASK-04 | Needs the theme; own lockfile write |
| 5 | TASK-05 | Needs the wrappers available |
| 6 | TASK-06 | Same file as TASK-05 (`DashboardPage.tsx`); also `StatusBar` |
| 7 | TASK-07 | Same file as TASK-06; needs the navigation hook |
| 8 | TASK-08 | Needs control ownership settled before grid internals change |
| 9 | TASK-09 | Styles the class names TASK-08 emits |
| 10 | **TASK-10 ∥ TASK-11 ∥ TASK-12** | **The only parallel batch.** Three file-disjoint lanes, forbidden lists attached, zero shared writes |
| 11 | TASK-13 | Requires all three lanes merged; consolidated gate |
| 12 | TASK-14 | Requires the merged tree validated |
| 13 | TASK-15 | Requires cleanup done |

**Maximum parallelism achieved: 3 concurrent tasks (batch 10).** Every other batch is a single task,
forced by a real dependency or by one of the ten overlap flags — eight of the ten are cross-phase
targets in `main.tsx`, the manifests, the lockfile or the token stylesheets.

### Parallel execution requirements for batch 10

1. Each lane's file list is exclusive; a lane editing any file outside it invalidates the dispatch.
2. Each lane passes `build packages/ui` → `build apps/web` → `test apps/web` **individually**.
3. Each lane records the dev-port state before its Playwright run (`M14`).
4. TASK-13 runs the full gate once more on the consolidated tree (`R-25`).
5. A merge conflict in batch 10 is a lane-boundary violation, not a Git problem — re-check the
   forbidden lists before retrying.

---

## Validation Order (all tasks)

1. `run-package-command build packages/ui` — mandatory before `apps/web`; `apps/web` resolves
   `@brkroutnxdle/ui` through `dist` and a targeted build does not rebuild it (`P-1`, `R-05`).
2. `run-package-command build apps/web`
3. `run-package-command test apps/web` — on a **fresh** dev server; `reuseExistingServer: true` makes a
   reused server invalid (`M14`).
4. **No lint step exists and none may be added or reported** (`P-4`).

TASK-01 and TASK-02 omit step 1 only because they change nothing in `packages/ui`.

---

## Constraint Coverage — all 32 IDs

| ID | Short title | Tasks that enforce it |
| --- | --- | --- |
| `B1` | No `SegmentedControl`; Day/Week/Month are real `<button>`s | TASK-04, TASK-05, TASK-07, TASK-15 |
| `B2` | Native `<select>` / `<input>` preserved | TASK-04, TASK-10, TASK-11, TASK-15 |
| `B3` | Defer `businessHours`; workday tint via CSS | TASK-07, TASK-08, TASK-09, TASK-14, TASK-15 |
| `B4` | Fix dark event contrast (no `#3CC4BD` + white) | TASK-02, TASK-03, TASK-09, TASK-15 |
| `M1` | Calendar CSS targets `.brk-calendar-google` | TASK-09, TASK-13, TASK-15 |
| `M2` | Reset in `global.css`; single token import | TASK-02, TASK-14, TASK-15 |
| `M3` | AppShell needs explicit bounded height | TASK-05 |
| `M4` | Theme breakpoints == `global.css` media queries | TASK-02, TASK-03, TASK-14, TASK-15 |
| `M5` | `eventContent` replaces FullCalendar internals | TASK-08, TASK-09, TASK-13, TASK-15 |
| `M6` | Theme via custom properties, no `!important` | TASK-09, TASK-14, TASK-15 |
| `M7` | Exactly one `<aside>` and one `Tasks (N)` panel | TASK-05, TASK-10, TASK-13, TASK-15 |
| `M8` | `ConflictDialog` unmounts when closed | TASK-11, TASK-13, TASK-15 |
| `M9` | Split `DashboardPage.tsx`; single owner | TASK-06, TASK-13, TASK-15 |
| `M10` | `TaskCard` / `BlockCard` / `CompletionAction` in scope | TASK-10, TASK-11 |
| `M11` | 400+ timezone options vs non-virtualized `Select` | TASK-12 |
| `M12` | Build cannot detect E2E selector drift | TASK-13, TASK-15, and every task's gate |
| `M13` | PostCSS in `apps/web`; Mantine styles first | TASK-01, TASK-03, TASK-09, TASK-14, TASK-15 |
| `M14` | Fresh dev server before every Playwright run | TASK-04 … TASK-15 (all gates) |
| `M15` | Five status regions; CalendarToolbar ownership | TASK-06, TASK-07, TASK-15 |
| `M16` | No competing sticky-header CSS | TASK-08, TASK-09, TASK-12 |
| `M17` | Split `CalendarStyles.css` before 400 lines | TASK-09, TASK-14 |
| `M18` | `TokenRetryModal` / `RefreshButton` in `features/completion/` | TASK-11 |
| `P-1` | Validation order ui → web → e2e | All tasks |
| `P-2` | `pnpm install` after every manifest change | TASK-01, TASK-04 |
| `P-3` | PostCSS config must be `.cjs` | TASK-01 |
| `P-4` | No lint gate | TASK-01 and every task's gate |
| `P-5` | E2E specs immutable | TASK-06 … TASK-15 |
| `P-6` | Never modify tsconfig / scripts to pass | TASK-01 and every task's forbidden list |
| `P-7` | Phases 1–5 sequential; only phase 6 parallel | This document; the batch plan |
| `P-8` | One `DashboardPage.tsx` owner | TASK-06; forbidden in TASK-07 and all three lanes |
| `P-9` | No file above 400 lines | TASK-06, TASK-09, TASK-15 |
| `P-10` | Fix the implementation, never the test | All tasks |

**Coverage: 32/32.** `B1`–`B4` (4) + `M1`–`M18` (18) + `P-1`–`P-10` (10).

---

## Risk Binding

| Risk | Primary tasks |
| --- | --- |
| `R-01` E2E selector breakage | TASK-05, TASK-07, TASK-10, TASK-11 |
| `R-02` Duplicate DOM copy (`Preview`, `<aside>`) | TASK-05, TASK-10, TASK-11, TASK-13 |
| `R-03` `eventContent` breaks event DOM | TASK-08, TASK-09 |
| `R-04` AppShell height breaks calendar | TASK-05 |
| `R-05` Stale `packages/ui/dist` | TASK-04 and every gate |
| `R-06` `businessHours` scope creep | TASK-07, TASK-08, TASK-09 |
| `R-07` Breakpoint drift | TASK-02, TASK-03, TASK-14 |
| `R-08` Contrast regression | TASK-02, TASK-03, TASK-09, TASK-15 |
| `R-09` Silent CSS no-op / cascade fight | TASK-09 |
| `R-10` Toolchain failure (`.cjs`, import order) | TASK-01, TASK-03 |
| `R-11` Stale dev server invalidates a run | Every gate; TASK-13, TASK-15 |
| `R-12` Build green while E2E is broken | TASK-13, TASK-15 |
| `R-13` `DashboardPage.tsx` concurrent writes | TASK-06 (owner), TASK-07, all lanes |
| `R-14` Missing / mis-pathed feature files | TASK-10, TASK-11 |
| `R-15` `SettingsModal` performance | TASK-12 |
| `R-16` Control duplication after the ownership split | TASK-06, TASK-07 |
| `R-17` Sticky-header jitter | TASK-08, TASK-09 |
| `R-18` Stylesheet exceeds 400 lines | TASK-09, TASK-14 |
| `R-23` Copy collision | TASK-07, TASK-10, TASK-11 |
| `R-25` Lane merge regression | TASK-13 |
