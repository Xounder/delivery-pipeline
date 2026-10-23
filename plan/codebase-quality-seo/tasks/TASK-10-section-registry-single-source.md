# Task — Section Registry as the Single Source of Truth

## Task Information

### ID

TASK-10

### Title

Unify the three section lists, extract the arrow arithmetic, hoist the motion constant, derive the route, scope the observer, hoist the element map

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 2 — Organisation, DRY & KISS

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W2.2** | High | Unify the 3 duplicated section lists into one registry | `src/data/sections.ts:1`, `src/pages/HomePage/HomePage.tsx:55-67`, `src/pages/HomePageScroll/HomePageScroll.tsx:104-110` |
| **W2.5** | High | Extract the duplicated arrow-navigation arithmetic into one helper (modulo wrap-around) | `src/pages/HomePage/HomePage.tsx:142-149`, `:165-172` |
| **W2.10** | Medium | Hoist `MOTION_VALUE` to module scope; reconcile `200` vs `100` | `src/pages/HomePage/HomePage.tsx:23,28` |
| **W2.11** | Medium | Fix stringly-typed `atualPage` and mixed `=== '/'` / `== '/'`; derive the destination from the route; make the no-op prop optional | `src/components/MenuBar/MenuBar.tsx:8-13,40,61`, `src/pages/HomePage/HomePage.tsx:102`, `src/pages/HomePageScroll/HomePageScroll.tsx:97-98` |
| **W2.12** | Medium | Replace `document.querySelectorAll("section")` with a scoped container ref | `src/pages/HomePageScroll/HomePageScroll.tsx:79`, `:48` |
| **W2.13** | Medium | Stop rebuilding `componentsMap` on every render | `src/pages/HomePage/HomePage.tsx:55-67,126` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **For W2.5, wrap-around behaviour must be byte-identical.** For W2.11, the navigation destination must be identical.
2. **Visual branding** — rendered output must be pixel-identical.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged. **The 5 section name strings are unchanged.**
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays. W2.11 changes *how* the destination is derived, never *which* routes exist.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **W2.12 changes what is observed, but the `data-section` values and the `section` element names stay exactly as they are.**

**Protected by OOS-2:** `HomePageScroll`'s progressive per-section reveal (`HomePageScroll.tsx:104-135`) — the `visibleSections.has(key)` gate and the `y: 300` animation — is **out of scope**. W2.2 touches `:104-110` only because that inline array is one of the three duplicated section lists; the reveal logic below it must not change.

---

## Description

Six DRY/KISS items, all in the "one source of truth" family, all in the two page components plus `MenuBar` and `sections.ts`.

**W2.2 — three sources of truth for the same 5 strings.** `sections.ts:1`, `HomePage.tsx`'s `componentsMap` keys (`:55-67`), and `HomePageScroll`'s inline array (`:104-110`) each enumerate the same 5 section names. Change one and the other two silently desynchronise. Collapse into one registry exposing **name + icons + render target** so all three sites derive from it. This completes W1.11's Stage 1 fallback into an exhaustive lookup.

**W2.5 — duplicated arrow arithmetic.** `HomePage.tsx:142-149` and `:165-172` are near-identical blocks differing only by `±1`, and both hand-roll wrap-around instead of using modulo. Extract one helper.

**W2.10 — `MOTION_VALUE`.** Declared inside the component body (`HomePage.tsx:28`, value `100`) while `mainMotionValue`'s initial state is `200` (`:23`). Hoist to module scope **and** reconcile the two values. `feasibility.md` Open Question 2 is unresolved, and `risks.md` **R15** is explicit: *"must not silently change first-render animation."* **This task is BLOCKED on that question.**

**W2.11 — stringly-typed route.** `atualPage` is a string compared with `=== '/'` at one site and `== '/'` at another. Derive the destination from the router instead of passing and re-comparing a string. Also remove the no-op `menuSetCurrentSection={() => {}}` by making the prop optional.

**W2.12 — global selector coupling.** `HomePageScroll.tsx:79` observes **every** `<section>` in the document, and `:48`'s `dataset.section || "About Me"` silently mislabels a foreign one. Replace with a scoped container ref. This is risk **T4**.

**W2.13 — rebuilt map.** `componentsMap` is re-created on every render. Hoist it, or render directly from the registry.

---

## Acceptance Criteria

