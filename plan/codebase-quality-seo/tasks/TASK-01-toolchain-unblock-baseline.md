# Task — Toolchain Unblock & Verified Lint/Build Baseline

## Task Information

### ID

TASK-01

### Title

Remove the self-referential dependency and establish a verified green lint/build baseline

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item |
|---|---|---|
| **W1.1** | Critical | Remove self-referential dependency `"portfolio": "file:"` |
| **W1.2** | Critical | Delete untracked `pnpm-workspace.yaml` + `pnpm-lock.yaml`; restore clean `git status` |
| **W1.3** | Critical | Record the verified lint + build baseline into `feasibility.md` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Additional gate for this task:** no file under `src/` may be modified. This task is purely root-config + documentation.

---

## Description

The project currently **cannot be validated at all**. `package.json:14` declares `"portfolio": "file:"`, making the package depend on itself; pnpm resolves it as `portfolio 0.0.0` and the implicit install fails. Two untracked artefacts (`pnpm-workspace.yaml`, `pnpm-lock.yaml`) were left behind by the failed validation attempt.

Until this task lands, **no other task in this plan can be verified** — `npm run lint` and `npm run build` cannot be run, so no acceptance criterion anywhere else in the plan is checkable. This task is therefore the hard prerequisite for all 67 remaining work items.

Deliverables:

1. Delete the `"portfolio": "file:"` entry from `package.json`.
2. Delete the two untracked root files.
3. Run `npm run lint`, `npm run build` (`tsc -b && vite build`) and record the exact baseline counts verbatim into `.opencode/plan/codebase-quality-seo/feasibility.md` § *Toolchain Baseline*.
4. Fix only lint/type failures that are demonstrably introduced by Stage 1 itself; every pre-existing failure is recorded, not silently fixed.

---

## Acceptance Criteria

- [ ] `package.json` no longer contains a `"portfolio"` key. No other dependency line changes.
- [ ] `pnpm-workspace.yaml` and `pnpm-lock.yaml` no longer exist at the repo root.
- [ ] `git status --short` reports only `?? .gitmodules` and `?? .opencode/`.
- [ ] `npm run build` (`tsc -b && vite build`) **passes** and its output is recorded.
- [ ] `npm run lint` runs to completion. If it fails, the exact `<errors> errors, <warnings> warnings` count is recorded verbatim in `feasibility.md`.
- [ ] `feasibility.md` contains the exact three-line block required by `feasibility.md` § *Baseline recording requirement*:
      `LINT BASELINE:  <error count> errors, <warning count> warnings   (command: npm run lint)`
      `BUILD BASELINE: pass | fail                                    (command: npm run build)`
      `TYPECHECK:      pass | fail                                    (command: npx tsc -b)`
- [ ] The three "known pre-existing issues" listed in `feasibility.md` are each either confirmed fixed or listed with its exact lint/type error identifier and location.
- [ ] No file under `src/` is modified (`git diff --stat -- src/` is empty).
- [ ] Any pre-existing lint error fixed is listed in a separate "Fixed during unblock" table, distinct from the baseline, so later stages can distinguish pre-existing from newly-introduced failures.
- [ ] The recorded baseline is echoed verbatim in this task file's *Post-Execution Record* section.

---

## Dependencies

### Required Tasks

- None

### Dependency Notes

This is the root of the dependency graph. `index.md` Stage 1 entry criteria list plan approval only; this task additionally satisfies risk **R5** and delivery risk **D2**. Every other task in the plan declares a dependency (direct or transitive) on this one because `npm run lint` / `npm run build` cannot be run until it lands.

Do **not** attempt to fix the pnpm `ERR_PNPM_IGNORED_BLASKS` policy error by installing a prerender tool, adding a test framework, or upgrading the bundler. `feasibility.md` explicitly records the blocked-esbuild condition as an **environment policy**, not a project defect.

---

## File Overlap Warning

| File | Shared with | Merge strategy |
|---|---|---|
| `package.json` | **TASK-13** (W2.15 — router types) | **Sequential.** TASK-01 owns `package.json` completely; TASK-13 runs in Stage 2 after this task lands. |
| `pnpm-lock.yaml` *(deleted)* | — | n/a |
| `pnpm-workspace.yaml` *(deleted)* | — | n/a |
| `.opencode/plan/codebase-quality-seo/feasibility.md` | No other task edits `feasibility.md` | n/a |

This task touches **no** `src/` file, so it is fully parallel-safe with every other Stage 1 task — but it must still complete first, because nothing else can be verified without it.

---

## Technical Context

### Relevant Components

- None (no `src/` file is modified)

### Relevant Modules

- `package.json` — script definitions and the dependency graph
- `.opencode/plan/codebase-quality-seo/feasibility.md` — baseline record target

### Relevant APIs

- `npm run lint` → `eslint .`
- `npm run build` → `tsc -b && vite build`
- `npx tsc -b` — typecheck only

### Relevant Types

- None

---

## Implementation Guidance

### Expected Changes

1. Edit `package.json`, remove the line `"portfolio": "file:",` from `dependencies`. Leave `name`, `version`, `private`, `type`, `scripts`, and every other dependency byte-identical.
2. Delete `pnpm-workspace.yaml` and `pnpm-lock.yaml` from the repo root. They are **untracked** — use a filesystem delete, and verify they do not appear in `git status` afterwards. Do not `git rm` them (they were never committed).
3. Run `npm install` once to regenerate a clean dependency tree if the toolchain requires it.
4. Run `npm run lint`, `npm run build`, `npx tsc -b`. Capture stdout/stderr verbatim.
5. Edit `feasibility.md` § *Toolchain Baseline*, replacing `**Not run**` with the real results and inserting the three-line baseline block verbatim.
6. If `npm run build` fails on **pre-existing** type errors, decide explicitly: fix them here (and record them in a "Fixed during unblock" table) or record them as part of the ceiling. Default is to record, not fix — fixing unrelated errors widens this task's blast radius.

