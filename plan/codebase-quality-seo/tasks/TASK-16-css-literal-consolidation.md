# Task — CSS Literal Consolidation: Colours, Overflow, Hover Mechanism, vh Simplification, Legacy Cleanup

## Task Information

### ID

TASK-16

### Title

Tokenise ~24 colour literals, fix small-screen horizontal overflow, consolidate the 18 `:hover` blocks, simplify unsatisfiable `vh` units, remove legacy CSS properties and magic-number centering

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 3 — Responsive & Design-System Consolidation

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W3.5** | Medium | Tokenise the ~24 hardcoded colour literals, values byte-identical. **`#646cff` is separately gated** | `src/index.css:6`, `src/components/Button/Button.css:15`, `src/components/DefaultCard/DefaultCard.css:13,18,89`, `src/components/AboutMe/AboutMe.css:48`, `src/components/MenuBar/MenuBar.css:79`, `src/components/MenuSectionButton/MenuSectionButton.css:39,47,53,65`, `src/components/DefaultSectionButton/DefaultSectionButton.css:23`, `src/components/InfoCard/InfoCard.css:13`, `src/components/DefaultCardGroup/DefaultCardGroup.css:11,17,29` |
| **W3.7** | Medium | Fix small-screen horizontal overflow: `.menu-section__items` (28rem/max 350px + absolute + translateX(-50%)) and `.button` padding | `src/components/MenuSectionButton/MenuSectionButton.css:13-22,41`, `src/components/Button/Button.css:2` |
| **W3.9** | Medium | Consolidate the 18 duplicated `:hover` blocks that animate `height`/`max-height`/`font-size` into one custom-property-driven mechanism | `src/components/DefaultCard/DefaultCard.css:17-21,74-76,94-97,106-108,116-118,129-131,146-149,164-166,174-177,183-185,197-199,205-207,217-219` |
| **W3.11** | Low | Remove `vh`-unit sprawl (71 occurrences); simplify only where provably equivalent | `src/components/AboutMe/AboutMe.css:19-20`, `src/components/DefaultCard/DefaultCard.css` (31), `src/components/ContactForm/ContactForm.css` (22), `src/pages/HomePage/HomePage.css:16,72,100` |
| **W3.12** | Low | Remove legacy duplicated CSS properties and the `content: ' '` hack | `src/components/DefaultCard/DefaultCard.css:79-84,95-96,170-176`, `src/components/MenuSectionButton/MenuSectionButton.css:66` |
| **W3.13** | Low | Replace the magic-number centering hack with robust positioning | `src/components/MenuBar/MenuBar.css:73-74` (`right: calc(50% - 80px)` + `width: 160px`) |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **W3.7 fixes overflow, which may add or remove a scrollbar — that is the intended fix. W3.9 must produce byte-identical hover dimensions.**
2. **Visual branding** — rendered output must be pixel-identical. **W3.9 is the exception-shaped item: consolidating hover mechanisms is a performance improvement, not a visual one.** Every hover dimension must be preserved exactly.
3. **Copy** — unchanged.
4. **Routing** — unchanged.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **`.group-cards` is the subject of `DefaultCardGroup.css:11,17,29` — the class name stays; only its colour literals move to tokens.**

**Two explicitly gated items — see Edge Cases:**
- **W3.5's `#646cff`** (`Button.css:15`) — the one tokenisation that could change a pixel. Gated on visual verification (`risks.md` R16, `feasibility.md` Open Question 1).
- **W3.11** — only simplify `vh` units where the simplification is *provably* equivalent.

---

## Description

Six CSS cleanup items, all mechanical, all following the TASK-15 token foundation.

**W3.5 — colour literals.** ~24 hardcoded colour values across 11 files become tokens. Values must be byte-identical. The single gated exception is `#646cff` at `Button.css:15` — a Vite-template leftover on a hover border. It may be a deliberate brand accent; **it may not be replaced without answering Open Question 1.**

