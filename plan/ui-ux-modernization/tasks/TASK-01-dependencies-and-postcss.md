# TASK-01 — Mantine Dependencies, Lockfile Regeneration, PostCSS Pipeline

## Task Information

### ID

TASK-01

### Title

Add Mantine 8.3.18 and the PostCSS pipeline to `apps/web`, regenerate `pnpm-lock.yaml`

### Phase

Phase 1 — Dependencies, lockfile, PostCSS pipeline

### Owner

senior-frontend

### Status

Pending

---

## Description

Add the approved component-library dependencies to `apps/web`, create the PostCSS configuration
required by Mantine, and regenerate the lockfile. **No UI change is expected in this task** — its
sole purpose is to make the toolchain capable of loading Mantine without breaking the existing
build.

The app is React `18.2`; Mantine `9.x` requires React `19.2+`, so every Mantine package MUST be
pinned to exactly `8.3.18`.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/package.json` | modify — add dependencies |
| 2 | `apps/web/postcss.config.cjs` | **create** |
| 3 | `apps/web/vite.config.ts` | modify — only if PostCSS auto-discovery fails |
| 4 | `pnpm-lock.yaml` | regenerate via `pnpm install` |

### File count

4 (1 created, 3 modified/regenerated)

### Do Not Touch

- `apps/web/e2e/**` — **never modified** (`P-5`)
- `apps/web/src/**` — no source change in this task
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, root `package.json` (`P-6`)
- `apps/web/package.json` **`scripts`** — no `lint` script may be added (`P-4`)
- `packages/ui/**`, `packages/shared/**`, `packages/domain/**`, `packages/calendar/**`
- `apps/api/**`
- `pnpm-lock.yaml` — never hand-edit; only `pnpm install` may write it (`OVERLAP-4`)

---

## Constraints Satisfied

- `P-2` — `pnpm install` MUST be run after every dependency-manifest change
- `P-3` — the PostCSS config MUST be `apps/web/postcss.config.cjs` (`.js` fails under `"type": "module"`)
- `P-4` — no `lint` script is added and no lint result is reported
- `P-6` — no project configuration is modified to make a build pass
- `M-13` — the PostCSS pipeline belongs to `apps/web` (Vite); `packages/ui` gains no CSS pipeline
- `R-10` — wrong PostCSS config extension fails the whole build

---

## Implementation Steps

1. **Edit `apps/web/package.json` `dependencies`** — add, pinned to the exact version:
   - `@mantine/core`: `8.3.18`
   - `@mantine/hooks`: `8.3.18` (peer of core; MUST match exactly)
   - `@mantine/modals`: `8.3.18`
   - `@mantine/notifications`: `8.3.18`
   - `@tabler/icons-react`: `^3.48.0`
2. **Edit `apps/web/package.json` `devDependencies`** — add the PostCSS pipeline:
   - `postcss`, `postcss-preset-mantine`, `postcss-simple-vars`
   - Keep the existing `scripts` block byte-for-byte identical. **Do not add `lint`.**
3. **Create `apps/web/postcss.config.cjs`** with CommonJS syntax:
   - `plugins: { "postcss-preset-mantine": {}, "postcss-simple-vars": { variables: { "mantine-breakpoint-xs": "36em", "mantine-breakpoint-sm": "48em", "mantine-breakpoint-md": "62em", "mantine-breakpoint-lg": "75em", "mantine-breakpoint-xl": "88em" } } }`
   - `postcss-preset-mantine` MUST run **before** `postcss-simple-vars`.
   - Export via `module.exports = { plugins: {...} }`.
   - The `.cjs` extension is mandatory: `apps/web/package.json` declares `"type": "module"`, so a
     `postcss.config.js` would be parsed as ESM and fail to load.
4. **Run `pnpm install`** from the repository root. Confirm:
   - it completes with **no peer-dependency error** (a React 19 peer complaint means a wrong version
     was resolved — fix the pin, do not use `--force` or `--legacy-peer-deps`);
   - `pnpm-lock.yaml` is updated and contains the new packages;
   - exactly one `@mantine+core` entry exists under `node_modules/.pnpm` (a second copy means a
     peer was declared as a hard runtime dependency somewhere — that must not happen yet).
5. **Only if the build reports that PostCSS was not loaded**, add an explicit `css: { postcss: "./postcss.config.cjs" }`
   block to `apps/web/vite.config.ts`. If Vite auto-discovery works, leave `vite.config.ts`
   completely untouched (it currently holds only the React plugin and the `/api` proxy — both must
   be preserved).

---

## Acceptance Criteria

- [ ] `apps/web/package.json` lists `@mantine/core`, `@mantine/hooks`, `@mantine/modals`, `@mantine/notifications` at exactly `8.3.18`, plus `@tabler/icons-react`
- [ ] `apps/web/package.json` `devDependencies` lists `postcss`, `postcss-preset-mantine`, `postcss-simple-vars`
- [ ] `apps/web/package.json` `scripts` is unchanged and contains **no** `lint` entry
- [ ] `apps/web/postcss.config.cjs` exists, is CommonJS, and lists `postcss-preset-mantine` before `postcss-simple-vars`
- [ ] `pnpm-lock.yaml` contains the new packages; no peer-dependency error was reported
- [ ] `run-package-command build apps/web` → **PASS**, module count ≥ 158
- [ ] No file under `apps/web/src/**` was modified
- [ ] No file exceeds 400 lines

---

## Dependencies

### Required Tasks

- None (first task of the delivery)

### Dependency Notes

TASK-01 unblocks every other task: `apps/web` cannot import Mantine until the packages exist and
the lockfile resolves them.

---

## Validation Gate

Order is mandatory (`P-1`; this task's gate omits `packages/ui` because nothing in `packages/ui`
changes yet).

1. `pnpm install` → exit code 0, lockfile updated
2. `run-package-command build apps/web` → **PASS**
3. No lint step (`P-4`)

---

## Edge Cases

- **React 19 peer error on install** — Mantine `9.x` peers on React `^19.2.0`. If pnpm resolves
  `9.x`, the pin was lost (caret range or stale lockfile). Re-pin to `8.3.18`, delete nothing by
  hand, re-run `pnpm install`.
- **PostCSS config silently ignored** — if `postcss-preset-mantine` never runs, `main.tsx` will
  still build (Vite falls back to no PostCSS), so the failure surfaces only in Phase 2 as
  unresolved `--mantine-*` variables. Confirm the config loads by checking the Phase 2 build.
- **`postcss.config.js` instead of `.cjs`** — the build fails with a module-parse error. Rename the
  file; do not change `"type": "module"`.
- **Two `@mantine/core` copies** — happens if `packages/ui` declares it as a hard runtime
  dependency. That is TASK-04's concern; if TASK-01 detects it, report it rather than fixing it.
- **Empty `pnpm-lock.yaml` diff** — means install was skipped. The lockfile is a required
  deliverable of this task (`P-2`).
- **Windows path/lockfile churn** — if `pnpm install` rewrites unrelated resolutions, re-run it
  once. A second identical run must produce no further diff.

---

## Testing

### Unit Tests

- [ ] None — no source behaviour changes in this task

### Integration Tests

- [ ] None — the immutable E2E suite is the integration gate

### Manual Validation

- [ ] Confirm the app still boots (`build apps/web` succeeded and no import error was emitted)
- [ ] Confirm `node_modules/.pnpm` contains exactly one `@mantine+core*` directory

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created, modified, renamed or deleted** (`P-5`)
- [ ] Run `run-package-command test apps/web` unchanged as the regression gate

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `pnpm install` clean, lockfile committed as part of the change
- [ ] `build apps/web` passes
- [ ] Validation completed

---

## References

- [`planning/feasibility.md` — Phase 1](../planning/feasibility.md#phase-1--dependencies-lockfile-postcss-pipeline)
- [`planning/constraints.md`](../planning/constraints.md)
- [`planning/dispatch-plan.md` — Phase 1 Gate](../planning/dispatch-plan.md#phase-1-gate--dependencies-lockfile-postcss)
- [`design-docs/design-01-library-selection.md`](../design-docs/design-01-library-selection.md)
- `apps/web/package.json`, `apps/web/vite.config.ts`