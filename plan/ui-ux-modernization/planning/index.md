# Planning — UI/UX Modernization

Pipeline context: `ui-ux-modernization`
Design status: **approved** (Solution Designer, Option C)
Planning status: **complete** — validated against the actual codebase
Task files: **not created here** (Tech Lead owns `.opencode/plan/ui-ux-modernization/tasks/`)

---

## Overview

### Objective

Deliver the approved Mantine-based UI/UX modernization of the `apps/web` frontend in a single
execution, in seven strictly ordered phases, without breaking the immutable Playwright suite
`apps/web/e2e/TASK-01.spec.ts` and `apps/web/e2e/TASK-02.spec.ts`, and without breaching the
project's 400-line file-size rule.

### Scope

**In scope**

- `apps/web` frontend only: design tokens, theme, app shell, calendar rendering, feature components.
- `packages/ui` wrapper components rebuilt on Mantine.
- Dependency additions (`@mantine/core@8.3.18`, `@mantine/hooks`, `@mantine/modals`,
  `@mantine/notifications`, `@tabler/icons-react`) and the PostCSS pipeline in `apps/web`.
- `pnpm-lock.yaml` regeneration via `pnpm install`.

**Out of scope**

- `packages/backend`, `packages/shared`, database schema, API contracts — **no changes**.
- `apps/web/e2e/**` — **immutable**, zero edits permitted.
- Project configuration used to make tests pass (tsconfig, eslint, vitest, package scripts).
- Task files (Tech Lead).

### Baseline (verified 2026-10-03)

`run-package-command build apps/web` → **PASS** (`tsc && vite build`, 158 modules transformed).

| Asset | Raw | Gzip |
| --- | --- | --- |
| `dist/assets/index-6jbnJNN9.js` | 524.63 kB | 156.62 kB |
| `dist/assets/index-D_hRdG_e.css` | 6.40 kB | 1.75 kB |

`apps/web` has **no `lint` script** — there is no lint gate at any phase.

---

## Recommendation

Proceed with the approved Option C design, subject to the mandatory corrections recorded in this
planning set:

- **4 blocking corrections** (`B1`–`B4`) → [`feasibility.md`](./feasibility.md), [`risks.md`](./risks.md)
- **18 material corrections** (`M1`–`M18`) → [`impact-analysis.md`](./impact-analysis.md)
- Phase 6 lane split, per-phase gates, `DashboardPage.tsx` ownership →
  [`dispatch-plan.md`](./dispatch-plan.md)
- Canonical cross-phase index of all 32 constraint IDs → [`constraints.md`](./constraints.md)

The design direction is sound and stays approved. The corrections above are code-level facts
discovered by validating the design against the real repository; they must be built in from the
first task rather than discovered during implementation.

---

## Phase Sequence

Phases 1–5 are **strictly sequential** — each depends on artifacts produced by the previous one.
Phase 6 is the only phase that fans out, into **3 file-disjoint lanes** that run in parallel.
Phase 7 runs after everything.

| # | Phase | Parallelism | Exit gate |
| --- | --- | --- | --- |
| 1 | Dependencies, lockfile, PostCSS pipeline | sequential | `pnpm install` OK, PostCSS `.cjs` config present, `build apps/web` PASS |
| 2 | Design tokens, theme, Mantine providers, dark mode | sequential | `build apps/web` PASS |
| 3 | `packages/ui` Mantine wrapper components | sequential | `build packages/ui` PASS **then** `build apps/web` PASS |
| 4 | App shell: AppShell, Header, Sidebar, StatusBar | sequential | full gate (ui → web → e2e) PASS |
| 5 | Calendar: tokens, `eventContent`, grid, weekend tint | sequential | full gate (ui → web → e2e) PASS |
| 6 | Feature component migration (tasks, blocks, preview, completion, settings, dev-mode) | **3 parallel lanes** | full gate (ui → web → e2e) PASS |
| 7 | Dead-CSS cleanup, bundle check, final validation | sequential | full gate (ui → web → e2e) PASS |

### Per-phase validation gate (identical for phases 3–7)

1. `run-package-command build packages/ui` — must run **first**; `apps/web` resolves `@brkroutnxdle/ui`
   through `packages/ui/dist`, and `packages/ui` is `tsc`-only, so a targeted `apps/web` build will
   silently consume **stale declarations** if `packages/ui` is not rebuilt.
2. `run-package-command build apps/web` — `tsc && vite build`.
3. `run-package-command test apps/web` — Playwright.

Playwright notes (mandatory reading before trusting a green run):

- `apps/web/playwright.config.ts` uses `webServer: pnpm dev` with **`reuseExistingServer: true`**.
  A stale dev server from a previous phase can be reused and produce false passes *and* false
  failures. The runner must ensure no stale server is bound to the dev port, or the run is invalid.
- `fullyParallel: false`, `retries: 1`, viewport `1440x900` — a single serial suite.
- `apps/web/tsconfig.json` **excludes `e2e`**, so `build apps/web` does not typecheck the specs.
  Playwright is the only check on spec/source contract drift.