- [ ] `src/data/sections.ts` is the **only** place the 5 section names are declared. Grep confirms zero other literal lists of those 5 strings in `HomePage.tsx` or `HomePageScroll.tsx`.
- [ ] The registry exposes name + icons + render target, and all three former sites derive from it.
- [ ] The section→icon lookup is now **exhaustive**: `sections.indexOf(currentSection)` no longer appears anywhere; an unknown name resolves through a typed path that cannot produce `undefined`.
- [ ] `HomePage.tsx:142-149` and `:165-172` call one shared helper. Wrap-around at both ends is verified for **all 5** sections in both directions.
- [ ] `MOTION_VALUE` is declared at module scope.
- [ ] **The `200` vs `100` reconciliation is recorded as an explicit decision**, not a silent choice. The first-render animation is byte-identical to before (see the constraint below — R15).
- [ ] `atualPage` is no longer a string prop compared for equality. The destination is derived from the router. No `==` loose-equality comparison remains.
- [ ] `menuSetCurrentSection` is optional, and the no-op `() => {}` call site is gone.
- [ ] `HomePageScroll.tsx:79` uses a scoped container ref. A `<section>` outside that container is never observed.
- [ ] `data-section` values are byte-identical.
- [ ] The progressive reveal is unchanged (OOS-2): same `visibleSections` gate, same `y: 300`, same timing, same `threshold: 0.5`.
- [ ] `componentsMap` is not rebuilt on every render (verified by hoisting it to module scope or removing it in favour of registry-driven rendering).
- [ ] Screenshot diff clean at all 13 widths.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-07
- Stage 1 exit criteria

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **Stage 1 exit** — `index.md` Stage 2 entry criteria.
- **TASK-07** (W1.11) — this task *completes* W1.11's Stage 1 safe fallback into the exhaustive registry. `HomePageScroll.tsx` and `MenuBar.tsx` were edited by TASK-07; re-read both before editing. **Sequential, same file.**

