# Task — Button Component Consolidation

## Task Information

### ID

TASK-11

### Title

Collapse four button components into the minimum genuinely-distinct set with an optional `onClick`

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 2 — Organisation, DRY & KISS

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W2.4** | High | Collapse the 4 button components; make `onClick` optional; reduce to the minimum set of genuinely distinct components | `src/components/Button/Button.tsx:3-15`, `src/components/ArrowSectionButton/ArrowSectionButton.tsx:5-11`, `src/components/DefaultSectionButton/DefaultSectionButton.tsx:9-15`, `src/components/MenuSectionButton/MenuSectionButton.tsx:13-51` + call sites `src/components/ContactForm/ContactForm.tsx:27,32`, `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx:13-17`, `src/components/ProjectCardInfo/ProjectCardInfo.tsx:19-23,28-32` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **Every click still does exactly what it did before.**
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. **This is the highest-risk task in Stage 2 for invariant 2: collapsing components means their class names may merge, and each component has its own stylesheet with its own padding, border-radius, hover states and focus rings.**
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **Component-internal class names (`.button`, `.arrow-section-button`, `.default-section-button`, `.menu-section-button`) are styling contracts, not JS contracts — but they must keep selecting what they select today unless the CSS is merged in the same change.**

**Protected by OOS-4:** nested interactive content — `<a>` wrapping `<button>` (`DefaultCard.tsx:15` → `ExperienceCardInfo.tsx:13`, `ProjectCardInfo.tsx:19,28`) — is **accepted out of scope**. W2.4 touches those call sites but **must not restructure the `<a>`/`<button>` nesting**.

**Protected by OOS-1:** the contact form's simulated submit. `ContactForm.tsx:27,32` call sites must keep their current behaviour, including the submit path.

**Required by TASK-05's Stage 1 work:** the `type="button"` attributes added in TASK-05 must **survive** this consolidation.

---

## Description

Four button components exist where far fewer are needed:

| Component | Lines | What it adds |
|---|---|---|
| `Button` | 11 | Base primitive; **forces every caller to pass a no-op `onClick`** |
| `ArrowSectionButton` | 11-line wrapper | Only adds a `<nav>` around a button |
| `DefaultSectionButton` | ~7 | Section navigation variant |
| `MenuSectionButton` | ~39 | The largest; the menu list item |

`ArrowSectionButton` is an 11-line wrapper whose only contribution is a `<nav>` element. `Button` forces callers who need no handler to pass `() => {}`. Reduce to the minimum set of genuinely distinct components, and make `onClick` optional.

`feasibility.md` records why this is safe: *"Safe because every `<Button>` call site passes a no-op handler today — no behaviour is lost by making `onClick` optional."*

**This task touches 8 files and must be executed sequentially — it cannot be parallelised internally.** It is also the single largest Stage 2 task by file count.

---

## Acceptance Criteria

- [ ] `onClick` is optional on the base button. No call site passes a no-op `() => {}` any more.
- [ ] `ArrowSectionButton` is removed or reduced to a non-component (a plain element wrapper). Its `<nav>` semantics are handled by W3.6 (TASK-17), **not** smuggled in here — record whether the `<nav>` survives.
- [ ] The remaining components are the **minimum genuinely-distinct** set. Any two components whose only difference is a class name have been merged, and their CSS merged in the same change.
- [ ] Every existing call site is updated: `ContactForm.tsx:27,32`, `ExperienceCardInfo.tsx:13-17`, `ProjectCardInfo.tsx:19-23,28-32`, plus any call site found by grep.
- [ ] Every click performs exactly the same action as before. Verified by exercising every button on both routes.
- [ ] Every `<button>` still declares `type="button"` (or is a verified form submit) — TASK-05's work survived.
- [ ] `:focus-visible` outlines from TASK-04 still visible on every merged button. **Merging components merges their stylesheets; TASK-04's focus work must not be lost.**
- [ ] Class names are preserved or merged deliberately: `.button`, `.arrow-section-button`, `.default-section-button`, `.menu-section-button` each either still exist and select the same element, or were merged with an equivalent rule.
- [ ] `<a>`-wrapping-`<button>` nesting unchanged (OOS-4).
- [ ] Contact form submit path unchanged (OOS-1).
- [ ] Screenshot diff clean at all 13 widths — **including all hover states and all focus states**.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-09
- Stage 1 exit criteria

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **Stage 1 exit** — `index.md` Stage 2 entry criteria.
- **TASK-09** — `feasibility.md`: *"W2.4 Depends on W2.1."* The button prop types must live in the data-layer types module created by TASK-09, not in a component file. Without TASK-09, W2.4 would reintroduce the inverted dependency it is meant to avoid.