**W3.7 — small-screen horizontal overflow.** `.menu-section__items` is `width: 28rem; max-width: 350px; position: absolute; left: 50%; transform: translateX(-50%)`. At 360px the viewport is narrower than 350px + centering, so it overflows both edges and causes page-level horizontal scroll. Same class of issue at `.menu-section__items li button` (`21.85rem`) and `.button` (`padding: 9.6px 96px`).

**W3.9 — 18 duplicated `:hover` blocks.** Each animates `height`/`max-height`/`font-size` independently. Consolidating into one custom-property-driven mechanism removes layout thrash on every card hover (an **INP** improvement per `impact-analysis.md`) — **but the rendered hover dimensions must be identical.**

**W3.11 — `vh` sprawl.** 71 `clamp()`/`vh` occurrences across 4 files, several unsatisfiable (e.g. `width: clamp(12em, 90vw, 35rem)` inside a parent capped at `20rem`/`90%` — the `90vw` term can never win).

**W3.12 — legacy properties.** `display: -webkit-box` + `display: box`, `box-orient`, `-webkit-line-clamp` + `line-clamp` pairs, and a `content: ' '` hack. **T8: must be verified in WebKit, not just Chrome.**

**W3.13 — magic-number centering.** `MenuBar.css:73-74`'s `right: calc(50% - 80px)` + `width: 160px` centers by hand.

---

## Acceptance Criteria

- [ ] **Zero hardcoded brand colour literals outside `:root`.** Grep each literal → zero hits.
- [ ] Every token value is byte-identical to the literal it replaced.
- [ ] **`#646cff` (`Button.css:15`) is either kept as a token or replaced — and the decision is recorded with visual verification evidence.** Default: keep as a token until Open Question 1 is answered.
- [ ] At width 1 (360×640), `document.documentElement.scrollWidth <= clientWidth` — no horizontal page scroll.
- [ ] At widths 2 and 3 (390×844, 430×932), no horizontal overflow either.
- [ ] `.menu-section__items` no longer overflows both edges below ~390px.
- [ ] `.button`'s `padding: 9.6px 96px` no longer forces overflow at narrow widths.
- [ ] `.group-cards` class name unchanged (invariant 5).
- [ ] The 18 `:hover` blocks are replaced by one mechanism driven by a single custom property (or `transform`).
- [ ] **Hover-state screenshot diff clean at all 13 widths** — `impact-analysis.md` lists this as a blocking check for W3.9.
- [ ] Hover dimensions are byte-identical: every `height`/`max-height`/`font-size` endpoint preserved.
- [ ] Every `vh`-simplification is provably equivalent; **unsatisfiable terms are the only ones changed**. Each simplification is recorded with its equivalence argument.
- [ ] Legacy `-webkit-box`/`box-orient`/`-webkit-line-clamp` duplicates removed; the modern `line-clamp` remains and behaves identically.
- [ ] `content: ' '` hack removed or replaced with a cleaner mechanism.
- [ ] `MenuBar.css:73-74`'s magic-number centering replaced with robust positioning, rendering identical.
- [ ] Screenshot diff clean at all 13 widths, both routes, **default and hover states**.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-14
- TASK-15

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-14** — the **hard gate**. R6.
- **TASK-15** — technically unavoidable, per file:
  - `feasibility.md`: W3.5 *"Depends on W3.1"* (the tokens must exist first).
  - `feasibility.md`: W3.7 *"Depends on W3.1"*, W3.9 *"Depends on W3.1"*.
  - `impact-analysis.md`: *"W3.5 and W3.2 both touch the same files, so **W3.5 must follow W3.2 per file**."* TASK-15 owns all 12 CSS files for media queries and layout literals; this task owns them for colours, hover, overflow and legacy cleanup. **Same files, sequential required.**

---

## File Overlap Warning

