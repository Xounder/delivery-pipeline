# TASK-12 — Phase 6 Lane 6-C: Settings, Dev Mode, Errors and Login Migration

## Task Information

### ID

TASK-12

### Title

Migrate `SettingsModal`, Dev-mode components, error boundary/fallback and `LoginPage` to Mantine; rebase `App.tsx` on Phase 2 providers

### Phase

Phase 6 lane 6-C — Settings + Dev mode + Errors + Login (≈633 lines, 7 files)

### Owner

senior-frontend

### Status

Pending

---

## Description

Seven files migrate to Mantine. Unlike lanes 6-A and 6-B, **no E2E spec asserts any of these
internals**, so Mantine form controls are permitted here — with one performance caveat (`M11`) and one
merge caveat (`OVERLAP-8`).

**Parallel dispatch:** runs concurrently with TASK-10 and TASK-11. Zero shared files.

---

## Files This Task May Create / Modify

| # | File | Lines now | Action |
| --- | --- | --- | --- |
| 1 | `apps/web/src/components/SettingsModal.tsx` | 263 | modify — Mantine controls; `M11` timezone caveat |
| 2 | `apps/web/src/components/DevModePanel.tsx` | 101 | modify — Mantine modal |
| 3 | `apps/web/src/components/DevModeBadge.tsx` | 38 | modify — Mantine badge |
| 4 | `apps/web/src/components/ErrorFallback.tsx` | 65 | modify — Mantine; consumes `@brkroutnxdle/ui` `Button` |
| 5 | `apps/web/src/components/ErrorBoundary.tsx` | 37 | modify — Mantine boundary |
| 6 | `apps/web/src/pages/LoginPage.tsx` | 72 | modify — Mantine form controls |
| 7 | `apps/web/src/App.tsx` | 57 | modify — Dev-mode provider tree; login/dashboard switch |

### File count

7 modified, 0 created

### Do Not Touch — forbidden for lane 6-C

- `apps/web/src/features/**` (TASK-10 / TASK-11)
- `apps/web/src/components/CalendarView.tsx`, `components/MainLayout.tsx`, `components/Header.tsx`,
  `components/Sidebar.tsx`, `components/StatusBar.tsx`, `components/calendar/**`
