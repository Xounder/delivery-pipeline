# Design 06 — Migration Order, File-Level Change List, Parallel Dispatch Plan

## Context

### Problem Statement

The approved scope is a single delivery spanning `packages/ui`, the app shell, the calendar,
six feature areas, and the theme layer. The orchestrator needs a file-level plan with
**explicit overlap flags** so parallel subagent dispatch does not produce conflicting edits.

### Goals

- Order the work so the build and the E2E suite stay green at every step.
- List every file to be created, modified or deleted, with its owning work item.
- Mark files touched by more than one work item so the orchestrator can serialise them.
- Keep every step independently revertible.

### Constraints

- The E2E suite must be runnable and green after each phase boundary.
- No pre-existing test may be modified.
- `apps/web` build must not break between phases.
- Every file stays under 400 lines (project rule); oversized files must be split rather than
  grown.

---

## Options Considered

### Option A (Selected): Foundation-first, 7 sequential phases with 3 parallel lanes

#### Description

Dependency order is enforced: build config → tokens → `packages/ui` → shell → calendar →
features → cleanup. Within phases 4 and 5, independent work items are marked `PARALLEL-OK`.

#### Advantages

- Build stays green throughout; `build` and Playwright can gate each boundary.
- Parallel lanes are explicit, so the orchestrator can dispatch safely.
- Each phase is revertible in isolation.

#### Disadvantages

- Tokens and `packages/ui` land before any visible change, so early phases look inert.

#### Effort Estimate

Large (whole delivery)

---

### Option B (Rejected): Vertical slice per feature area

**Rejected because** each slice would need its own brand decisions before the token layer and
`packages/ui` exist, producing inconsistent styling and duplicated wrappers.

### Option C (Rejected): Big-bang rewrite

**Rejected because** the E2E suite could not bisect a regression, and it contradicts the
project's incremental delivery conventions.

---

## Decision

### Selected Option

Option A.

### Reasoning

The dependency graph is real: nothing can consume `packages/ui` primitives before they exist,
and nothing can consume `--brk-*` tokens before they are defined. Encoding that order
explicitly is what makes parallel dispatch safe.

### Trade-offs Accepted

- Later phases depend on earlier ones completing; no fully parallel start.

---

## Impact

### Architecture Affected

All of `packages/ui` and the `apps/web` UI surface. No changes to `apps/api`,
`packages/shared`, `packages/domain` or `packages/calendar`.

---

## Phase Plan

### Phase 1 — Build configuration (no UI change)

| Action | File | Note |
|---|---|---|
| add | `apps/web/postcss.config.cjs` | `postcss-preset-mantine` + `postcss-simple-vars` with the three breakpoints from `design-03` |
| modify | `apps/web/src/main.tsx` | Import `@mantine/core/styles.css`, `@mantine/notifications/styles.css` **first** |
| modify | `apps/web/package.json` | Add the 8 dependencies from `design-01` |
| modify | `apps/web/index.html` | Insert `ColorSchemeScript` before the app script |

**Gate:** `build` passes. No visual change expected. **PARALLEL-OK: none (sequential).**

### Phase 2 — Theme / token layer

| Action | File | Note |
|---|---|---|
| modify | `apps/web/src/styles/design-tokens.css` | Split into Tier 1 / 2 / 3; keep all existing names; pin `--mantine-color-*` |
| modify | `apps/web/src/styles/global.css` | Remove duplicated reset; remove dead `@media` blocks; keep `.sr-only` |
| add | `apps/web/src/theme/mantineTheme.ts` | `createTheme` with the `teal` tuple |
| add | `apps/web/src/theme/ThemeProvider.tsx` | `MantineProvider` + `ModalsProvider` + `Notifications` |
| modify | `apps/web/src/main.tsx` | Wrap the provider tree; move stylesheet imports into documented order |

**Gate:** `build` + Playwright green, both schemes render. **PARALLEL-OK: none.**

