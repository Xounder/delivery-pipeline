# Task — Semantic Landmarks, Inline-Style Removal & Shared Motion Constants

## Task Information

### ID

TASK-17

### Title

Correct `<nav>`/`<aside>` landmark misuse, wrap cards in `<article>`, remove inline styles, extract shared framer-motion variants and duration constants

### Owner

senior-frontend

### Status

Implemented; source-only validation completed, browser verification pending

### Stage

Stage 3 — Responsive & Design-System Consolidation

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W3.6** | Medium | Correct semantic landmark misuse: `<nav>` around a single arrow button, `<aside>` for primary navigation arrows. Every card becomes an `<article>`. No visual effect | `src/components/ArrowSectionButton/ArrowSectionButton.tsx:7`, `src/pages/HomePage/HomePage.tsx:138,161`, `src/components/DefaultCard/DefaultCard.tsx:15-31` |
| **W3.8** | Medium | Remove inline styles that override the stylesheet and duplicate CSS rules | `src/pages/HomePage/HomePage.tsx:136`, `src/components/DefaultCardGroup/DefaultCardGroup.tsx:38` |
| **W3.10** | Medium | Replace the 6 hardcoded framer-motion `transition` objects with shared variants + a duration token; reduce magic animation values (`200vh` fake div, `500ms`, `100ms`, `200ms`) to named constants. **Timings must not change** | `src/pages/HomePage/HomePage.tsx:97,124,135,38,52,77`, `src/pages/HomePageScroll/HomePageScroll.tsx:92,117,143`, `src/components/MenuBar/MenuBar.tsx:45` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **W3.10's invariant is absolute: every animation duration, delay and easing must be numerically identical.** W3.6 changes element names only — behaviour must be untouched.
2. **Visual branding** — rendered output must be pixel-identical. **W3.8 (removing inline styles) is the exception-shaped item: the stylesheet must produce exactly what the inline style was overriding.**
3. **Copy** — unchanged.
4. **Routing** — unchanged.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **W3.6 changes element names — `nav`, `aside`, and the card wrapper. Verify no CSS or JS selector depends on those element names** (`feasibility.md`: *"verify no CSS selector depends on `nav`/`aside` element names"*).

**Protected by OOS-2:** `HomePageScroll`'s progressive per-section reveal (`HomePageScroll.tsx:104-135`) is out of scope. W3.10 touches `:92,117,143` — the transition objects around the reveal — but **the reveal gate and its `y: 300` offset must not change.**

**Protected by OOS-3:** `HomePage`'s scroll-restoration fake `<div>` at `HomePage.tsx:36-53` is out of scope. W3.10 mentions "the `200vh` fake div" as a **magic value to name** — **naming a constant is in scope; changing or removing the hack is not.**

**Protected by OOS-4:** `<a>`-wrapping-`<button>` (`DefaultCard.tsx:15` → `ExperienceCardInfo.tsx:13`, `ProjectCardInfo.tsx:19,28`) is out of scope. W3.6 wraps cards in `<article>` — it **must not** restructure the inner `<a>`/`<button>` nesting.

---

## Description

Three items, all in TSX. This task is **TSX-only** — it touches no CSS file, which is what allows it to run in parallel with TASK-16.

**W3.6 — landmark misuse.** `ArrowSectionButton.tsx:7` wraps a single arrow button in a `<nav>`. `HomePage.tsx:161` uses `<aside>` for the primary navigation arrows — an `aside` is tangential content, not navigation. Every card should become an `<article>`. No visual effect.

**W3.8 — inline styles.** `HomePage.tsx:136` and `DefaultCardGroup.tsx:38` carry inline styles that override the stylesheet and duplicate rules that already exist in CSS. Moving them into the stylesheet is a cleanup — but the resulting rendering must be identical, and **invariant 5 requires that any class name being used is not one of the JS-consumed contracts** (or, if it is, that the CSS keeps working).

**W3.10 — framer-motion constants.** Six hardcoded `transition` objects become shared variants plus a duration token. Magic values — the `200vh` fake div, `500ms`, `100ms`, `200ms` — become named constants. **Timings must not change.** `feasibility.md`: *"Pure extraction of existing values. Timings unchanged."*

**Note on TASK-10's decision:** W2.10 hoisted `MOTION_VALUE` and left the `200` vs `100` discrepancy unresolved pending `feasibility.md` Open Question 2 (`risks.md` **R15**). **W3.10 must not resolve it either** — it extracts values into constants, and the reconciliation remains blocked. If a duration constant here collides with the open question, flag it; do not decide it.

