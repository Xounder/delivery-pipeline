# Task — Error Boundary, Catch-All Route & Tooling Corrections

## Task Information

### ID

TASK-13

### Title

Add an error boundary and a `*` catch-all route; correct router types, add lint plugins, relocate the `Icon` type

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 2 — Organisation, DRY & KISS

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W2.6** | High | Add an error boundary and a `*` catch-all route | `src/main.tsx:8-15`, new `src/components/ErrorBoundary/` |
| **W2.15** | Low | Install the correct router types: `@types/react-router-dom@^5.3.3` is v5 types against `react-router-dom@^7` | `package.json:17,23` |
| **W2.16** | Low | Add `eslint-plugin-react` and `eslint-plugin-jsx-a11y` to `eslint.config.js` | `eslint.config.js:16-26` |
| **W2.17** | Low | Fix the `react-refresh/only-export-components` warning by moving the `Icon` type out of the component module | `src/components/MenuBar/MenuBar.tsx:15-19` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **`/` and `/scroll` must behave exactly as before.**
2. **Visual branding** — rendered output must be pixel-identical. The error boundary and catch-all render only where nothing rendered before, so there is no prior appearance to preserve — but the boundary must **not** wrap content that renders successfully.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged. **The boundary's and the catch-all's own fallback text is new UI that only appears in previously-blank states — that is additive, not a copy change.**
4. **Routing** — `/` and `/scroll` only, `vercel.json`'s SPA rewrite untouched. **Invariant 4 explicitly permits a `*` catch-all route because it *replaces a blank page*.** No other route may be added or removed.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **`#root` is the boundary's critical boundary — the boundary must not be placed inside a way that changes `#root`'s identity or position in the tree.**

---

## Description

Four infrastructure items that fix the app's two worst failure modes (blank page on a bad URL, blank page on any render error) plus two tooling corrections that make similar defects visible in future.

**W2.6 — error boundary + catch-all.** Today `vercel.json` rewrites `/(.*)` → `/index.html`, so `/typo` matches no route in `main.tsx:10-13` and renders **nothing at all**. Any render error is also a blank page. This is risk **O1** and **O2** — and O2 is also the single largest SEO risk, because an error means a crawler receives an empty page.

