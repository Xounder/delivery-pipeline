# Task — Route-Level Code Splitting & `LazyMotion` Async Features

## Task Information

### ID

TASK-22

### Title

Lazy-load the `/scroll` route with `React.lazy` + `Suspense`, and move `framer-motion` to `LazyMotion` with async `domAnimation` + `m` components

### Owner

senior-frontend

### Status

Source implementation complete; browser acceptance NOT VERIFIED (browser use prohibited for this run).

### Stage

Stage 4 — SEO & Performance

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W4.12** | Medium | Lazy-load the `/scroll` route with `React.lazy` + `Suspense` and a `main.tsx`-level fallback | `src/main.tsx:4-19` |
| **W4.13** | Medium | Use `framer-motion`'s `LazyMotion` with async-loaded `domAnimation` features; prefer `m` components over `motion` | `src/pages/HomePage/HomePage.tsx:19`, `src/components/HomePageScroll/HomePageScroll.tsx:12` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, **no animation timing**, no data, no link target, no form semantics change. **This task is the highest-risk item in the plan for invariant 1.** Splitting a route changes load order; switching to `LazyMotion` changes the animation engine's initialisation path. **Every animation must still run, with identical timing.**
2. **Visual branding** — rendered output must be pixel-identical. **A missing Suspense fallback or a `LazyMotion` feature that fails to load renders a visibly different page.** No fallback may persist after load.
3. **Copy** — unchanged.
4. **Routing** — `/` and `/scroll` only, `vercel.json` untouched. **Direct navigation to `/scroll` must still work** — this is the exact scenario lazy loading breaks.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **The `m` component renders the same DOM as `motion`; verify `data-section` and the class names survive.**

**Why this is last (ordering):**
- `impact-analysis.md`: *"`main.tsx` is on the **critical path**. The bundle is 258KB. `motion` from framer-motion accounts for 34KB. But HomePageScroll is 11KB and is a separate route."*
- `feasibility.md`: *"Interactive animations can silently stop. Verify in a real browser, not by build output."*

---

## Description

The final performance item, combining the two bundle-size work items into one coherent change.

**W4.12 — route-level code splitting.** `main.tsx:4-19` statically imports both route components. `HomePageScroll` is 11KB and only reachable at `/scroll`. Splitting it out of the `/` bundle is the highest-value split available, and it requires no behaviour change.

**W4.13 — `LazyMotion`.** `framer-motion`'s full `motion` component pulls in the entire animation engine (34KB of the 258KB bundle). `LazyMotion` with async-loaded `domAnimation` features defers that, and lets JSX use the lighter `m` component. `impact-analysis.md` flags a **non-performance risk**: *"Risk that animations silently stop working."*

---

## Acceptance Criteria