---

## File Overlap Warning

| File | Other tasks | Lines | Merge strategy |
|---|---|---|---|
| `src/components/Button/Button.tsx` | TASK-04 (`.css` only), TASK-16 (`.css` only) | whole file (W2.4) | **Sequential** — TASK-04 done; TASK-16 is Stage 3 CSS only. |
| `src/components/Button/Button.css` | TASK-04 (W1.14 `:18-21`), TASK-16 (W3.5 `:15`, W3.7 `:2`) | — | **Sequential for Stage 3.** This task may merge CSS here; TASK-16 (Stage 3) runs after and expects the post-consolidation file. |
| `src/components/ArrowSectionButton/ArrowSectionButton.tsx` | TASK-17 (W3.6 `:7`), TASK-15 (`.css` W3.2) | whole file | **Sequential** — TASK-17 rewrites the `<nav>` semantics at `:7`. This task runs first; verify TASK-17's expectation still holds. |
| `src/components/ArrowSectionButton/ArrowSectionButton.css` | TASK-15 (W3.2) | — | **Sequential** — Stage 3. |
| `src/components/DefaultSectionButton/DefaultSectionButton.tsx` | TASK-05 (W1.19 `:11`, done), TASK-17 (W3.6), TASK-20 (W4.7 `:12`) | whole file | **Sequential** — TASK-05 done; TASK-17 and TASK-20 later. |
| `src/components/DefaultSectionButton/DefaultSectionButton.css` | TASK-16 (W3.5 `:23`) | — | **Sequential** — Stage 3. |
| `src/components/MenuSectionButton/MenuSectionButton.tsx` | TASK-04 (`.css`), TASK-05 (W1.19 `:33`, done), TASK-15/TASK-16 (`.css`) | whole file | **Sequential** — TASK-05 done; TASK-15/16 are Stage 3 CSS. |
| `src/components/MenuSectionButton/MenuSectionButton.css` | TASK-04 (W1.14, W1.15 — done), TASK-15 (W3.2, W3.3, W3.4), TASK-16 (W3.5, W3.7, W3.12) | — | **Sequential** — Stage 3, 9 plan items on this file. |
| `src/components/ContactForm/ContactForm.tsx` | TASK-06 (W1.13, done), TASK-08 (W1.17, done), TASK-20 (W4.8, W4.9) | `:27,32` | **Sequential** — Stage 4. |
| `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx` | TASK-20 (W4.10 `:11-18`) | `:13-17` | **Sequential** — Stage 4. **High merge risk**: TASK-20 re-wraps the card in `<article>` and adds `<time>` in the same region. |
| `src/components/ProjectCardInfo/ProjectCardInfo.tsx` | TASK-12 (W2.8 `:18`) | `:19-23,28-32` | **Sequential** — TASK-12 runs after this task (Stage 2 batch S2-C). **Adjacent lines** (`:18` vs `:19`). |

**Do not touch** `DefaultCard.tsx:15` — the `<a>` wrapper is OOS-4 and belongs to no task in this plan.

---

## Technical Context

### Relevant Components

