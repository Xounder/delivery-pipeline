# TASK-04 — `packages/ui` Mantine Wrapper Layer

## Task Information

### ID

TASK-04

### Title

Rebuild the four `packages/ui` primitives on Mantine while preserving the public API; add `@mantine/core` as a peer dependency

### Phase

Phase 3 — `packages/ui` wrapper components

### Owner

senior-frontend

### Status

Pending

---

## Description

`packages/ui` currently exports `Button`, `Modal`, `FAB`, `BottomSheet` as hand-styled inline-style
primitives. Two app files consume it: `components/ErrorFallback.tsx` (`Button`) and
`features/completion/TokenRetryModal.tsx` (`Modal`). Re-implement those four wrappers over Mantine
and **preserve their current prop names** so both consumers keep compiling untouched.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `packages/ui/package.json` | modify — `@mantine/core` as **peer + dev** |
| 2 | `packages/ui/src/components/Button.tsx` | modify — wrap Mantine `Button` |
| 3 | `packages/ui/src/components/Modal.tsx` | modify — wrap Mantine `Modal`, map `isOpen` → `opened` |
| 4 | `packages/ui/src/components/FAB.tsx` | modify — wrap Mantine `ActionIcon` |
| 5 | `packages/ui/src/components/BottomSheet.tsx` | modify — wrap Mantine `Drawer` |
| 6 | `packages/ui/src/index.ts` | modify — keep the same five export lines |
| 7 | `pnpm-lock.yaml` | regenerate via `pnpm install` |

### File count

7 (all modified/regenerated; no new source files)

### Do Not Touch

- `apps/web/**` — no app file changes in this task; both consumers must compile **unmodified**
- `apps/web/e2e/**` (`P-5`)
- `packages/ui/dist/**` — generated artifact, never hand-edited (`OVERLAP-10`)
- `packages/ui/tsconfig.json` — do not change compiler settings (`P-6`)
- `packages/shared/**`, `packages/domain/**`, `packages/calendar/**`

---

## Constraints Satisfied

- `B1` — **`SegmentedControl` must NOT be added or exported from this package**
- `B2` — this package must not ship a `TextField`/`NumberField`/`SelectField` that invites Mantine
  inputs into the two E2E-asserted forms
- `M18` — corrected paths (`features/completion/TokenRetryModal.tsx`,
  `features/completion/RefreshButton.tsx`) recorded here so no new file is created in `components/`
- `P-1` — `build packages/ui` runs before `build apps/web`
- `P-2` — `pnpm install` after the manifest change
- `P-4`, `P-6` — no lint script, no config change
- `R-05` — build-order requirement documented in the validation gate

---

## Implementation Steps

1. **`packages/ui/package.json`** — add `@mantine/core` at exactly `8.3.18` to **both**
   `peerDependencies` and `devDependencies`. Do **not** add it to `dependencies`: a hard runtime
   dependency would let pnpm install a second Mantine instance (`R-03`/`design-05`).
   Keep `"build": "tsc"` unchanged — this package is `tsc`-only and gains **no** CSS pipeline
   (`design-05`). Never `import "@mantine/core/styles.css"` from this package.
2. **Run `pnpm install`** to regenerate `pnpm-lock.yaml` (`P-2`). Confirm no React 19 peer error and
   exactly one `@mantine+core*` entry under `node_modules/.pnpm`.
3. **`Button.tsx`** — re-implement over Mantine `Button`, keeping the public surface:
   - Keep `ButtonVariant = "primary" | "secondary" | "danger" | "ghost"` and
     `ButtonSize = "sm" | "md" | "lg"` exported exactly as today.
   - Keep `isLoading?: boolean`, `children`, and the rest-spread of
     `React.ButtonHTMLAttributes<HTMLButtonElement>` (including `style` pass-through).
   - Map internally: `primary → filled`, `secondary → default`, `danger → filled color="red"`,
     `ghost → subtle`; `sm/md/lg → size`. Map the four variants onto Mantine's `variant`/`color`/`size`
     **inside** the wrapper — never leak Mantine's names to call sites.
   - `isLoading` MUST keep rendering `aria-busy` and a spinner; the button MUST stay `disabled`.
   - The wrapper MUST render a real `<button>` element (Mantine `Button` does).
4. **`Modal.tsx`** — re-implement over Mantine `Modal`, keeping the public surface:
   - Props stay `isOpen: boolean`, `onClose: () => void`, `title: string`, `children`,
     `ariaLabel?: string`.
   - Map `isOpen` → Mantine's `opened` **inside** the wrapper. Keep `title` rendering the dialog
     title and keep the dialog's accessible name derived from `ariaLabel ?? title`.
   - Do **not** set `keepMounted`; Mantine's default (`false`) is required so a closed dialog
     leaves **nothing** in the DOM (`M8`). `TokenRetryModal` is E2E-adjacent.
   - Return the same semantics as today: nothing rendered when `isOpen` is false.
5. **`FAB.tsx`** — re-implement over Mantine `ActionIcon` with `position: fixed`, keeping the
   existing prop names.