### Constraints

- **No `src/` edits.** This task must be provably limited to root config + `feasibility.md`.
- **Do not add any dependency** — not a test framework, not a prerenderer, not an ESLint plugin. `feasibility.md` § *Custom Approach* lists a test harness as an **unapproved** custom option (OOS-7) that requires explicit approval. `public/` must not be created here either.
- **Do not upgrade** `react`, `react-dom`, `react-router-dom`, `vite`, `typescript`, or `eslint`. Zero dependency upgrades anywhere in this plan.
- **Do not change any `scripts` entry.** `build` must remain `tsc -b && vite build` so the baseline is comparable with every later stage.
- Do not commit. Leave the working tree with the intended changes for review.

### Validation Rules

- A green `build` is mandatory. A red `build` means Stage 1 cannot proceed; escalate rather than continue.
- The lint baseline is a **ceiling, not a target**. Later stages may not raise it. Pre-existing errors and newly-introduced errors must be distinguishable from the recorded table.
- `tsc -b` uses project references — if it succeeds, `npm run build` cannot fail on typecheck alone.

---

## Edge Cases

- **`pnpm-lock.yaml` turns out to be tracked.** `git status` currently shows it as untracked (`??`). If it is in fact tracked in some branch or is referenced by CI, deleting it would change the install contract. Check `git ls-files --error-unmatch pnpm-lock.yaml` before deleting. If tracked, stop and escalate — the package manager of record may be pnpm, not npm, which would change every validation command in the plan.
- **`pnpm-workspace.yaml` is legitimate, not an artefact.** Its content is `allowBuilds:\n  esbuild: set this to true or false` — clearly generated by pnpm's `approve-builds` flow. If a maintainer has since hand-edited it into a real workspace definition, deleting it silently converts a single-package repo into a different install topology. Confirm the content matches the known generated string before deleting.
- **Removing the self-dependency does not fix the install.** `ERR_PNPM_IGNORED_BUILDS` is reported as an *environment policy* (pnpm blocks dependency build scripts by default; `esbuild@0.25.12` was not approved). If the install still fails after W1.1 + W1.2, do **not** add `pnpm.onlyBuiltDependencies`, do not switch package managers, and do not vendor `esbuild`. Record the failure verbatim and escalate — the fallback is manual verification (`npm run lint` / `npm run build` through an approved runner) rather than a project change.
- **Lint has a large number of pre-existing errors.** Possible — `feasibility.md` predicts at least `react-hooks/exhaustive-deps` at `HomePage.tsx:70` and `react-refresh/only-export-components` at `MenuBar.tsx:15`. Record the exact count. Do **not** fix them here; `W2.16` (TASK-13) will add plugins that surface more, and each set must be baselined separately.
- **`tsc -b` fails on a pre-existing JSX prop-contract hole.** `feasibility.md` predicts this may surface at `HomePageScroll.tsx:123-131` (a `Component` union including `ContactForm: () => JSX.Element` being passed `cardType`/`image`/`title`/`description`). This is a *typing hole*, not a proven compile error. If it does fail, record it — the real fix is W2.3's typed registry in TASK-09, which is Stage 2. Do not widen Stage 1.
- **A build artefact directory is now dirty.** `dist/` or `*.tsbuildinfo` may appear after the first successful build. Confirm they are gitignored so they do not pollute the baseline `git status`.
- **Deleting the lockfile changes installed versions.** If a fresh install resolves different patch versions than `node_modules` currently holds, the recorded baseline reflects the *new* resolution. Note the resolved versions in `feasibility.md` so later stages can tell a real regression from a version drift.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7). Adding one is explicitly out of scope.

### Integration Tests

- None.

### Manual Validation

- [ ] `npm run lint` → record exact error and warning counts.
- [ ] `npm run build` → confirm `dist/` is produced and the process exits 0.
- [ ] `npx tsc -b` → confirm exit 0.
- [ ] `git status --short` → confirm only `?? .gitmodules` and `?? .opencode/` (plus intended `package.json` modification) appear.
- [ ] `git diff package.json` → confirm exactly one deleted line.
- [ ] Open `feasibility.md` and confirm the three-line baseline block matches the captured output verbatim.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Verification is manual. Cost note: manual lint + build ≈ 30–60s per run; every subsequent task in the plan must run both, so budget ~1 minute per task for validation.

---

## Definition of Done

- [ ] `package.json` self-dependency removed
- [ ] `pnpm-workspace.yaml` + `pnpm-lock.yaml` deleted
- [ ] `npm run lint` baseline recorded verbatim in `feasibility.md`
- [ ] `npm run build` passes and is recorded
- [ ] `npx tsc -b` result recorded
- [ ] `git status` clean of the two pnpm artefacts
- [ ] Zero `src/` files modified
- [ ] Build passes
- [ ] Lint does not regress above the recorded baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.1, W1.2, W1.3; Stage 1 entry/exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — § *Toolchain Baseline*, § *Baseline recording requirement*
- `.opencode/plan/codebase-quality-seo/risks.md` — R5, D2, D8
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 0 file overlap map
- `package.json:14`