---

## Acceptance Criteria

- [ ] `ArrowSectionButton.tsx:7`'s `<nav>` wrapper is corrected. Landmark count recorded before and after.
- [ ] `HomePage.tsx:161`'s `<aside>` for the primary navigation arrows is corrected to an appropriate landmark.
- [ ] Every card is wrapped in an `<article>` (`DefaultCard.tsx:15-31`).
- [ ] **No CSS selector and no JS query depends on the old `nav`/`aside` element names.** Verified by grep.
- [ ] W3.6 produces **zero** visual change. Screenshot diff clean at all 13 widths.
- [ ] `<a>`-wrapping-`<button>` nesting unchanged (OOS-4).
- [ ] `HomePage.tsx:136` and `DefaultCardGroup.tsx:38` have no inline styles.
- [ ] The stylesheet rules now producing those values are byte-equivalent in effect — screenshot diff clean.
- [ ] The 6 hardcoded framer-motion `transition` objects are replaced by shared variants plus a duration token.
- [ ] The `200vh`, `500ms`, `100ms`, `200ms` magic values are named constants.
- [ ] **Every animation duration, delay and easing is numerically identical to before.** Verified by recording every animation's computed timing before and after.
- [ ] The progressive reveal is unchanged (OOS-2): same gate, same `y: 300`, same `threshold: 0.5`.
- [ ] The scroll-restoration fake `<div>` is unchanged in behaviour (OOS-3) — only its `200vh` is now a named constant.
- [ ] `MOTION_VALUE`'s `200` vs `100` discrepancy remains **unresolved and documented** (R15, Open Question 2).
- [ ] Screenshot diff clean at all 13 widths, both routes.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-10
- TASK-11
- TASK-14

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-14** — the **hard gate**. R6.
- **TASK-10** — owns `HomePage.tsx:23,28` (W2.10's `MOTION_VALUE`) and `HomePageScroll.tsx:92,117` region. This task's W3.10 touches `HomePage.tsx:97,124,135,38,52,77` and `HomePageScroll.tsx:92,117,143`. **`:92` and `:117` overlap exactly.** Sequential, and the `MOTION_VALUE` decision must be preserved.
- **TASK-11** — collapsed the button components. **TASK-11 explicitly deferred the `<nav>` landmark work to W3.6 and instructed that its decision be recorded.** Read `ArrowSectionButton.tsx` after TASK-11 before changing it — the component may be a call-site element rather than a component.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/pages/HomePage/HomePage.tsx` | **13 — the plan's hottest file** | `:38, 52, 77, 97, 124, 135, 136, 161` | TASK-02 (done), TASK-07 (done), **TASK-10** (`:23,28,55-67,102,126,142-149,165-172`), **TASK-22** (W4.12 `:4-19`, W4.13 `:19`) | **Sequential.** **High merge risk at `:97,124,135`** — TASK-10 rewrites the same transition objects for W2.5's helper. **High merge risk at `:23,28`** — TASK-10's unresolved `MOTION_VALUE`. |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | **9** | `:92, 117, 143` | TASK-03, TASK-07, TASK-08, TASK-10 (all done), TASK-21 (W4.17 `:76`), TASK-22 (W4.13 `:12`) | **Sequential.** **`:92,117` overlap with TASK-10.** |
| `src/components/DefaultCard/DefaultCard.tsx` | 4 | `:15-31` | TASK-08 (W1.17 `:22`, done), TASK-11 (indirect), **TASK-20** (W4.7 `:18`, W4.10 `:15-31`) | **Sequential.** **`:15-31` overlaps TASK-20 exactly.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.tsx` | 4 | `:38` | TASK-06, TASK-09, TASK-12 (all done), — | **Sequential.** No overlap remaining. |
| `src/components/ArrowSectionButton/ArrowSectionButton.tsx` | 2 | `:7` | TASK-11 (W2.4 — may have removed this component) | **Sequential.** **Read the file first** — TASK-11 may have folded it into a call site, in which case this work item lands there. |
| `src/components/MenuBar/MenuBar.tsx` | 5 | `:45` | TASK-05, TASK-07, TASK-08, TASK-10, TASK-12, TASK-13 (all done), TASK-20 (`:29,32`) | **Sequential.** `:45` is isolated from all of them. |

**`HomePage.tsx` merge ledger (chronological, verify at each step):**
TASK-02 → TASK-07 → TASK-10 → **TASK-17** → TASK-22

---

## Technical Context

### Relevant Components

- `ArrowSectionButton` — `<nav>` around a single button
- `HomePage` — `<aside>` for navigation arrows, inline style at `:136`, 6 transitions
- `HomePageScroll` — 3 transitions
- `DefaultCard` — card wrapper becomes `<article>`
- `DefaultCardGroup` — inline style at `:38`
- `MenuBar` — transition at `:45`

### Relevant Modules

- `src/components/ArrowSectionButton/ArrowSectionButton.tsx:7`
- `src/pages/HomePage/HomePage.tsx:38, 52, 77, 97, 124, 135, 136, 161`
- `src/pages/HomePageScroll/HomePageScroll.tsx:92, 117, 143`
- `src/components/DefaultCard/DefaultCard.tsx:15-31`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:38`
- `src/components/MenuBar/MenuBar.tsx:45`

### Relevant APIs

- HTML landmarks: `<nav>`, `<aside>`, `<article>`, `<main>`, `<header>`, `<footer>`
- React inline `style` prop
- `framer-motion` — `transition` objects, variants, `animate`, `initial`
- `useReducedMotion` (must be respected if already used)

### Relevant Types

- framer-motion `Transition` / `Variants`
- `MOTION_VALUE` (from TASK-10, still `200` vs `100` unresolved)
- The duration values `500ms`, `100ms`, `200ms`

---

## Implementation Guidance

### Expected Changes

1. **W3.6**
   - `ArrowSectionButton.tsx:7` — replace the `<nav>` around a single button with an appropriate element (or remove the wrapper). Record the decision made in TASK-11.
   - `HomePage.tsx:161` — replace `<aside>` with a proper navigation landmark. Note the plan says "`<aside>` for the primary navigation arrows" — a navigation landmark is `<nav>`, but TASK-11 was told *not* to introduce `<nav>` reuse ambiguously; **choose carefully and record it.**
   - `DefaultCard.tsx:15-31` — wrap each card in `<article>`.
   - Grep for CSS/JS selectors depending on `nav` / `aside` element names.
2. **W3.8** — move `HomePage.tsx:136` and `DefaultCardGroup.tsx:38`'s inline style values into the corresponding stylesheet rules (or delete them if a rule already declares the same values). Verify the computed result is identical.
3. **W3.10** — define shared variants and a duration token. Replace the 6 `transition` objects. Name the `200vh`, `500ms`, `100ms`, `200ms` values as module constants.

### Constraints

- **W3.6 must produce zero visual change.** Changing `<nav>`/`<aside>` to `<article>` changes the default `display` of the element (`nav`/`aside` are block-level, `article` is block-level too — verify no selector relies on the difference) and may affect CSS specificity where element selectors are used.
- **Do not restructure `<a>`-wrapping-`<button>`.** OOS-4. W3.6 adds an `<article>` wrapper; it does not touch the inner nesting.
- **Do not change any animation timing.** R15-adjacent risk: this is invariant 1. Record every duration, delay and easing before and after; they must match numerically.
- **Do not resolve the `MOTION_VALUE` `200`/`100` discrepancy.** Open Question 2 is unresolved. Extract it as a constant if it is not already; do not pick a value.
- **Do not change or remove the `200vh` fake div.** OOS-3. Name its value only.
- **Do not change the progressive reveal.** OOS-2.
- **Do not change `threshold: 0.5`.** W4.17 (TASK-21) documents it.
- **Do not add `transform` or `will-change` optimisations.** Out of scope.
- **Do not remove an inline style whose stylesheet equivalent does not exist yet** without adding the rule. An inline style overriding a stylesheet is a bug; removing it without transferring the value is a visual regression.
- Do not introduce new element wrappers beyond `<article>` — each added element is a potential layout and selector change.

### Validation Rules

- **Screenshots at all 13 widths must be byte-identical.** W3.6 and W3.8 are nominally no-visual-change; any diff is a regression.
- **Record every animation's timing before and after** (duration, delay, easing) and compare numerically. This is the only reliable verification for W3.10.
- Verify each animation **visually** on both routes: `HomePage`'s entrance and arrow transitions, `HomePageScroll`'s section reveal, `MenuBar`'s fade.
- Grep for CSS selectors containing `nav` or `aside` → confirm none breaks.
- Verify the landmark tree with an accessibility tool: exactly the expected landmarks, correct names.
- Confirm all 14 cards render as `<article>` with their content intact.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **`<article>` changes the default display.** `article` is block-level, so most usages are unaffected — but if a card's wrapper was `display: flex` via a selector targeting the old element name, the new element loses it. Grep for `nav`, `aside`, and the old card wrapper element in every stylesheet.
- **Specificity collisions.** A selector like `.card > div` may have matched a `<div>` inside the card; wrapping in `<article>` adds a level and may break a child combinator. Verify the full card subtree renders correctly at all 13 widths.
- **TASK-11 may have removed `ArrowSectionButton`.** W2.4 collapses it into its call sites. If so, `<nav>` now lives in `HomePage.tsx` or another call site. **Read the post-TASK-11 file before editing** — `:7` may not exist.
- **Adding `<article>` inside an existing `<a>`** produces `<a><article>…</article></a>`, which is invalid HTML (block content inside an inline `<a>`). OOS-4 forbids restructuring the nesting. **This is a genuine tension**: the plan asks for `<article>` cards while accepting invalid `<a>`-wrapping-`<button>`. Resolve by placing `<article>` *outside* the `<a>` where the structure permits, and **record the resolution**. If it cannot be resolved without restructuring, escalate rather than guess.
- **Removing an inline style can expose a *different* rule.** An inline style has the highest precedence; the stylesheet rule underneath may declare a different value that was never visible. Transfer the inline value into the stylesheet explicitly — do not simply delete the attribute.
- **W3.10's shared variants may not be truly shared.** Six `transition` objects may differ in duration or easing. Forcing them into one variant changes timings. **Audit all six first**; consolidate only the ones that are genuinely identical, and record the grouping.
- **`transition` vs `animate` timing semantics.** Framer-motion applies `transition` per-property; a shared variant may apply the same timing to properties that previously had different ones. Verify per-property timings, not just the object.
- **The `200vh` constant naming is cosmetic but reviewable.** Ensure the name communicates intent (`SCROLL_RESTORATION_SPACER_HEIGHT`), not just the value.
- **`MenuBar.tsx:45`'s transition is a fade** (from TASK-05's timer context). Its duration must match the 500ms timer window or the fade will be cut short.
- **Reduced-motion users.** If `prefers-reduced-motion` is handled anywhere, shared variants must respect it identically.
- **StrictMode double-render** can make framer-motion initial states flash. Compare in the production build, not only dev.

---

## Testing

### Implementation evidence (TASK-17)

- **W3.6 / TASK-11 decision:** TASK-11 removed `ArrowSectionButton.tsx` and inlined its two `<nav className="arrow-section">` wrappers in `HomePage.tsx`. TASK-17 changed those single-button wrappers to `<div>` without changing their class or child buttons; changed both arrow `<aside>` elements to `<div>` and the existing footer `motion.div` to one labelled `motion.nav`. No extra layout wrapper. Source landmark counts (menu closed): `/` nav 2 → 1, aside 3 → 1; `/scroll` nav 0 → 0, aside 1 → 1. When the existing expandable section menu is open, `/` nav 3 → 2 (its own `<nav>` remains). CSS and JS source searches found no element-name `nav`/`aside` selectors/queries for the removed wrappers. The menu's nav remains nested inside the new footer nav when open.
- **Cards / OOS-4:** `DefaultCard` changes its existing `.default-card__content` direct child of the original anchor from `<div>` to `<article>`; HTML5 `<a>` accepts flow content. All 14 data-driven cards (8 projects, 1 experience, 5 recognitions) use this component. No wrapper level was added; the pre-existing anchor/button nesting was not altered.
- **W3.8:** `HomePage` footer's former inline `width: 100%; height: 100%; display: flex; justify-content: center` is reproduced on `.section-buttons__navigation` with the same declarations. `DefaultCardGroup` keeps `.group-cards` (CSS `justify-content: center`) for the single-card group; multi-card groups additionally get `.group-cards--multiple { justify-content: start; }`. Both JSX `style` props were removed; TASK-16's existing stylesheet rules are retained.

| Animation/timer | Before duration / delay / easing | After duration / delay / easing |
|---|---|---|
| `/` header opacity/x | 0.5s / 0 / default | 0.5s / 0 / default |
| `/` main opacity/x/y (initial position 200; subsequent 100) | 0.5s / 0 / default | 0.5s / 0 / default |
| `/` footer opacity/y | 0.5s / 0 / default | 0.5s / 0 / default |
| `/scroll` header opacity/x | 0.4s / 0 / easeOut | 0.4s / 0 / easeOut |
| `/scroll` section opacity/y, each of five sections | 0.6s / 0 / easeOut | 0.6s / 0 / easeOut |
| `/scroll` footer opacity/y | 0.6s / 0 / easeOut | 0.6s / 0 / easeOut |
| MenuBar visibility timer | 200ms / — / — | 200ms / — / — |
| HomePage page-ready timer | 100ms / — / — | 100ms / — / — |
| HomePage fake-spacer cleanup timer | 500ms / — / — | 500ms / — / — |

All six motion transition objects use shared tokens (home 0.5s, scroll header 0.4s/easeOut, scroll reveal/footer 0.6s/easeOut). The `200vh` fake spacer remains; its value is named only. Progressive-reveal gate, y=300 and IntersectionObserver threshold=0.5 are unchanged. `MOTION_VALUE=100` versus initial `mainMotionValue=200` remains unresolved per R15 / Open Question 2.

**Validation scope:** TASK-14 explicitly skipped under the approved source-only gate. User instruction: *não abra o navegador*. No browser, GUI, headless/CDP automation or server was used. Accessibility-tree counts, computed CSS, animation visuals and screenshots at 13 widths on both routes are **NOT VERIFIED**; the counts and equivalence above are source-based, not runtime measurements. No test script/framework is configured. `npm run build` passed on retry (the first run had a transient Windows `realpath` error reading a framer-motion dependency). `npm run lint` reports 6 a11y errors, 0 warnings, below the accepted baseline of 7 errors, 0 warnings.

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Accessibility-tree landmark check on both routes; before/after landmark counts recorded.
- [ ] Grep all CSS for `nav` / `aside` element selectors → confirm none breaks.
- [ ] All 14 cards render as `<article>`; content intact.
- [ ] Confirm `<a>`-wrapping-`<button>` structure unchanged; record how the `<article>` was placed relative to it.
- [ ] Inline-style check: DevTools confirms no `style` attribute on `HomePage`'s `:136` element and `DefaultCardGroup`'s `:38` element, with identical computed values.
- [ ] Animation timing table: every duration/delay/easing before vs after — must match numerically.
- [ ] Visual check of every animation on both routes.
- [ ] Progressive reveal unchanged: scroll `/scroll`, each section animates in once, in order.
- [ ] Screenshot diff at all 13 widths, both routes.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; the animation-timing table is ~20 min, the accessibility-tree check ~5 min, the 13-width diff ~7 min.

---

## Definition of Done

- [ ] `<nav>`/`<aside>` landmark misuse corrected; landmark counts recorded
- [ ] Every card wrapped in `<article>`
- [ ] No CSS/JS selector broken by the element-name change
- [ ] Inline styles at `HomePage.tsx:136` and `DefaultCardGroup.tsx:38` removed with equivalent stylesheet rules
- [ ] 6 framer-motion transitions replaced by shared variants + duration token
- [ ] Magic animation values named; every timing numerically identical
- [ ] Progressive reveal unchanged (OOS-2); fake div unchanged (OOS-3)
- [ ] `MOTION_VALUE` `200`/`100` still unresolved and documented
- [ ] `<a>`-wrapping-`<button>` unchanged (OOS-4)
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W3.6, W3.8, W3.10; Stage 3 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 row for W2.10, Stage 3 rows for W3.6 / W3.8 / W3.10; Open Question 2
- `.opencode/plan/codebase-quality-seo/risks.md` — R15, T2, T5, OOS-2, OOS-3, OOS-4, § *Accessibility Regression Risk*
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 2 / Tier 3 overlap maps; § *Non-performance risks introduced* (W4.13's animation-silence risk)
- `src/components/ArrowSectionButton/ArrowSectionButton.tsx:7`
- `src/pages/HomePage/HomePage.tsx:38, 52, 77, 97, 124, 135, 136, 161`
- `src/pages/HomePageScroll/HomePageScroll.tsx:92, 117, 143`
- `src/components/DefaultCard/DefaultCard.tsx:15-31`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:38`
- `src/components/MenuBar/MenuBar.tsx:45`