- `Button` — base primitive, 11 lines
- `ArrowSectionButton` — 11-line wrapper adding a `<nav>`
- `DefaultSectionButton` — section navigation button
- `MenuSectionButton` — menu list item, the largest of the four

### Relevant Modules

- `src/components/Button/Button.tsx:3-15`
- `src/components/ArrowSectionButton/ArrowSectionButton.tsx:5-11`
- `src/components/DefaultSectionButton/DefaultSectionButton.tsx:9-15`
- `src/components/MenuSectionButton/MenuSectionButton.tsx:13-51`
- `src/components/ContactForm/ContactForm.tsx:27,32`
- `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx:13-17`
- `src/components/ProjectCardInfo/ProjectCardInfo.tsx:19-23,28-32`

### Relevant APIs

- React `ComponentProps` / prop spreading
- HTML `<button>` with `type`, `<nav>` wrapper
- `React.lazy` (only if consolidation interacts with TASK-22's code splitting — it must not)

### Relevant Types

- `Button` props including the currently-mandatory `onClick`
- The prop types relocated by TASK-09 (W2.1)

---

## Implementation Guidance

### Expected Changes

1. Make `onClick` optional on the base `Button`. Update its type signature.
2. Remove every no-op `onClick={() => {}}` at every call site.
3. Fold `ArrowSectionButton` into its call sites. Its only contribution is a `<nav>` element — decide whether to keep it inline at the call site or leave the landmark work to W3.6 (TASK-17), and **record which**.
4. Merge components whose only difference is a class name. When two components merge, **merge their CSS in the same change** so no orphaned rule remains and no specificity conflict is introduced.
5. Update every call site, including any found by grep that the plan did not enumerate.

### Constraints

- **Do not change any rendered output.** This is the invariant most at risk. Padding, border-radius, font, hover state, focus ring, transition and z-index must all be identical.
- **Do not lose TASK-04's `:focus-visible` outlines** when merging stylesheets. Verify by keyboard walkthrough after the merge.
- **Do not lose TASK-05's `type="button"`** attributes.
- **Do not restructure `<a>`-wrapping-`<button>`.** OOS-4. Consolidating components is not permission to fix invalid nesting.
- **Do not change the contact form's submit path.** OOS-1.
- **Do not handle the `<nav>` landmark misuse here.** W3.6 (TASK-17) owns it. Note the decision; do not act on it.
- **Do not put button prop types back in a component file.** TASK-09 (W2.1) just moved them to the data layer.
- **Do not delete a component's CSS file** without merging its rules. An orphaned stylesheet is worse than duplication.
- **Do not add a new visual variant.** If two buttons look different, they are genuinely distinct and must stay distinct.
- Do not touch `DefaultCard.css` or any Stage 3 file's media queries.

### Validation Rules

- **Screenshot diff at all 13 widths, including hover and focus states**, is blocking. Merge regressions in CSS are invisible in a default-state diff.
- Exercise **every** button on both routes and confirm identical behaviour. Enumerate them: menu-bar view change, both section arrows, `MenuSectionButton` items, `ContactForm` buttons, card `<a>`/`<button>` pairs.
- Keyboard walkthrough after the merge — TASK-04's exit criterion must still hold.
- Grep for `onClick={() => {}}` → expect zero hits.
- Grep for every removed component's import → expect zero hits.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **Optional `onClick` changes the rendered DOM.** A `<button>` with no `onClick` is still a `<button>`, but if a component conditionally renders an `<a>` when there is no handler, the DOM changes. Confirm no such branch exists.
- **`Button` may render a different element based on props** (e.g. `<Link>` vs `<button>`). Consolidating components can collapse a distinction that mattered. Read all four components fully before merging.
- **CSS merge creates specificity conflicts.** `.button` (0,1,0) and `.menu-section-button` (0,1,0) have equal specificity; when merged into one rule, source order decides. The surviving declaration must be the one that wins today.
- **Merging `Button.css` into another file changes cascade order** relative to files that override it. Screenshot-diff at all 13 widths, not just one.
- **Merging components can drop a `:hover` transition or an animation.** TASK-04 (W3.9's Stage 1 cousin, and Stage 3's W3.9) depends on hover behaviour being present and identical. Verify hover timings.
- **`ArrowSectionButton`'s `<nav>` is a landmark.** Removing the component removes one `<nav>`, changing the landmark count. W3.6 (TASK-17) audits landmark correctness — it must know the pre-consolidation state. Record the landmark count before and after.
- **`<a>`-wrapping-`<button>` is invalid HTML but is OOS-4.** If consolidation makes it *more* visible or changes which element receives the click, that is a behaviour change. Do not "fix" it.
- **Removing a no-op `onClick` may make a button non-focusable in some browsers?** No — `<button>` is focusable regardless. But confirm no CSS relies on `:hover`/`:active` states that were only reachable through the handler.
- **`MenuSectionButton` is the largest component (`MenuSectionButton.tsx:13-51`).** It may contain logic beyond rendering (state, handlers, refs) that looks like button boilerplate but is not. Read all 39 lines. Do not merge it into `Button` on the basis of its name.
- **TASK-10 (W2.11) makes `menuSetCurrentSection` optional** and may change `MenuSectionButton`'s props. Stage 2 order: TASK-10 is in batch S2-A, this task in S2-B. **Read `MenuBar.tsx` after TASK-10** — its prop contract may have changed.
- **`type="button"` on a `<button>` nested in an `<a>`** is fine. Do not remove it while restructuring.
- **Lazy components and consolidation interact with TASK-22 (W4.12).** Do not introduce `React.lazy` here. TASK-22 owns code splitting.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep `src/` for `onClick={() => {}}` → expect zero hits.
- [ ] Grep `src/` for every removed component's import path → expect zero hits.
- [ ] Enumerate and exercise every button on `/` and `/scroll` — confirm identical behaviour.
- [ ] Grep `src/` for `<button` → every one declares `type` or is a verified form submit (TASK-05's work survived).
- [ ] Keyboard walkthrough on both routes — visible focus ring on every button (TASK-04's work survived).
- [ ] Screenshot diff at all 13 widths, **default + hover + focus**.
- [ ] Landmark count before and after recorded.
- [ ] Confirm `<a>`-wrapping-`<button>` structure unchanged (OOS-4).
- [ ] Contact form: fill, submit, confirm the `<dialog>` opens (OOS-1).
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; the button enumeration + hover/focus diffs are the expensive part (~25 min including the 13-width diffs at three states).

---

## Definition of Done

- [ ] `onClick` optional; zero no-op handlers at call sites
- [ ] `ArrowSectionButton` collapsed or reduced
- [ ] Minimum genuinely-distinct component set achieved, CSS merged alongside
- [ ] All call sites updated
- [ ] Every button behaves identically
- [ ] `type="button"` attributes preserved (TASK-05)
- [ ] `:focus-visible` outlines preserved (TASK-04)
- [ ] `<a>`-wrapping-`<button>` unchanged (OOS-4)
- [ ] Contact form submit path unchanged (OOS-1)
- [ ] Screenshot diff clean at all 13 widths, hover and focus included
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W2.4
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 row for W2.4 ("Depends on W2.1"; "every `<Button>` call site passes a no-op handler today")
- `.opencode/plan/codebase-quality-seo/risks.md` — § *Accessibility Regression Risk* (nested interactive content note), OOS-1, OOS-4
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 3 overlap map
- `src/components/Button/Button.tsx:3-15`
- `src/components/ArrowSectionButton/ArrowSectionButton.tsx:5-11`
- `src/components/DefaultSectionButton/DefaultSectionButton.tsx:9-15`
- `src/components/MenuSectionButton/MenuSectionButton.tsx:13-51`