- [ ] `HomePageScroll` is loaded via `React.lazy(() => import(...))`.
- [ ] A `Suspense` boundary wraps the routed content in `main.tsx`.
- [ ] **The fallback is minimal and transient** — a neutral placeholder, not a spinner over a flash of wrong content.
- [ ] The `/` route's bundle no longer includes `HomePageScroll`.
- [ ] **Direct navigation to `/scroll` works** (fresh load, no client-side navigation first).
- [ ] Client-side navigation `/` → `/scroll` works, and `/scroll` → `/` works.
- [ ] **`framer-motion`'s `LazyMotion` wraps the app** with `features={domAnimation}` loaded asynchronously.
- [ ] JSX uses `m` (or `LazyMotion`'s `m` export) instead of `motion` where the plan directs it.
- [ ] **`strict` mode is set** if the plan directs it — do not guess; record the decision.
- [ ] **Every animation still runs, with identical timing.** Verified in a real browser, both routes.
- [ ] No animation is silently disabled — specifically: the entrance transitions on `/`, the arrow transitions, the `MenuBar` fade, and the `/scroll` progressive reveal (OOS-2 — unchanged).
- [ ] `data-section` attributes and all class names intact after the `motion` → `m` switch.
- [ ] The IntersectionObserver and `content-visibility` behaviour on `/scroll` unchanged.
- [ ] No console errors or unhandled promise rejections on either route.
- [ ] Screenshot diff clean at all 13 widths, both routes, **after animations settle**.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.
- [ ] Bundle size reduction is measured and recorded.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-13
- TASK-14
- TASK-15
- TASK-17

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run. **Bundle-size measurement requires a working build.**
- **TASK-14** — the **hard gate**. R6.
- **TASK-13** — **technically unavoidable**: W2.6/W2.17 rewrote the route tree in `main.tsx` and added an `ErrorBoundary`. **This task rewrites the same route tree.** The boundary must survive the `Suspense` conversion — a lazy-load failure must be caught. Sequential; read `main.tsx` after TASK-13.
- **TASK-15** — `HomePage.tsx` / `HomePageScroll.tsx` imports; establishes the CSS/structure this task preserves.
- **TASK-17** — W3.10 touched `HomePage.tsx`'s and `HomePageScroll.tsx`'s transition objects, and W3.6 changed element names. **The `motion` → `m` switch must operate on TASK-17's post-extraction variants.** Sequential. Also TASK-21 owns `HomePageScroll.tsx:76` — S4-A must finish before this task starts.

**Parallelism:** S4-C alongside TASK-20. `main.tsx` / `HomePage.tsx` / `HomePageScroll.tsx` vs `AboutMe` / `DefaultCard` / `MenuBar` / `DefaultSectionButton` / `ContactForm` / `ExperienceCardInfo` / `data.ts` — **completely disjoint.** Per `impact-analysis.md` Tier 0, parallel-safe.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/main.tsx` | 3 | `:4-19` (W4.12) | TASK-08 (W1.18, done), **TASK-13** (W2.6, W2.17 `:22-50`) | **Sequential.** Ledger: TASK-08 → **TASK-13** → **TASK-22**. **`:4-19` vs TASK-13's `:22-50` are adjacent — a merge can silently drop the `ErrorBoundary` or the `Suspense` wrapper. Verify both survive.** |
| `src/pages/HomePage/HomePage.tsx` | **13 — the plan's hottest file** | `:19` (W4.13 `LazyMotion`/`m`) | TASK-02, TASK-07, **TASK-10** (`:23,28,55-67,102,126,142-149,165-172`), **TASK-17** (`:38,52,77,97,124,135,136,161`) | **Sequential.** Ledger: TASK-02 → TASK-07 → **TASK-10** → **TASK-17** → **TASK-22**. **`:19` is the import line TASK-10 and TASK-17 both edited.** |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | **9** | `:12` (W4.13) | TASK-03, TASK-07, TASK-08, TASK-10, **TASK-17** (`:92,117,143`), **TASK-21** (`:76`) | **Sequential.** Ledger: … → **TASK-17** → **TASK-21** → **TASK-22**. **This file has 9 work items and 4 of them land in Stage 4.** |
| `src/index.css` | 4 | — (not touched) | TASK-02, **TASK-16**, **TASK-19**, **TASK-21** | No overlap. **`LazyMotion` styles, if any, belong in a component stylesheet — confirm before adding CSS.** |
| `vercel.json` | 1 | — (not touched) | **TASK-21** | No overlap. |

---

## Technical Context

### Relevant Components

- `main` — the route tree, `ErrorBoundary`, and the new `Suspense` boundary
- `HomePage` — `motion`/`m` imports, entrance transitions, arrow transitions
- `HomePageScroll` — `motion`/`m` imports, section reveal transitions

### Relevant Modules

- `src/main.tsx:4-19, 22-50`
- `src/pages/HomePage/HomePage.tsx:19`
- `src/components/HomePageScroll/HomePageScroll.tsx:12`

### Relevant APIs

- `React.lazy()` + dynamic `import()`
- `React.Suspense` with a `fallback`
- `framer-motion`: `LazyMotion`, `domAnimation` (async features), `m` component, `strict`
- Vite code splitting / dynamic import chunking
- Bundle-size measurement (`vite build` output, `rollup` chunk report)

### Relevant Types

- `LazyMotion` `features` prop (a promise or a synchronous feature bundle)
- `m` component props (identical to `motion`'s minus the internal engine)
- React `ComponentType` for the lazy route element

---

## Implementation Guidance

### Expected Changes

1. **W4.12** — convert `HomePageScroll` to a dynamic import wrapped in `React.lazy`. Add a `Suspense` boundary in `main.tsx` around the routed content, **outside** the `ErrorBoundary` so both catch failures. Provide a minimal fallback.
2. **W4.13** — wrap the app (or the animated subtree) in `LazyMotion` with `domAnimation` loaded asynchronously. Replace `motion` with `m` at the import sites the plan directs.

### Constraints

- **W4.13 has no perf benefit if the features are loaded synchronously.** `LazyMotion` only saves bytes when `domAnimation` is itself dynamically imported. Loading it eagerly defeats the purpose.
- **`LazyMotion` `strict` mode is stricter about animating unsupported values.** Read the current transition objects (TASK-17's extracted variants) before enabling it; some easing/transition combinations are rejected in strict mode. **Do not enable `strict` blindly.**
- **Do not change any animation's duration, delay or easing.** Invariant 1. The `m` component applies the same values; the risk is a *silent* failure to animate, not a different duration.
- **Direct `/scroll` navigation must work.** `vercel.json`'s rewrite serves `index.html`, and React Router then lazy-loads the route. **This is the scenario most likely to regress.**
- **Do not remove TASK-13's `ErrorBoundary`.** A failed dynamic import must be caught. Keep the boundary outside `Suspense`.
- **Do not add a spinner that persists.** The fallback must be transient and visually neutral.
- **The fallback must not cause a layout shift on `/`.** If `/` is not lazy-loaded, `/` never shows the fallback — **only lazy-load `/scroll`, not both routes.** `feasibility.md` and `impact-analysis.md` both scope the split to `HomePageScroll`.
- **Do not change the IntersectionObserver or `content-visibility` on `/scroll`.** TASK-03 and TASK-21 own those.
- **Do not touch `vercel.json`.** W4.11 is TASK-21.
- **Do not touch `index.css`.** If `LazyMotion` needs a style, use an existing component stylesheet.
- **Do not combine `m` with `layout`/`drag` gestures** that require the full feature set — `domAnimation` excludes layout-projection features. Audit the components for `layout`, `drag`, `useAnimate` usage.

### Validation Rules

- **Verify animations in a real browser, not by build output.** `feasibility.md` is explicit: *"Interactive animations can silently stop. Verify in a real browser, not by build output."*
- **Walk the animation inventory explicitly:** `/` entrance, `/` arrow transitions, `MenuBar` fade, `/scroll` progressive reveal. Each must still animate.
- **Compare timings numerically** against TASK-17's recorded table.
- **Load `/scroll` directly in a fresh tab** (not via client-side navigation) — this is the regression case.
- **Check the console** for errors, warnings and unhandled rejections on both routes.
- **Verify `data-section` and class names** after the `motion` → `m` switch.
- **Measure bundle sizes** before and after (`vite build` output); record the reduction.
- Screenshot diff at all 13 widths **after animations settle** — a screenshot taken mid-animation produces a false diff.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **Direct `/scroll` navigation is the classic lazy-loading break.** If the rewrite or the router base path is wrong, the chunk request 404s and the page is blank. **Test a hard reload on `/scroll` first.**
- **A `Suspense` fallback that flashes on every `/scroll` visit is a visible regression.** The chunk is cached after the first load — verify the second navigation shows no fallback.
- **`LazyMotion`'s async features create a window where animations cannot run.** If a component mounts before the features resolve, its animation may be skipped permanently. **This is `impact-analysis.md`'s named risk: "animations silently stop working."** Verify the very first paint of both routes.
- **React 18 `StrictMode` double-mounts** in dev, which can mask or exaggerate the features-loading race. **Verify in the production build.**
- **`strict` mode rejects some transition objects** that non-strict mode silently accepts — a thrown error mid-animation. Audit all of TASK-17's extracted variants before enabling it.
- **`domAnimation` does not include layout-projection features.** Any `layout`, `layoutId`, `drag` or `useAnimate` usage needs `domMax` or the full feature set instead. **Grep for these before switching.**
- **The `m` component cannot be used outside a `LazyMotion` boundary.** Any component that renders `m` must be a descendant of the provider — including one rendered through a portal or a lazily-loaded route mounted outside the boundary.
- **TASK-13's `ErrorBoundary` placement.** If `Suspense` is placed inside the boundary, a rejected dynamic import shows the error UI; if outside, it is uncaught. **Verify the intended behaviour for a failed chunk load** and record it.
- **A `content-visibility: auto` section (TASK-21's note) inside a lazy route** interacts with the IntersectionObserver — `content-visibility` skips rendering for off-screen content, and `IntersectionObserver` still fires. Verify the reveal still triggers for every section.
- **The scroll-restoration fake `<div>` (OOS-3) and its 500ms timer** run on `/`. Lazy-loading `/scroll` must not change `/`'s timing.
- **The progressive reveal (OOS-2)** on `/scroll` depends on the route mounting and the observer firing in order. If `HomePageScroll` mounts later (lazy), the observer initialises later — **verify sections reveal in the same order and at the same scroll positions.**
- **Bundle-size claims can be misleading.** `LazyMotion` may reduce the initial chunk but add a shared vendor chunk. Measure the **`/` initial payload**, which is the metric that matters, and record it honestly even if the total is unchanged.
- **`data-section` attributes drive section activation** (W2.12's DATA_SECTIONS). The `motion` → `m` switch must not alter attribute rendering.
- **Suspense boundaries and `AnimatePresence` exit animations**: if `HomePageScroll` unmounts on navigation, its exit animation may be skipped by a `Suspense` fallback appearing first. Verify transitions between routes.

---

## Testing

### Source-only validation (retry attempt 2)

- `npm run build` succeeds (baseline and after). `npm run lint` still reports 6 errors / 0 warnings in `ContactForm` (4) and `MenuBar` (2); these belong to parallel TASK-20, not TASK-22. No test script is configured.
- Current baseline build immediately before TASK-22 (not the historical 258 kB estimate): single initial JS `index-BkR0F8La.js` 338.48 kB raw / 117.75 kB gzip; initial CSS `index-DncmCBFR.css` 20.16 kB raw / 4.16 kB gzip. No static JS dependencies besides that index chunk.
- After: initial JS static graph `index-COKxhtoY.js` 242.27 kB raw / 87.44 kB gzip; initial CSS `index-Dmalq_0z.css` 18.94 kB raw / 4.00 kB gzip. JS reduction 96.21 kB raw / 30.31 kB gzip; JS + CSS reduction 97.43 kB raw / 30.47 kB gzip. Dynamic chunks (not in initial static graph): `HomePageScroll-C5CY47la.js` 2.54 kB raw / 1.12 kB gzip, `HomePageScroll-BLd72eg4.css` 1.37 kB raw / 0.49 kB gzip, `domAnimationFeatures-DUompC8l.js` 53.27 kB raw / 18.62 kB gzip. Feature import begins asynchronously at app startup; these numbers are *static entry graph*, not a claim that features never download on `/`.
- Only `/scroll` uses `React.lazy`, with a null minimal Suspense fallback. `ErrorBoundary` remains outside `Suspense` and the motion provider. Source inspection confirms unchanged section element names, `data-section`, observer threshold 0.5 and registration, variants and transition values (home 0.5 s, scroll header 0.4 s easeOut, reveal 0.6 s easeOut); home page readiness 100 ms and fake-div cleanup 500 ms remain unchanged. `motion` JSX replaced with `m` in both pages; no `layout`, `layoutId`, `drag`, or `useAnimate` usage found in source. `strict` enabled: only `m` components are used and no layout/drag features required. The installed `LazyMotion` version installs features in an effect; `MotionReady` gates route mounting until feature installation and forwards import failures to `ErrorBoundary` (without an unhandled rejection from LazyMotion's missing rejection handler).
- NOT VERIFIED: production-browser first paint, all animations including arrow/MenuBar, direct `/scroll` hard reload, round-trip navigation, console, rendered DOM attributes, screenshot diffs at 13 widths, and network-failure handling in browser. TASK-14 was skipped and browser/GUI/headless/CDP/test servers are prohibited in this run; these remain manual acceptance gates, not claimed passing.

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] `vite build` output shows a separate `HomePageScroll` chunk; record sizes before/after.
- [ ] Hard reload `https://<domain>/scroll` in a fresh tab → page renders, no console error.
- [ ] Client-side navigation `/` → `/scroll` → `/` → `/scroll` → all render; **second `/scroll` visit shows no fallback**.
- [ ] Animation inventory, first paint of each route:
  - [ ] `/` entrance transitions run
  - [ ] `/` arrow section-button transitions run
  - [ ] `MenuBar` fade runs, with the 500ms window intact
  - [ ] `/scroll` progressive reveal runs, in order, at the same scroll positions (OOS-2 unchanged)
- [ ] Compare animation timings against TASK-17's recorded table → identical.
- [ ] Console: zero errors, zero unhandled rejections, on both routes, in the production build.
- [ ] Grep for `layout`, `layoutId`, `drag`, `useAnimate` → confirm `domAnimation` covers what is used.
- [ ] Verify `data-section` attributes and class names intact on both routes.
- [ ] Screenshot diff at all 13 widths, both routes, after animations settle.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. **This task has the highest manual-verification cost in the plan**: the animation inventory plus direct-navigation tests plus bundle measurement ≈ 1 hour. Record it as the plan's most expensive task.

---

## Definition of Done

- [ ] `HomePageScroll` lazy-loaded; `/` bundle excludes it
- [ ] `Suspense` boundary added in `main.tsx` with a minimal, transient fallback
- [ ] Direct `/scroll` navigation works; repeat navigation shows no fallback
- [ ] TASK-13's `ErrorBoundary` preserved and positioned to catch a failed chunk load
- [ ] `LazyMotion` + async `domAnimation` in place; `m` components used where directed
- [ ] Every animation verified running in a real browser, with identical timing
- [ ] `strict` mode decision recorded and justified
- [ ] `data-section` and class names intact; IntersectionObserver and `content-visibility` behaviour unchanged
- [ ] Progressive reveal unchanged (OOS-2); scroll-restoration timing unchanged (OOS-3)
- [ ] No console errors on either route in the production build
- [ ] Bundle-size reduction measured and recorded for the `/` initial payload
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes; `npm run lint` green
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W4.12, W4.13; Stage 4 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 row for W2.6/W2.17, Stage 4 rows for W4.12/W4.13; § *Bundle & Main Thread Rationale*; the real-browser verification requirement
- `.opencode/plan/codebase-quality-seo/risks.md` — R7, T5, OOS-2, OOS-3, § *Non-performance risks introduced*
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — § *Bundle & Main Thread* (258KB bundle, 34KB `motion`, 11KB `HomePageScroll`); § *Verification Plan*; § *Expected Changes* (`main.tsx`)
- `src/main.tsx:4-19, 22-50`
- `src/pages/HomePage/HomePage.tsx:19`
- `src/components/HomePageScroll/HomePageScroll.tsx:12`