`src/components/DefaultCard/DefaultCard.css` is a **max-1-concurrent-editor** file with **7 work items** across the plan.

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/components/DefaultCard/DefaultCard.css` | **7** | W3.5 (`:13,18,89`), W3.9 (18 blocks), W3.11 (31 occurrences), W3.12 (`:79-84,95-96,170-176`) | TASK-05 (W1.21 `:143`, done), **TASK-15** (W3.2, W3.3 `:188`, W3.4) | **Sequential, strictly.** TASK-15 before this task. |
| `src/components/MenuSectionButton/MenuSectionButton.css` | **9** | W3.5 (`:39,47,53,65`), W3.7 (`:13-22,41`), W3.12 (`:66`) | TASK-04 (done), **TASK-15** (W3.2, W3.3 `:79`, W3.4 `:14`) | **Sequential, strictly.** |
| `src/components/MenuBar/MenuBar.css` | 6 | W3.5 (`:79`), W3.13 (`:73-74`) | **TASK-15** (W3.2, W3.3 `:117`, W3.4) | **Sequential.** |
| `src/pages/HomePage/HomePage.css` | 6 | W3.11 (`:72,100`) | TASK-02 (done), **TASK-15** (W3.2, W3.3, W3.4) | **Sequential.** |
| `src/components/ContactForm/ContactForm.css` | 4 | W3.11 (22 occurrences) | **TASK-15** (W3.2, W3.3 `:111`) | **Sequential.** |
| `src/components/AboutMe/AboutMe.css` | 4 | W3.5 (`:48`), W3.11 (`:19-20`) | **TASK-15** (W3.2, W3.3 `:76`) | **Sequential.** |
| `src/components/Button/Button.css` | 3 | W3.5 (`:15`), W3.7 (`:2`) | TASK-04 (W1.14, done), **TASK-15** (W3.2), TASK-11 (W2.4 CSS merge, done) | **Sequential.** |
| `src/components/InfoCard/InfoCard.css` | 3 | W3.5 (`:13`) | TASK-05 (done), TASK-08 (read-only), **TASK-15** (W3.2) | **Sequential.** |
| `src/components/DefaultSectionButton/DefaultSectionButton.css` | 1 | W3.5 (`:23`) | **TASK-15** (W3.2) | **Sequential.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.css` | 3 | W3.5 (`:11,17,29`) | TASK-09 (W2.7, done), **TASK-15** (W3.2) | **Sequential.** |
| `src/index.css` | 4 | W3.5 (`:6` colour tokens) | TASK-02 (done), **TASK-15** (W3.1 `:root`), **TASK-21** (W4.15 `:2`) | **Sequential.** |

**TASK-15 owns every one of these files first. This task runs strictly after.**

---

## Technical Context

### Relevant Components

- `MenuSectionButton` — the horizontal-overflow source
- `Button` — `padding: 9.6px 96px` and the gated `#646cff`
- `DefaultCard` — 18 hover blocks, 31 `vh` occurrences, legacy `-webkit-box` rules
- `MenuBar` — magic-number centering

### Relevant Modules

- 11 CSS files (see the overlap table)

### Relevant APIs

- CSS custom properties — for the W3.9 hover mechanism
- CSS `clamp()`, `vh`/`vw` units
- `-webkit-line-clamp` / `line-clamp`
- `scrollWidth` / `clientWidth` for overflow verification
- `:hover` pseudo-class

### Relevant Types