- `apps/web/src/pages/DashboardPage.tsx` — **OVERLAP-1** (`P-8`); `pages/LoginPage.tsx` is in scope, other pages are not
- `apps/web/src/styles/**` (TASK-02 / TASK-09 / TASK-14)
- `apps/web/src/main.tsx` (**OVERLAP-2**)
- `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `packages/ui/src/**`, `packages/ui/package.json`, `packages/ui/dist/**` — **OVERLAP-9 read-only**;
  **OVERLAP-10** generated
- `apps/web/e2e/**`, `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-5`, `P-6`)
- `apps/api/**`, `packages/shared/**`, `packages/backend/**`, `packages/domain/**`, `packages/calendar/**`

---

## Constraints Satisfied

- `M11` — `SettingsModal`'s 400+ timezone options must not be swapped for a non-virtualized `Select`
- `M16` — no sticky CSS added
- `OVERLAP-8` — `App.tsx` rebases onto Phase 2 provider changes instead of reverting them
- `OVERLAP-9` — `packages/ui` consumed read-only
- `P-5`, `P-6`, `P-9`

---

## Implementation Steps

1. **`SettingsModal.tsx` (263 lines) — `M11` is the only hard constraint here.**
   `timezoneOptions` is built from `Intl.supportedValuesOf("timeZone")` (400+ entries) at mount.
   Mantine `Select` is **not virtualized** — it renders every option into the dropdown DOM, which is a
   measurable open cost on a field users open often. Either:
   - keep a **native `<select>`** for the timezone field (lower risk, and consistent with `B2`'s
     rationale), or
   - add a search/filter affordance **before** switching to Mantine `Select`.
   The other two selects in this file may be migrated freely. No spec asserts this component's
   internals. Preserve every existing copy string, the persistence behaviour (settings survive a
   reload) and the `onSave` payload shape.
2. **`DevModePanel.tsx` / `DevModeBadge.tsx`** — Mantine `Modal`, `Badge`, `Text`, `Group`. Preserve
   the badge's trigger text and the panel's auth-token copy. Note: the calendar error path in
   TASK-06's `StatusBar` re-opens this panel when the API reports an expired token — the existing
   prop/callback contract must survive.
3. **`ErrorBoundary.tsx` / `ErrorFallback.tsx`** — Mantine `Alert`, `Stack`, `Text`,
   `Code`. `ErrorFallback` is one of only **two** consumers of `@brkroutnxdle/ui` (the other is
   lane 6-B's `TokenRetryModal`); its `Button` usage must stay compatible with the Phase 3 wrapper API
   (`variant` / `size` prop names preserved, `isOpen`-style naming mapped inside the wrapper).
   `ErrorBoundary` must keep resetting child renders on error the way it does today.
4. **`LoginPage.tsx`** — Mantine form controls are permitted (no spec asserts its internals).
   Preserve the existing labels, the submit behaviour and the dev-mode entry point if one exists.
5. **`App.tsx` — `OVERLAP-8`, rebase, do not revert.** Phase 2 may have added providers here. Read
   the file as it currently stands, keep every Phase 2 change, and layer the Dev-mode provider tree
   and the login/dashboard switch on top. **Do not** revert a provider you did not understand.
6. **Scan all seven files** for leftover `style={{` inline styles.
7. **No new files** in this lane.

---

## Acceptance Criteria

- [ ] The timezone field is either still a native `<select>` or has a working search/filter before any Mantine `Select` swap (`M11`)
- [ ] No Mantine `Select` anywhere in this lane renders the full 400+ option list without filtering
- [ ] `ErrorFallback` still renders a working `@brkroutnxdle/ui` `Button` with the original action behaviour
- [ ] `ErrorBoundary` still catches render errors and still resets its subtree
- [ ] `App.tsx` retains every Phase 2 provider change (no reverts)
- [ ] The dev-mode badge/panel auth flow still works end to end
- [ ] Settings persist across a reload
- [ ] All seven files remain under 400 lines
- [ ] No file outside this lane's table is modified
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-09

### Dependency Notes

May only start after Phase 5 is merged and `packages/ui` has been built. `App.tsx` must be read in its
**current** state at dispatch time because Phase 2 owns `OVERLAP-8` there. Runs concurrently with
TASK-10 and TASK-11.

### Runs In Parallel With

- TASK-10 (lane 6-A) — file-disjoint
- TASK-11 (lane 6-B) — file-disjoint

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** on a fresh dev server (`M14`)
4. No lint step (`P-4`)

**Merge rule:** after TASK-10, TASK-11 and TASK-12 all complete, the full gate runs once more on the
consolidated tree (TASK-13).

---

## Edge Cases

- **Settings modal becomes slow to open** — the 400+ timezone entries were moved into a
  non-virtualized Mantine `Select`. Revert that field to a native `<select>` or add filtering
  (`M11`). No E2E assertion will catch this; only manual observation will.
- **Settings no longer persist** — the `onSave` payload shape changed during migration; the API
  contract is unchanged by this delivery, so a shape change is a defect.
- **Provider tree broken on `App.tsx`** — a Phase 2 provider was reverted instead of rebased
  (`OVERLAP-8`). Diff `App.tsx` against the Phase 2 result before editing.
- **Dev-mode badge no longer reachable** — the `StatusBar`'s token-modal re-open path depends on the
  existing callback contract; changing the callback shape breaks TASK-06's retry behaviour.
- **`ErrorFallback` `Button` type error or visual regression** — the Phase 3 wrapper kept
  `variant` / `size` names and maps Mantine's naming internally. Passing `opened`, `color` or Mantine
  `size` values that no longer exist breaks the API surface (`R-05`).
- **Error boundary swallows errors silently** — the reset path changed, or the fallback stopped
  rendering inside the boundary's contract.
- **Login page loses the dev-mode link** — a control was dropped during the Mantine rewrite.
- **400-line ceiling** — `SettingsModal.tsx` at 263 lines is the risk; extract inside the same folder
  if the rewrite grows it.
- **Stale dev server** — false 9/9 against another lane's bundle (`M14`).
- **Cross-lane ownership conflict** — if this lane believes it must change a feature or calendar file,
  that is a lane-boundary violation. Report it; do not edit.

---

## Testing

### Unit Tests

- [ ] None — no component test runner exists in `apps/web` and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9 — the suite does not exercise these internals, so a green run here is necessary but **not sufficient** for this lane; manual validation below is mandatory

### Manual Validation

- [ ] Open Settings; measure that the timezone dropdown opens promptly with 400+ options
- [ ] Change a setting, save, reload, confirm persistence
- [ ] Trigger a deliberate render error; confirm the boundary shows the fallback and reset works
- [ ] Toggle dev mode, use the token modal, confirm the badge and panel work
- [ ] Load the login page, submit, confirm navigation

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Do not add specs to cover untested components in this lane
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] Lane forbidden lists respected
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Manual validation of settings, errors, dev mode and login completed
- [ ] Validation completed

---

## References

- [`planning/dispatch-plan.md` — Lane 6-C](../planning/dispatch-plan.md#lane-6-c--settings--dev-mode--errors--login-633-lines-7-files)
- [`planning/file-impact.md` — Misc components table](../planning/file-impact.md#misc-components)
- [`planning/impact-analysis.md` — M11, OVERLAP-8, OVERLAP-9, R-05](../planning/impact-analysis.md#file-overlap-flags)
- [`design-docs/design-05-component-inventory.md`](../design-docs/design-05-component-inventory.md)
- [`design-docs/design-06-migration-plan.md`](../design-docs/design-06-migration-plan.md)