6. **`BottomSheet.tsx`** — re-implement over Mantine `Drawer` positioned bottom on small screens,
   keeping the existing prop names.
   Both `FAB` and `BottomSheet` currently have **zero consumers**; they are optional cleanup. If
   either cannot be migrated without changing its public props, leave it as-is and record the reason
   in the task report — do not break the export surface.
7. **`index.ts`** — keep the export surface **byte-compatible**:
   ```ts
   export { Button } from "./components/Button";
   export type { ButtonVariant, ButtonSize } from "./components/Button";
   export { Modal } from "./components/Modal";
   export { FAB } from "./components/FAB";
   export { BottomSheet } from "./components/BottomSheet";
   ```
   **Do not add new exports in this task.** In particular do **not** export a `SegmentedControl`
   wrapper (`B1`) and do **not** export `TextField` / `NumberField` / `SelectField` (`B2`).
   App components in Phases 4–6 import `@mantine/core` directly; `packages/ui` remains a
   compatibility layer for its two existing consumers.

---

## Acceptance Criteria

- [ ] `@mantine/core@8.3.18` is in `peerDependencies` **and** `devDependencies`, and **not** in `dependencies`
- [ ] `packages/ui/package.json` `scripts` still contains exactly `"build": "tsc"`
- [ ] No `.css` import exists anywhere under `packages/ui/src`
- [ ] `Button`, `ButtonVariant`, `ButtonSize`, `Modal`, `FAB`, `BottomSheet` are still exported with unchanged prop names
- [ ] `Modal` accepts `isOpen` (not `opened`) publicly
- [ ] `SegmentedControl` does not appear in any file under `packages/ui`
- [ ] `pnpm-lock.yaml` updated; exactly one `@mantine+core*` install
- [ ] `run-package-command build packages/ui` → **PASS**
- [ ] `run-package-command build apps/web` → **PASS** with **zero** changes to `apps/web/src`
- [ ] `run-package-command test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-03

### Dependency Notes

The wrapper layer sits on the theme; the app consumes it from Phase 4 onwards. `packages/ui/dist`
must be rebuilt before every later `apps/web` build (`P-1`, `R-05`).

---

## Validation Gate

The order is **mandatory** — `apps/web` resolves `@brkroutnxdle/ui` through `packages/ui/dist`, and
`packages/ui` is `tsc`-only, so a targeted web build silently consumes stale declarations.

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **Wrapper leaks Mantine naming** — a call site starts passing `opened=` or `variant="filled"`.
  If `build apps/web` fails on `ErrorFallback.tsx` or `TokenRetryModal.tsx`, the wrapper is wrong,
  not the consumer. Do not edit `apps/web` to accommodate it.
- **`isOpen` renamed to `opened`** — silent API break; both consumers would fail to compile, which
  is the intended guard.
- **`keepMounted` inherited from a preset** — a closed `Modal` leaves its children in the DOM and
  would break the `text=Preview` uniqueness assertion. Assert `keepMounted={false}` explicitly or
  rely on the documented default and verify it.
- **Second Mantine instance** — `@mantine/core` accidentally added to `dependencies`; pnpm installs
  a second copy and styling/context break. Peer + dev only.
- **No CSS pipeline in `packages/ui`** — adding `import "@mantine/core/styles.css"` will fail `tsc`
  (`.css` is not resolvable) and, if worked around, would double-load Mantine. Styles belong in
  `apps/web/src/main.tsx`.
- **`FAB` / `BottomSheet` have no consumers** — no test can validate them. Migrate conservatively or
  defer with a recorded reason; never change their public props to make a rewrite easier.
- **`Button` variant gradient** — the current `primary` variant is a yellow gradient; Mantine's
  `filled` primary is brand teal. That is an intentional visual change, not a regression.
- **Modal portal** — Mantine modals render into `document.body`. `TASK-02` locates the task name
  with `page.locator("aside").locator(...)`; that name lives in `TaskCard` inside the `<aside>`,
  not in the modal, so the portal is safe. Do not move the modal content back inside the `<aside>`.
- **400-line ceiling** — all five files must stay under 400 lines.

---

## Testing

### Unit Tests

- [ ] None — `packages/ui` has no test runner configured and none may be added (`P-6`)

### Integration Tests

- [ ] `build apps/web` compiles both unmodified consumers — this is the real API-compatibility test
- [ ] `run-package-command test apps/web` → 9/9

### Manual Validation

- [ ] `ErrorFallback` renders a visible, clickable button when a render error is forced
- [ ] `TokenRetryModal` opens/closes and leaves no DOM behind

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Do not add a test runner, config or script to `packages/ui` (`P-6`)

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `build packages/ui` passes, **then** `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/feasibility.md` — Phase 3 + build-order gate](../planning/feasibility.md#phase-3--packagesui-mantine-wrapper-components)
- [`planning/dispatch-plan.md` — Phase 3 Gate](../planning/dispatch-plan.md#phase-3-gate--packagesui-wrappers)
- [`design-docs/design-05-component-inventory.md`](../design-docs/design-05-component-inventory.md)
- `packages/ui/src/index.ts`, `packages/ui/src/components/*`, `apps/web/src/components/ErrorFallback.tsx`, `apps/web/src/features/completion/TokenRetryModal.tsx`