- Colour tokens (from TASK-15's `:root`)
- The single hover custom property introduced by W3.9

---

## Implementation Guidance

### Expected Changes

1. **W3.5** — replace each colour literal with its token. Byte-identical values. **Gate `#646cff`:** default to declaring it as a named token (which is a pure rename and changes nothing) rather than replacing it with the branding hover colour. Replacing requires the Open Question 1 answer.
2. **W3.7** — fix `.menu-section__items` so it fits within the viewport at 360px. Use fluid sizing (`min()`, `%`, or a `max-width` that accounts for the viewport) rather than removing the centering transform — **the centering must remain visually identical at widths where the menu fits**. Same for `.menu-section__items li button` (`21.85rem`) and `.button` (`padding: 9.6px 96px`).
3. **W3.9** — introduce one custom property that the `:hover` state flips, with a single rule set reacting to it. Preserve every `height`/`max-height`/`font-size` endpoint exactly.
4. **W3.11** — simplify only unsatisfiable `vh`/`clamp()` terms, with an equivalence argument recorded per change. Leave all satisfiable terms alone.
5. **W3.12** — remove `-webkit-box`/`box-orient` duplicates, keeping modern `line-clamp`. Remove or replace the `content: ' '` hack.
6. **W3.13** — replace `right: calc(50% - 80px)` + `width: 160px` with a robust centering mechanism (e.g. `left: 50%; transform: translateX(-50%)`) that renders identically.

### Constraints

- **Every token value must be byte-identical** to the literal it replaced (R1).
- **Do not replace `#646cff` without the Open Question 1 answer.** Declaring it as a token is a rename and is always safe; changing its value is not.
- **W3.9 must produce byte-identical hover dimensions.** This is a performance refactor, not a visual one. If the mechanism change alters a single rendered pixel, it is wrong.
- **W3.11: simplify only what is provably equivalent.** An unsatisfiable term (one whose parent constraint always wins) is provably dead; a satisfiable one is not.
- **Do not remove `-webkit-line-clamp` if WebKit support requires it.** `line-clamp` without the `-webkit-` prefix is not supported in older Safari. T8: *"Chrome-only verification is insufficient."* Removing the prefixed pair is only safe once the minimum supported WebKit version supports unprefixed `line-clamp` — **verify that, do not assume it.**
- **Do not remove the `left: 50%` + `translateX(-50%)` centering in W3.7** while fixing the overflow. W3.13 is the task that may replace that pattern — here, only the sizing changes.
- **Do not touch `HomePage.css:16`** — TASK-02's `dvh` fix. W3.11 targets `:72,100` there.
- **Do not consolidate the hover blocks into `transform` unless the rendering is provably identical.** Animating `height`/`max-height` is layout-thrashing but produces a specific rendering; `transform` produces a composited one. They can differ by a sub-pixel.
- Do not introduce new breakpoint values.

### Validation Rules

- **Hover-state screenshot diff at all 13 widths is blocking for W3.9** (`impact-analysis.md` § *Verification Plan*).
- **Horizontal-overflow check at widths 1, 2, 3, 4, 5 is blocking for W3.7.**
- **WebKit/Safari verification is required for W3.12.** If Safari is unavailable, record the residual risk explicitly rather than claiming Chromium coverage.
- Grep each colour literal → zero hits outside `:root`.
- Record the `#646cff` decision with evidence.
- Record each W3.11 simplification with its equivalence argument.
- Screenshot diff at all 13 widths, both routes, default **and** hover.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **W3.7 changes the menu's rendered size at narrow widths.** That is the fix (the menu currently overflows). But it means widths 1 and 2 will **diff against the TASK-14 baseline** — and the baseline captured a *defect*. Classify those diffs as intended and record them; do not revert them as regressions.
- **Reducing `.menu-section__items` width may reflow the menu text to more lines**, changing the menu's height and the space it occupies. Verify the open menu at all 13 widths, not just the closed state.
- **`.button`'s `padding: 9.6px 96px` is a large fixed horizontal padding.** Reducing it changes the button's clickable area and its visual width on **every** screen, not just narrow ones. The narrow-screen fix must not alter desktop rendering — verify widths 11–13 explicitly.
- **`width: 28rem` vs `max-width: 350px`** — the effective width is `min(28rem, 350px)`. At 360px viewport the 350px cap plus the `translateX(-50%)` centering still overflows because the *containing block* is narrower. Fixing this means the constraint must reference the viewport, not a fixed cap.
- **W3.9's 18 blocks may not animate identically.** Some animate `height`, some `max-height`, some `font-size`, and the endpoints may differ per card. **One mechanism with one set of endpoints will not fit all 18** unless they are genuinely uniform. Audit all 18 first; if they differ, consolidate per-mechanism (not per-block) and record the grouping.
- **Animating a custom property requires `@property` for smooth interpolation.** A plain custom property is not animatable without `@property` registration (or it snaps). If W3.9 uses a custom property, register it with `@property` including a syntax and initial value — otherwise the hover transition will not animate, which **is** a visual change.
- **`@property` support** is good in modern Chromium/Firefox/Safari but is a relatively recent feature. Verify the hover transition animates (not snaps) in all target browsers.
- **Hover on touch devices.** `:hover` behaves differently on touch. Consolidating the mechanism may change the sticky-hover behaviour on mobile. Verify at widths 1–3 with a touch emulator.
- **`#646cff` may be genuinely intentional.** `feasibility.md` Open Question 1 is unresolved. Declaring it as a token is the safe, no-visual-change action. **Do not "clean up the Vite leftover" on suspicion.**
- **W3.11's unsatisfiable `clamp()` example.** `width: clamp(12em, 90vw, 35rem)` inside a parent capped at `20rem`/`90%` — the `90vw` term can only win if `90vw` is between `12em` and `35rem`, which the parent's own cap prevents. **Verify the parent's constraint is actually tighter at every width** before declaring the term dead. If the parent cap is itself conditional, the term may win at some widths.
- **`vh` vs `dvh` in the same file.** TASK-02 introduced `dvh` at `HomePage.css:16`. W3.11 must not silently convert other `vh` values to `dvh` — that is a behaviour change, not a simplification.
- **W3.12's `line-clamp` requires `display: -webkit-box` in some engines.** The pair `-webkit-box` + `-webkit-line-clamp` is the legacy path; modern `line-clamp` works with `display: block` (or is being standardised that way). **If the stylesheet relies on `-webkit-box` for the clamp to work at all, removing it breaks the clamp.** Verify the clamped text still shows the same number of lines with ellipsis, in Safari.
- **W3.13's `calc(50% - 80px)` centers relative to the *offset parent*,** not the viewport. Replacing it with `translateX(-50%)` changes the reference frame if the element's containing block is not the full width. Verify at every width — this is easy to get right on desktop and wrong on mobile.
- **11 files edited in one task.** Diff **per file** (R1), not as one aggregate. A colour token change in `AboutMe.css` and a hover-mechanism change in `DefaultCard.css` produce unrelated failure modes.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep each of the ~24 colour literals → zero hits outside `:root`.
- [ ] Record the `#646cff` decision with evidence.
- [ ] `scrollWidth <= clientWidth` at widths 1, 2, 3, 4, 5.
- [ ] Open-menu screenshot diff at widths 1, 2, 3 and 11, 12, 13 (menu reflow + desktop button padding).
- [ ] Hover-state screenshot diff at **all 13 widths**, both routes.
- [ ] Confirm the hover transition **animates** (does not snap) in each target browser.
- [ ] Safari/WebKit check for W3.12: clamped text shows the same line count and ellipsis.
- [ ] Record each W3.11 simplification with its equivalence argument.
- [ ] Per-file screenshot diff against the TASK-14 baseline (default state).
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. **Cost note:** hover-state diffs double the matrix cost. 13 widths × 2 routes × 2 states ≈ 26 min minimum, plus the Safari pass. Budget ~1 hour.

---

## Definition of Done

- [ ] Zero hardcoded brand colour literals outside `:root`; values byte-identical
- [ ] `#646cff` decision recorded with evidence
- [ ] No horizontal overflow at widths 1–5
- [ ] Menu centering preserved at widths where the menu fits
- [ ] 18 `:hover` blocks consolidated into one mechanism; dimensions byte-identical
- [ ] Hover-state diff clean at all 13 widths
- [ ] Every `vh` simplification provably equivalent and recorded
- [ ] Legacy CSS properties removed; WebKit verified
- [ ] Magic-number centering replaced; rendering identical
- [ ] `.group-cards` class name unchanged
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W3.5, W3.7, W3.9, W3.11, W3.12, W3.13; Stage 3 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 3 rows for all six items; Open Question 1 (`#646cff`); W3.12's WebKit note
- `.opencode/plan/codebase-quality-seo/risks.md` — R1, R16, T8, § *Performance & Core Web Vitals* (INP improvement from W3.9)
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — § *Verification Plan* (hover-state diff, horizontal-overflow check); Tier 3 overlap map
- `src/components/DefaultCard/DefaultCard.css:13,18,79-84,89,95-96,170-176`
- `src/components/MenuSectionButton/MenuSectionButton.css:13-22,39,41,47,53,65,66`
- `src/components/Button/Button.css:2,15`
- `src/components/MenuBar/MenuBar.css:73-74,79`

---

## Source-only implementation record (TASK-16)

User explicitly skipped TASK-14 and prohibited browser, GUI, headless/CDP, server and screenshot verification. The checks below are source-level reasoning, **not** claims of visual or WebKit verification. TASK-15's `width < 600px` and `(max-width: 1000px) and (aspect-ratio > 1/1)` queries were retained. The TASK-14 screenshot and overflow gates remain unverified by user direction; no screenshot baseline was fabricated.

### Changes by file

| File | Rationale |
|---|---|
| `src/index.css` | Added semantic colour tokens with exactly the same original value spellings; `--button-hover-border: #646cff` intentionally retains the disputed colour without answering Open Question 1. Existing TASK-15 tokens and breakpoints remain unchanged. |
| `src/components/Button/Button.css` | Uses the exact hover accent and other same-value tokens. Only `width < 600px` caps inline padding at `min(96px, 8vw)`; 600px and up retain `9.6px 96px`, transition and click area. |
| `src/components/AboutMe/AboutMe.css` | Same-value surface and neutral-text tokens. The image width clamp is **not** removed: the flex child's intrinsic sizing and percentage image width do not establish a universal cap on the child's own specified width. |
| `src/components/ContactForm/ContactForm.css` | Same-value surface/backdrop tokens; one provable constant clamp simplified (below). Other `vh` clamps can win at some viewport heights and stay. |
| `src/components/DefaultCard/DefaultCard.css` | Same-value colour tokens; one hover rule flips endpoint variables while the existing `height`, `max-height`, `font-size`, `width` and `box-shadow` properties remain the animated/rendered properties. Kept `display: -webkit-box`, `-webkit-box-orient` and both clamp spellings to preserve older WebKit. Removed only unsupported `display: box` / `box-orient` and the duplicate base `-webkit-line-clamp: 4`. The other `vh` values are potentially satisfiable and remain. |
| `src/components/MenuSectionButton/MenuSectionButton.css` | Same-value tokens, preserved `left: 50%` / `translateX(-50%)`; cap list, button, and divider against viewport without changing their existing desktop maximums. Empty-string pseudo-element content still generates the same divider box; whitespace inside its otherwise empty box did not draw a glyph. |
| `src/components/MenuBar/MenuBar.css` | Same-value tokens. Explicit `box-sizing: border-box` fixes the button's 160px border-box width (the common button UA default); old `right: 50% - 80px` puts its left edge at `50% - 80px`. New `left: 50%; translateX(-50%)` puts its left edge at `50% - 160px/2 = 50% - 80px` in the same fixed containing block. No other transform is set on this button. UA defaults in engines not tested remain a browser-dependent caveat. |
| `src/components/DefaultSectionButton/DefaultSectionButton.css` | Reuses exact active-border token. |
| `src/components/InfoCard/InfoCard.css` | Reuses exact text, border and shadow tokens. |
| `src/components/DefaultCardGroup/DefaultCardGroup.css` | Reuses exact scrollbar thumb and transparent tokens; `.group-cards` class and all media queries unchanged. |

### Hover endpoints preserved by the one `.default-card:hover` rule

The hover rule switches `--card-height`, `--card-max-height`, `--card-heading-size`, `--card-clamp`, `--card-component-size`, `--card-content-height`, `--card-image-width`, `--card-image-height` and `--card-shadow-current` to their corresponding `-hover` endpoints. Actual transitions remain on real properties, not unregistered custom properties; `@property` is unnecessary and could change interpolation. The original card transition is `0.2s ease-in-out`, and the content transition remains `0.2s ease-in-out` for width >= 600px. Other child properties did not have transitions and still do not.

| Endpoint | Base -> hover; media overrides in original cascade order |
|---|---|
| card `height` | `clamp(30rem,60vh,45rem)` -> `clamp(35rem,65vh,52rem)`; >=600: `clamp(10rem,22vh,22rem)` -> `clamp(12rem,27vh,25rem)`; compact <=1000 landscape: base `clamp(10rem,25vh,22rem)`, hover remains the applicable preceding endpoint. |
| card `max-height` | `70vh` -> `75vh` in every media state. |
| heading `font-size` | `clamp(1rem,3vh,2.4rem)` -> `clamp(1.1rem,3vh,2.5rem)`; >=600: `clamp(1rem,2vh,2rem)` -> `clamp(1.1rem,2vh,2.1rem)`; compact: `clamp(1rem,3.5vh,1.4rem)` -> `clamp(1.1rem,3.5vh,1.5rem)`. |
| paragraph `line-clamp` / `-webkit-line-clamp` | 4 -> 5; >=600: 3 -> 4; compact leaves preceding values. Paragraph font size unchanged, including its compact override. |
| component `font-size` | `clamp(0.8rem,3vh,1.7rem)` -> `clamp(0.9rem,3vh,1.8rem)`; >=600: `clamp(0.8rem,2vh,1.7rem)` -> `clamp(0.9rem,2vh,1.8rem)`; compact: `clamp(0.8rem,3vh,1.7rem)` -> `clamp(0.9rem,3vh,1.8rem)`. |
| content `height` | Under 600 remains 95% except compact landscape: `clamp(9rem,25vh,20rem)` -> `clamp(10rem,30vh,21rem)`; >=600: `clamp(9rem,20vh,20rem)` -> `clamp(11rem,25vh,23rem)` (compact wins when overlapping). |
| image `width` / `height` | Under 600 stays `clamp(15em,30vh,27rem)` / `clamp(10em,25vh,20rem)` on hover; >=600: `clamp(10em,20vw,15rem)` / `clamp(10em,20vh,15rem)` -> `clamp(11em,20vw,18rem)` / `clamp(11em,20vh,18rem)`. |
| shadow | `2px 2px 8px 0 rgba(255,255,255,0.3)` -> `1px 1px 8px 0 rgba(255,255,255,0.8)` (token values preserve original whitespace/spelling). |

### Equivalence and gated verification

- W3.11: At >=1440px the form button's `clamp(1rem, 1.5vh, 1rem)` is `1rem` for **every** viewport height, because equal minimum and maximum force the clamp to that bound. No other `vh` expressions were simplified: a capped parent does not prove its child `width: clamp(12em, 90vw, 35rem)` is dead; the task example conflates a parent constraint with the child's computed width. HomePage's `dvh` fallback and `vh` rules were untouched.
- W3.7: Menu's old used width was `min(28rem,350px)`; new used width is `min(28rem,350px,100vw - 16px)`, so centering and desktop width are unchanged whenever the old width fits the viewport minus 16px. Button and divider caps similarly retain 290px and 270px whenever they fit. No new breakpoint was introduced. Narrow-screen `.button` padding is intentionally reduced; desktop padding is unchanged.
- W3.12: WebKit verification was not run; preserving the prefixed display/orient/clamp path avoids a known cross-browser regression, so the requested removal of *all* legacy declarations is deliberately partial. `content: ''` still creates the pseudo-element's positioned divider; it removes only a blank text node.
- Source grep confirms the replacement literals occur only in `:root` for the TASK-16 CSS targets. Other CSS files outside the task still have literal `white`/`black` and are intentionally untouched. Browser scrollWidth checks at widths 1–5, screenshots at 13 widths on both routes and states, hover animation checks, and Safari/WebKit checks are **NOT VERIFIED**, as instructed. Therefore pixel identity and runtime overflow cannot be asserted; the browser-dependent DoD items remain gated, rather than treated as passed.
- Source validation: `npm run build` passes (Vite notes three unrelated mixed dynamic/static imports); `npm run lint` returns the established 7 TSX a11y errors and 0 warnings, with no CSS lint findings. `git diff --check` for the edited CSS files reports no whitespace errors. No test script exists; no browser or server was opened. The pre-existing dirty TSX and other work was left untouched.