---

## Parallel Dispatch Summary

### Phases 1–5: strictly sequential

Do **not** overlap. `main.tsx`, the package manifests, `pnpm-lock.yaml`, the token stylesheets and
`DashboardPage.tsx` are each touched by more than one of these phases.

### Phase 6: definitive 3-lane split (file-disjoint)

| Lane | Scope | Files |
| --- | --- | --- |
| **6-A** | Tasks + Blocks | `features/tasks/TaskPanel.tsx`, `TaskModal.tsx`, `TaskCard.tsx`, `features/blocks/BlockPanel.tsx`, `BlockModal.tsx`, `BlockCard.tsx` |
| **6-B** | Preview + Completion + Shared | `features/preview/PreviewToolbar.tsx`, `SaveButton.tsx`, `CalculateWeekButton.tsx`, `AllocationFailurePanel.tsx`, `DeleteEventDialog.tsx`, `features/completion/ConflictDialog.tsx`, `CompletionAction.tsx`, `TokenRetryModal.tsx`, `RefreshButton.tsx`, `features/shared/CreateActionModal.tsx` |
| **6-C** | Settings + Dev mode + Errors + Login | `components/SettingsModal.tsx`, `DevModePanel.tsx`, `DevModeBadge.tsx`, `ErrorFallback.tsx`, `ErrorBoundary.tsx`, `pages/LoginPage.tsx`, `App.tsx` |

The three lanes share **zero** files. Roughly balanced by line count (758 / 865 / 633).
Full detail, including the forbidden-file lists per lane, is in
[`dispatch-plan.md`](./dispatch-plan.md#phase-6--definitive-lane-split).

### `DashboardPage.tsx` single-owner rule

`apps/web/src/pages/DashboardPage.tsx` (567 lines) is required by phases 4, 5 **and** 6.
It must have **exactly one owning lane for the whole delivery**. Two named options:

- **Option 1 (recommended)** — Phase 4 owner performs the mandatory 567-line split early
  (see `M9`), and phases 5 and 6 route all changes through that owner via explicit interface tasks.
- Option 2 — one lane is designated "DashboardPage owner" and holds it exclusively for phases 4–7.

Under no arrangement may two lanes edit `DashboardPage.tsx` concurrently.

---

## Planning Documents

| Document | Purpose |
| --- | --- |
| [`index.md`](./index.md) | This overview: scope, phase sequence, dispatch summary |
| [`feasibility.md`](./feasibility.md) | Per-phase verdicts, gates, blocking corrections `B1`–`B4` |
| [`constraints.md`](./constraints.md) | Canonical index of all 32 constraint IDs, phase, enforcement point |
| [`impact-analysis.md`](./impact-analysis.md) | Affected areas, overlap flags, material corrections `M1`–`M18` |
| [`dispatch-plan.md`](./dispatch-plan.md) | Dispatch order, per-phase gates, Phase 6 lane split + forbidden lists, `DashboardPage.tsx` single-owner rule |
| [`file-impact.md`](./file-impact.md) | Per-file inventory with line counts, owning phase, lane, overlap flag |
| [`risks.md`](./risks.md) | Merged risk register, technical/delivery/operational risks, decision summary |
| [`risk-register.md`](./risk-register.md) | Full risk table with IDs, likelihood, impact, mitigation, source |

Design source of truth: `.opencode/plan/ui-ux-modernization/design-docs/`
(`index.md`, `design-01` … `design-07`).

---

## Decision Summary

### Selected Approach

Approved Option C (Mantine 8.3.18 + three-tier CSS custom properties + AppShell shell +
FullCalendar re-skin + component migration), executed as one delivery across 7 phases, with
`B1`–`B4` and `M1`–`M18` applied as mandatory implementation constraints.

### Reasoning

The library choice, token architecture and shell direction are sound and were validated as
implementable. However, four design assumptions are provably false against the current code
(`SegmentedControl` emits no `<button>`; the E2E suite depends on native `<select>`; FullCalendar
`businessHours` shades the E2E-asserted Sunday slot; the proposed dark event colour fails contrast).
None of these invalidate the design — they constrain how it must be built. Since the user approved
folding them into planning rather than reopening design, they are recorded here with stable IDs so
the Tech Lead can reference them directly from task files.

### Expected Outcome

A modernised, dark-mode-capable, token-driven frontend with a Mantine-based shell and feature
components, with all 20+ Playwright assertions in `TASK-01.spec.ts` and `TASK-02.spec.ts`
still passing, no file above 400 lines, and no backend or API change.

### Open Questions

None blocking. Two items are explicitly **deferred by user decision** and are recorded as
constraints rather than questions:

- `B3` — FullCalendar `businessHours` prop is **deferred**; workday tint is delivered via CSS.
- `M3` / `M15` — Header vs. CalendarToolbar ownership is **fixed** by the split recorded in
  `impact-analysis.md` and `M15`.