**No dependency on TASK-09** — this task and TASK-09 touch disjoint files (`sections.ts`, `HomePage.tsx`, `HomePageScroll.tsx`, `MenuBar.tsx` vs `data.ts`, `DefaultCardGroup.tsx`, `DefaultCardGroup.css`). They run in parallel in batch S2-A. W2.2's registry may consume W2.3's union **if** TASK-09 has landed, but must not require it.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/pages/HomePage/HomePage.tsx` | **13** — the plan's hottest file | `:23,28,55-67,102,126,142-149,165-172` | TASK-07 (`:72-83,101,141,156,163`), TASK-17 (W3.8 `:136`, W3.10 `:23,28,38,52,77,97,124,135`), TASK-22 (W4.12 `:4-19`, W4.13 `:19`) | **Sequential, always.** **High merge risk at `:23,28`**: TASK-17 (W3.10) also touches `MOTION_VALUE` for the duration token. This task's decision must survive that merge. **High merge risk at `:142-149,165-172`**: TASK-17 (W3.10) rewrites the same transition objects. |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | **9** | `:48, 79, 97-98, 104-110` | TASK-03, TASK-07, TASK-08 (all done), TASK-17 (`:92,117,143`), TASK-21 (`:76`), TASK-22 (`:12`) | **Sequential.** |
| `src/components/MenuBar/MenuBar.tsx` | 5 | `:8-13, 40, 61` | TASK-05 (done), TASK-07 (done), TASK-08 (done), TASK-12 (W2.8 `:35`, W2.14 `:40,61`), TASK-13 (W2.17 `:15-19`), TASK-17, TASK-20 (`:29,32`) | **Sequential.** **Highest merge risk in the plan at `:40,61`** — four tasks touch these two lines (this, TASK-12 twice). Apply in order and re-verify after each. |
| `src/data/sections.ts` | 2 | `:1` → full registry | TASK-07 (done) | **Sequential.** This task owns it thereafter. |

**`HomePage.tsx` merge ledger (chronological, verify at each step):**
TASK-02 → TASK-07 → **TASK-10** → TASK-17 → TASK-22

---

## Technical Context

### Relevant Components

- `HomePage` — arrow navigation, section registry consumer, scroll-restoration host
- `HomePageScroll` — inline section array, global observer
- `MenuBar` — receives `atualPage`, calls `setCurrentSection`

### Relevant Modules

- `src/data/sections.ts:1`
- `src/pages/HomePage/HomePage.tsx:23, 28, 55-67, 102, 126, 142-149, 165-172`
- `src/pages/HomePageScroll/HomePageScroll.tsx:48, 79, 97-98, 104-110`
- `src/components/MenuBar/MenuBar.tsx:8-13, 40, 61`

### Relevant APIs

- `react-router-dom` — `useLocation` / route params (the replacement for the `atualPage` string prop)
- `IntersectionObserver` with a scoped root
- `IntersectionObserverEntry.target.dataset.section`
- Ref scoping (`useRef<HTMLElement>` + `root` option)
- `framer-motion` `motion` values (`MOTION_VALUE`, `mainMotionValue`)

### Relevant Types

- The 5 section names — the registry's key type
- `atualPage` — stringly-typed today; to be derived from the route
- `menuSetCurrentSection` — to become optional
- `mainMotionValue` (`200`) vs `MOTION_VALUE` (`100`)

---

## Implementation Guidance

### Expected Changes

1. **W2.2** — expand `sections.ts` into a registry keyed by the section name, each entry exposing the name, its icons, and its render target. Rewrite `HomePage.tsx:55-67` and `HomePageScroll.tsx:104-110` to derive from it. Keep the 5 name strings byte-identical.
2. **W2.5** — extract one helper taking the current index and a delta, returning the wrapped index via modulo. Replace both blocks. Preserve each block's exact behaviour, including which section follows which.
3. **W2.10** — hoist `MOTION_VALUE` to module scope. **Reconcile the `200`/`100` discrepancy only after Open Question 2 is answered, and record the answer.** If unanswered, hoist the constant and leave both values exactly as they are, with a comment pointing at the open question.
4. **W2.11** — remove the `atualPage` string prop; derive the destination from the router (`useLocation` or the route path). Replace the `=== '/'` and `== '/'` comparisons. Make `menuSetCurrentSection` optional and delete the no-op `() => {}` call site.
5. **W2.12** — add a container ref around the scroll page's sections; pass it as the observer's `root` (or scope the query to it). Keep `dataset.section` handling intact.
6. **W2.13** — hoist the element map to module scope, or eliminate it by rendering directly from the registry.

### Constraints

- **Never change a section name string.** Invariant 3 (copy) plus `data-section` (invariant 5). `HomePageScroll.tsx:48`'s `"About Me"` fallback string must remain byte-identical.
- **Do not change the progressive reveal.** OOS-2 — `:104-135` is off limits below the inline array.
- **Do not change `threshold: 0.5`.** W4.17 (TASK-21) documents it.
- **Do not change arrow-navigation order.** W2.5 is a refactor of the arithmetic, not of the sequence. Verify all 5 wrap-around transitions in both directions.
- **Do not change first-render animation timing.** R15. Hoisting `MOTION_VALUE` is safe; reconciling `200` → `100` is a behaviour change and requires the Open Question 2 answer.
- **Do not add a new route** or change the SPA rewrite. Invariant 4.
- **Do not remove `menuSetCurrentSection`'s existing behaviour** where it is actually called — only the no-op call site goes.
- **Do not move DOM elements.** W2.12 adds a ref; it must not wrap sections in a new element that changes layout.
- Do not touch `HomePage.tsx:36-53` (the fake div, OOS-3) or `:72-83` (W1.6, done in TASK-07).

### Validation Rules

- Grep for the 5 section-name literals → they must appear **only** in `sections.ts`.
- Grep for `sections.indexOf` → zero hits.
- Wrap-around test: from each of the 5 sections, `next` and `prev` must produce the correct neighbour. Record all 10 transitions.
- Observer scoping test: inject a `<section>` outside the container (DevTools) and confirm it is never observed and never mislabelled.
- Animation test: compare first-render frame timings before and after — R15 makes this blocking.
- Screenshot diff at all 13 widths is blocking.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **The registry's render target creates a circular import.** If `sections.ts` holds JSX referencing components that import `sections.ts`, the module graph cycles. `feasibility.md` notes the registry must expose "name + icons + render target" — verify the target is a *lazy component reference* (`React.lazy`) or a separate map, not a direct import that cycles. If a cycle is unavoidable, keep `sections.ts` data-only (name + icons) and put the render targets in a sibling module.
- **`sections.indexOf` returning `-1` for a *valid but differently-cased* name.** `"about me"` vs `"About Me"`. A registry keyed by literal name has the same behaviour; the difference is that a `Map` lookup returns `undefined` rather than `-1`, so the failure mode shifts from "negative index" to "missing key". Both must be handled by the same safe path. Do not add case-normalisation as a fix — it would mask a genuine data bug.
- **Modulo with a single-element array.** If the registry ever holds one entry, `index % 1 === 0` always — safe. But `index - 1` on index 0 gives `-1`, and `-1 % 5` is `-1` in JavaScript, **not** `4`. A naive modulo wrap-around is wrong for negative inputs. The helper must normalise: `((index + delta) % length + length) % length`. This is the most likely bug in W2.5.
- **Two arrow blocks may not be exact `±1` mirrors.** `:142-149` and `:165-172` are "near-identical", not identical. One may skip a section, or wrap differently. Diff their behaviour **before** extracting; extracting them into one helper assumes they agree. If they disagree, the disagreement is a finding — record it, and do not silently pick one.
- **`MOTION_VALUE` hoisting changes module evaluation order.** Moving it out of the component body means it is evaluated once at import time instead of per render. If it were ever mutated, this would break. Confirm it is `const` and never reassigned.
- **Reconciling `200` → `100` changes the first-render animation.** R15 forbids doing this silently. The two values produce different `y` offsets on first paint. If the Open Question is unanswered, **hoist and leave both values**. A "cleanup" that picks 100 (or 200) based on which looks better violates invariant 1.
- **Deriving the destination from the route changes `MenuBar`'s contract.** `MenuBar` currently receives `atualPage` as a string and compares it. With `useLocation`, `MenuBar` becomes route-aware. If `MenuBar` renders on a route other than `/` and `/scroll` (it does not today), the derived destination could differ. Verify both routes.
- **The `== '/'` → `=== '/'` change alters behaviour for non-string values.** Today `== '/'` matches only the string `'/'`; `''`, `'/'` with whitespace, or a `String` object would differ. In practice `atualPage` is a `string`, so the two are equivalent — but confirm the type is `string` and not `any` before assuming.
- **Making `menuSetCurrentSection` optional** can silently skip a state update if a call site passes `undefined` where the callback was previously required. Grep all call sites and verify each one that needs the update still passes a real function.
- **Scoping the observer changes what "no sections observed" means.** If the container ref is not attached on first render (it will not be, on the very first pass), the observer may be constructed with `root: null` — which means *viewport*, the current behaviour. Confirm the observer is (re)created after the ref attaches, or the scoping silently does nothing. This is a real race: the effect at `:43-83` has an empty dependency array after W1.4, so it runs once on mount, when refs on child elements may not yet be populated.
- **Adding a container ref must not add a DOM element.** Wrapping the sections in a `<div>` would change layout and break `data-section` targeting. Use a ref on an existing element.
- **Hoisting `componentsMap` to module scope** requires its values to be module-stable. If it holds `lazy()` calls created per render, hoisting changes when the dynamic import is triggered. Verify the lazy chunk still loads and the Suspense fallback timing is unchanged.
- **W2.13 + W2.2 interact.** Removing `componentsMap` in favour of registry-driven rendering is cleaner than hoisting it, but it is a bigger change. If hoisting is chosen, confirm the map is not rebuilt on every render (acceptance criterion).

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep for the 5 section-name literals → only `sections.ts`.
- [ ] Grep for `sections.indexOf` → zero hits.
- [ ] Wrap-around matrix: from each of the 5 sections, record `next` and `prev`. All 10 transitions correct.
- [ ] DevTools: inject a `<section data-section="Injected">` outside the scroll container → confirm it is never observed and never rendered as "About Me".
- [ ] First-render animation timing compared against the pre-change recording (R15 blocking).
- [ ] Reveal still works: scroll `/scroll` top-to-bottom; each section animates in once, in order, unchanged.
- [ ] `/` ↔ `/scroll` navigation via `MenuBar` — destination correct in both directions.
- [ ] Confirm `menuSetCurrentSection` is not skipped anywhere it was previously required.
- [ ] Screenshot diff at all 13 widths.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; ~20 min for the wrap-around matrix, observer-scoping test and animation timing comparison, ~7 min for the 13-width diff.

---

## Definition of Done

- [ ] One registry; zero duplicate section lists
- [ ] Section→icon lookup exhaustive; `sections.indexOf` gone
- [ ] One arrow-navigation helper; all 10 wrap-around transitions verified
- [ ] `MOTION_VALUE` hoisted; `200`/`100` decision recorded (R15)
- [ ] `atualPage` string prop removed; no loose equality remains
- [ ] `menuSetCurrentSection` optional; no-op call site removed
- [ ] Observer scoped to a container ref
- [ ] `componentsMap` not rebuilt per render
- [ ] Progressive reveal unchanged (OOS-2 respected)
- [ ] `data-section` values unchanged
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W2.2, W2.5, W2.10, W2.11, W2.12, W2.13; Stage 2 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 rows (W2.11 depends on W2.2; W2.13 depends on W2.2), Open Question 2
- `.opencode/plan/codebase-quality-seo/risks.md` — R15, T3, T4, T5, OOS-2, OOS-3
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 1 (`sections.ts`), Tier 2 (`HomePage.tsx` 13 items, `HomePageScroll.tsx` 9 items), § *Contention summary*
- `src/data/sections.ts:1`
- `src/pages/HomePage/HomePage.tsx:23, 28, 55-67, 102, 126, 142-149, 165-172`
- `src/pages/HomePageScroll/HomePageScroll.tsx:48, 79, 97-98, 104-110`
- `src/components/MenuBar/MenuBar.tsx:8-13, 40, 61`