### Phase 3 — `packages/ui` primitives

| Action | File | Note |
|---|---|---|
| modify | `packages/ui/package.json` | Add `@mantine/core` peer + dev dep |
| add | `packages/ui/src/components/IconButton.tsx` | `label` → `aria-label`, required |
| add | `packages/ui/src/components/SegmentedControl.tsx` | `label` → `aria-label`, required |
| add | `packages/ui/src/components/Badge.tsx`, `Alert.tsx`, `StatusMessage.tsx`, `Card.tsx` | |
| add | `packages/ui/src/components/TextField.tsx`, `NumberField.tsx`, `SelectField.tsx`, `ToggleField.tsx`, `CheckboxGroup.tsx` | |
| add | `packages/ui/src/components/Tooltip.tsx`, `Spinner.tsx`, `Kbd.tsx`, `ThemeToggle.tsx` | |
| modify | `packages/ui/src/components/Button.tsx`, `Modal.tsx`, `FAB.tsx`, `BottomSheet.tsx` | Re-implement over Mantine; keep existing prop names |
| add | `packages/ui/src/theme.ts`, `BrkroutnxdleProvider.tsx` | |
| modify | `packages/ui/src/index.ts` | New export surface |

**Gate:** `packages/ui` `build` passes. `apps/web` unchanged so far → Playwright trivially green.
**PARALLEL-OK: yes** — component files are independent; only `index.ts` and `package.json` are shared.

### Phase 4 — App shell

| Action | File | Notes |
|---|---|---|
| modify | `apps/web/src/components/MainLayout.tsx` | → `AppShell`, `padding={0}` |
| modify | `apps/web/src/components/Header.tsx` | → `Group`/`SegmentedControl`/`IconButton`; add `aria-label`s |
| modify | `apps/web/src/components/Sidebar.tsx` | → `AppShell.Navbar component="aside"` + `Accordion` |
| add | `apps/web/src/components/StatusBar.tsx` | Priority-ordered banner region |
| modify | `apps/web/src/pages/DashboardPage.tsx` | Replace 4 inline banners with `StatusBar`; move calendar nav into `CalendarToolbar` |

**Gate:** Playwright **TASK-02 green** (exercises `aside` + `Tasks (N)` + modals).
**PARALLEL-OK: partial** — `MainLayout` / `Header` / `Sidebar` may be done in parallel by one
lane; `DashboardPage.tsx` must be done by the same lane as `StatusBar`.

> **Overlap flag — `DashboardPage.tsx` is touched by Phase 4, 5 and 6.** It must be owned by a
> single agent for the whole delivery, or the phases must be serialised.

### Phase 5 — Calendar UX

| Action | File | Notes |
|---|---|---|
| modify | `apps/web/src/styles/CalendarStyles.css` | Rewrite as `--fc-*` layer; drop all `!important` |
| modify | `apps/web/src/components/CalendarView.tsx` | `eventContent`; remove `eventDidMount`/`eventWillUnmount` delete logic; extend `classNames`; `businessHours` |
| add | `apps/web/src/components/calendar/CalendarToolbar.tsx` | |
| add | `apps/web/src/components/calendar/EventContent.tsx` | |
| add | `apps/web/src/components/calendar/calendarTheme.ts` | `--brk-*` → `--fc-*` map |
| modify | `apps/web/src/pages/DashboardPage.tsx` | Render `CalendarToolbar` (**overlap flag**, see Phase 4) |

**Gate:** Playwright **both specs green** — this phase carries every `.fc-*` assertion.
**PARALLEL-OK: partial** — the three new `calendar/*` files may be built in parallel;
`CalendarView.tsx` and `CalendarStyles.css` must be sequenced together (same agent).

### Phase 6 — Feature areas

All rewrite hand-rolled markup to `packages/ui` primitives. **PARALLEL-OK: yes**, grouped into
three independent lanes:

| Lane | Files | E2E strings at risk |
|---|---|---|
| **L1 — tasks + blocks** | `features/tasks/TaskPanel.tsx`, `TaskModal.tsx`, `features/blocks/BlockPanel.tsx`, `BlockModal.tsx` | `Tasks (N)`, `New Task`, `Edit Task` |
| **L2 — preview + completion** | `features/preview/PreviewToolbar.tsx`, `SaveButton.tsx`, `CalculateWeekButton.tsx`, `AllocationFailurePanel.tsx`, `DeleteEventDialog.tsx`, `features/completion/ConflictDialog.tsx`, `RefreshButton.tsx`, `TokenRetryModal.tsx`, `features/shared/CreateActionModal.tsx` | `Preview:`, `Create in this slot` |
| **L3 — settings + dev + shell-adjacent** | `components/SettingsModal.tsx`, `DevModePanel.tsx`, `DevModeBadge.tsx`, `ErrorFallback.tsx`, `ErrorBoundary.tsx`, `pages/LoginPage.tsx`, `App.tsx` | none |

**Gate:** Playwright both specs green after **each** lane, since L1 and L2 both touch
E2E-asserted copy.

### Phase 7 — Cleanup and validation

| Action | File | Notes |
|---|---|---|
| verify | `apps/web/src/**` | Zero remaining hardcoded hex outside `design-tokens.css` |
| verify | `apps/web/src/styles/global.css` | No orphaned selectors |
| modify | `apps/web/src/main.tsx` | Final import order |

**Gate:** `build`, `typecheck`, `lint`, `test` all green; no `!important` left in
`CalendarStyles.css`.

---

## Overlap Flags for Parallel Dispatch

| File | Phases | Dispatch rule |
|---|---|---|
| `apps/web/src/pages/DashboardPage.tsx` | 4, 5, 6 | **Single owner for the whole delivery.** Never parallel. |
| `apps/web/src/main.tsx` | 1, 2, 7 | Sequential (build config, then provider, then final order) |
| `apps/web/package.json` | 1 | Single edit |
| `packages/ui/src/index.ts` | 3 | Merge after all Phase-3 component files land |
| `apps/web/src/components/CalendarView.tsx` | 5 | Same agent as `CalendarStyles.css` |
| `apps/web/src/styles/design-tokens.css` | 2 | Single owner; consumed by 4, 5, 6 |
| `apps/web/src/features/**` | 6 | Three parallel lanes; lanes are file-disjoint |
| `apps/web/e2e/*.spec.ts` | — | **Never modified.** Read-only reference for copy constraints |

**Critical-path ordering for the orchestrator:**

```
1 → 2 → 3 → 4 → 5 → 6 (L1 ∥ L2 ∥ L3) → 7
```

Phases 1–5 are strictly sequential. Only Phase 6 is genuinely parallel (3 lanes), and Phase 7
follows all lanes.

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| `DashboardPage.tsx` edited by two agents in parallel | Medium | **High** | Explicit single-owner flag above; the orchestrator must not split it |
| Long sequential chain delays feedback | Medium | Medium | Every phase has a `build` + Playwright gate, so failures surface early |
| Phase 3 lands with no consumer, appearing to do nothing | High | Low | Phases 4–6 consume it immediately; keep 3 and 4 in the same PR if possible |
| A component exceeds 400 lines after the rewrite | Medium | Medium | `SettingsModal.tsx` (240+ lines) and `TaskModal.tsx` are the candidates; extract field groups into `packages/ui` or per-feature files |
| Lane L1 or L2 breaks copy in isolation | Medium | **High** | Run Playwright after **each** lane, not just after all three |

---

## Open Questions

- None blocking. If the orchestrator can only afford a single dispatch, Phase 1 + 2 + 5 is the
  highest-value subset (it delivers the themed calendar the user explicitly complained about)
  — but it requires the `packages/ui` wrappers for the toolbar, so Phase 3 must be included.