The catch-all **replaces a blank page**, so it is an improvement, not a routing change (invariant 4's own note). The boundary must render a **visible** fallback **and** log to console in development (`risks.md` R17).

**W2.15 — stale router types.** `@types/react-router-dom@^5.3.3` provides v5 types against `react-router-dom@^7`. Either migrate to the v7 types package or remove the stale `@types` and rely on the bundled types.

**W2.16 — lint plugins.** `eslint-plugin-react` and `eslint-plugin-jsx-a11y` enforce `react/jsx-key` and the a11y rules. This is *why* the index-key defects (W2.8) and the a11y defects were invisible. **Expect many pre-existing violations** (`risks.md` R12, `feasibility.md` W2.16) — they must be baselined separately, not silently fixed.

**W2.17 — `Icon` type export.** Exporting a type alongside a component triggers `react-refresh/only-export-components`. Move the type to its own module.

---

## Acceptance Criteria

- [ ] An error boundary exists, is mounted, and renders a **visible** fallback when a descendant throws.
- [ ] The boundary **logs** caught errors to the console in development. R17.
- [ ] `/typo` (and any unmatched path) renders the catch-all fallback instead of a blank page.
- [ ] Unmatched paths are **logged**. R18 — the catch-all must not mask genuine routing bugs.
- [ ] `/` and `/scroll` render and behave **identically** to before. `vercel.json`'s SPA rewrite is untouched.
- [ ] A deliberate test render of a throwing component is caught and the fallback appears; the instrumentation is then removed.
- [ ] `#root`'s identity and position in the DOM tree are unchanged.
- [ ] `@types/react-router-dom@^5.3.3` is either removed (relying on v7's bundled types) or replaced with the correct v7 types package. Recorded which.
- [ ] `tsc -b` passes immediately after the router-types change, with no new suppression comments.
- [ ] `eslint-plugin-react` and `eslint-plugin-jsx-a11y` are added to `eslint.config.js` and active.
- [ ] **The new lint plugins' output is recorded as a separate baseline** in `feasibility.md` — separate from the TASK-01 baseline. Pre-existing violations are not Stage 2 regressions.
- [ ] The `Icon` type no longer lives in `MenuBar.tsx`; `react-refresh/only-export-components` at `MenuBar.tsx:15` is resolved.
- [ ] No runtime dependency is added. Only dev dependencies.
- [ ] Screenshot diff clean at all 13 widths for `/` and `/scroll`.
- [ ] `npm run build` passes; `npm run lint` does not regress **above the Stage 2 entry baseline** (which includes the new plugins).

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-10
- Stage 1 exit criteria

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run. TASK-01 also owns `package.json` in Stage 1; this task's W2.15 edit is Stage 2, so no conflict.
- **Stage 1 exit** — `index.md` Stage 2 entry criteria.
- **TASK-10** — owns `MenuBar.tsx:8-13,40,61` for W2.11. This task edits `MenuBar.tsx:15-19` for W2.17. **Same file → sequential**, though the line ranges do not overlap.

**Parallelism note:** this task runs in Stage 2 batch S2-B alongside TASK-11. Their file sets are disjoint.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/main.tsx` | 2 (W2.6, W4.12) | `:8-15` | **TASK-22** (W4.12 `:5-6`) | **Sequential** — Stage 4. TASK-22 rewrites the route tree for lazy loading; the boundary and catch-all must survive that rewrite. |
| `src/components/MenuBar/MenuBar.tsx` | 5 | `:15-19` | TASK-05, TASK-07, TASK-08, **TASK-10** (`:8-13,40,61`), **TASK-12** (`:35,40,61`), TASK-17, TASK-20 (`:29,32`) | **Parallel-safe with non-overlapping sections.** `:15-19` (type export) does not overlap TASK-10's `:8-13,40,61` or TASK-12's `:35,40,61`. TASK-12 runs **after** this task (batch S2-C). **If in doubt, serialise** — this file has been edited 4 times already. |
| `package.json` | 2 (W1.1, W2.15) | `:17,23` | TASK-01 (done) | **Sequential.** |
| `eslint.config.js` | 1 | `:16-26` | — | **No overlap.** This is the plan's only task touching `eslint.config.js`; it is fully parallel-safe with every other task. |
| `src/components/ErrorBoundary/` *(new)* | 1 | created here | — | **No overlap.** |

**Tier 0 files** (`package.json`, `eslint.config.js`, `vercel.json`, `index.html`, `public/*`) have **no overlap with Tier 1+** per `impact-analysis.md`. `eslint.config.js` in particular can be dispatched as its own parallel instance if preferred.

---

## Technical Context

### Relevant Components

- `ErrorBoundary` (new) — catches render errors
- `main.tsx` — router setup, `createRoot`, route tree
- `MenuBar` — exports the `Icon` type alongside a component

### Relevant Modules

- `src/main.tsx:8-15`
- `src/components/ErrorBoundary/` (new — `ErrorBoundary.tsx`)
- `package.json:17` (`react-router-dom`), `:23` (`@types/react-router-dom`)
- `eslint.config.js:16-26`
- `src/components/MenuBar/MenuBar.tsx:15-19`

### Relevant APIs

- React error boundaries (class components with `componentDidCatch` / `getDerivedStateFromError`)
- `react-router-dom` v7 — `createBrowserRouter` or `<Routes>`/`<Route>`, and the `*` catch-all
- `vercel.json`'s `rewrites` — the SPA fallback that causes `/typo` to reach `index.html`
- ESLint flat config, `eslint-plugin-react`, `eslint-plugin-jsx-a11y`
- `react-refresh/only-export-components`

### Relevant Types

- `Icon` — moving out of `MenuBar.tsx:15-19`

---

## Implementation Guidance

### Expected Changes

1. **W2.6** — create `src/components/ErrorBoundary/ErrorBoundary.tsx`. `feasibility.md` permits *"Class component or a small library-free boundary."* A class component is the simplest option and requires no dependency.
   - Mount it in `main.tsx` **inside** `createRoot` but **outside** the router, so it catches router-level and page-level errors.
   - Render a visible fallback. Log the error to console in development (`R17`).
   - Add a `*` catch-all route rendering a visible "not found" fallback. Log the unmatched path (`R18`).
2. **W2.15** — check whether `react-router-dom@7` ships its own types. If yes, remove `@types/react-router-dom`. If no, install the correct v7 types. **Run `tsc -b` immediately** — R11 warns that v5→v7 differences around `Route`/`Routes` may require adjustments.
3. **W2.16** — add `eslint-plugin-react` and `eslint-plugin-jsx-a11y` to `eslint.config.js`, wire up `react/jsx-key` and the relevant a11y rules. **Record the new baseline immediately.**
4. **W2.17** — move the `Icon` type into its own module (e.g. `src/data/types.ts` from TASK-09, or a sibling file). Update importers.

### Constraints

- **Do not change `vercel.json`'s SPA rewrite.** Invariant 4 and R18.
- **Do not add a `BrowserRouter`/router provider that changes how `/` and `/scroll` resolve.** Verify both routes after the change.
- **Do not place the boundary inside `#root`'s children in a way that changes the tree shape.** `#root` is an invariant-5 contract.
- **Do not add a runtime dependency** for the error boundary. No `react-error-boundary` package.
- **Do not silently fix pre-existing lint violations** surfaced by W2.16. Baseline them (R12, `feasibility.md`).
- **Do not fix the errors the new plugins surface in the same change.** That would make Stage 2 look like it regressed.
- **Do not enable rules so strict that they fail the build on pre-existing code.** If `react/jsx-key` surfaces W2.8's index keys, that is the correct signal — but TASK-12 already fixed those, so verify the ordering holds.
- **Do not upgrade `react-router-dom`.** Only the *types* change. `feasibility.md`: *"Runtime — unchanged. No runtime package is added, removed or upgraded anywhere in the plan."*
- Do not change the `Icon` type's *shape*, only its location.

### Validation Rules

- The boundary must be **tested by deliberately throwing** from a component, confirming the fallback renders, then reverting the instrumentation.
- The catch-all must be **tested by navigating** to `/typo`, `/scroll/extra` and `/anything`.
- `tsc -b` must pass **immediately** after the router-types change (R11).
- The lint baseline **must be recorded before** any plugin-driven fix, and the two baselines (pre-plugin and post-plugin) kept separate.
- `react-refresh/only-export-components` must be **absent** from the lint output for `MenuBar.tsx`.
- Screenshot diff at all 13 widths for `/` and `/scroll` — expect zero diff.
- `npm run build` passes.

---

## Edge Cases

- **The boundary itself throws.** R17's worst case: a boundary that throws during `render` cannot catch itself and the whole tree unmounts to a blank page. Keep the fallback minimal — no data lookups, no hooks that can fail, no context reads. A fallback that depends on context is a boundary that can fail.
- **The boundary catches errors that were previously visible in the console.** Errors that used to surface loudly now render a fallback. That is the intended improvement, but it can hide a regression during Stage 2/3 verification. R17: log in development. Confirm the console still shows the error.
- **The catch-all masks a genuine routing bug.** R18. `/scroll` reached via a typo like `/scrolll` would render "not found" instead of failing loudly. Logging the unmatched path is the mitigation — and it must actually be visible in development.
- **The catch-all could catch a legitimate asset request.** With `vercel.json` rewriting `/(.*)` → `/index.html`, a request for a missing JS/CSS asset also reaches `index.html`. If the router matches `*`, a missing chunk request renders the catch-all instead of a 404. Verify asset 404s still behave sensibly.
- **Lazy chunks and the catch-all interact (W4.12, TASK-22).** If TASK-22 lazy-loads routes with `React.lazy`, a failed chunk fetch throws during render and lands in the boundary — showing the error fallback instead of the catch-all. That is the correct behaviour, but verify it reads sensibly.
- **The boundary changes the accessibility tree in error states only.** No impact on the success path.
- **`getDerivedStateFromError` is static.** It cannot access props or context. The fallback must be a constant, or derived from state set in `componentDidCatch`.
- **`window.location` in the catch-all is a module-scope hazard.** W4.14 (TASK-19) requires **no module-scope `window`/`document` access** for prerender-readiness. Reading the unmatched path inside the component (or via the router's location) is fine; reading it at module scope is not.
- **Router v5→v7 type differences (R11).** `Route`/`Routes` generics differ. `tsc -b` may surface errors in `main.tsx` and `MenuBar.tsx`. These are **compile-time only, no behaviour change** — fix them properly rather than with `@ts-ignore`.
- **Removing `@types/react-router-dom` when v7 does *not* bundle types** leaves the app untyped and `tsc -b` may pass with implicit `any`. Check the package's `types`/`exports` field before deciding which branch to take.
- **`eslint-plugin-react` needs `settings.react.version`.** Without it, some rules misfire on React 18. Configure it explicitly.
- **`jsx-a11y` will surface many pre-existing violations** — missing labels (W4.8), landmark misuse (W3.6), heading order (W1.17), decorative images (W1.20). **Several are already fixed by Tasks 6–8, 17 and 20.** Verify against the current state, not the audit's original list, and baseline whatever remains.
- **`eslint-plugin-react` must not enable `react/prop-types`** — the project is TypeScript, and that rule produces hundreds of false positives. Enable only the rules that matter: `react/jsx-key` above all.
- **Moving the `Icon` type** requires updating every importer. Grep for `Icon` across `src/`. `iconsData.ts` is the likely source and may need to export it.
- **The lint config change may fail `npm run lint` outright** if a plugin's default config is applied wholesale and errors on existing code. Configure rules explicitly, not by extending a recommended set blindly.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Navigate to `/typo`, `/scroll/extra`, `/anything` → each renders the catch-all, not a blank page; each path logged.
- [ ] Temporarily throw from a component → boundary fallback renders visibly, error logged to console; then revert.
- [ ] `/` and `/scroll` render and behave identically; `vercel.json` untouched (diff it).
- [ ] `npm run build` after the router-types change → passes with zero type errors.
- [ ] `npm run lint` after the plugin addition → **record the new baseline count** in `feasibility.md`, separate from TASK-01's.
- [ ] Confirm `react-refresh/only-export-components` is gone for `MenuBar.tsx`.
- [ ] Confirm no runtime dependency was added (`git diff package.json` shows devDependencies only).
- [ ] Screenshot diff at all 13 widths for `/` and `/scroll`.
- [ ] `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; ~10 min for the routing/boundary tests and lint baselining.

---

## Definition of Done

- [ ] Error boundary exists, mounted, renders a visible fallback, logs in development
- [ ] `*` catch-all route renders a visible fallback and logs unmatched paths
- [ ] `/` and `/scroll` unchanged; `vercel.json` SPA rewrite untouched
- [ ] `#root` tree shape unchanged
- [ ] Router types corrected (removed or v7), `tsc -b` green
- [ ] `eslint-plugin-react` + `eslint-plugin-jsx-a11y` active, new baseline recorded separately
- [ ] `Icon` type relocated; `react-refresh` warning resolved
- [ ] No runtime dependency added
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the Stage 2 entry baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W2.6, W2.15, W2.16, W2.17; invariant 4's catch-all note
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 rows for all four items
- `.opencode/plan/codebase-quality-seo/risks.md` — R11, R12, R17, R18, O1, O2, § *Accessibility Regression Risk* (error boundary entry)
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 0 / Tier 1 overlap maps, § *Breaking Changes*
- `src/main.tsx:8-15`
- `package.json:17, 23`
- `eslint.config.js:16-26`
- `src/components/MenuBar/MenuBar.tsx:15-19`
- `vercel.json:1-5`