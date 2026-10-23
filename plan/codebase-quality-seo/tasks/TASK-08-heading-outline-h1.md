# Task — Document Outline: Exactly One `<h1>` and Re-levelled Headings

## Task Information

### ID

TASK-08

### Title

Introduce the missing `<h1>` and re-level `h2`–`h4` without changing any rendered size

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.17** | Medium | Add exactly one `<h1>`; re-level headings so the outline is `h1 → h2 → h3 → h4`; must not change the rendered text or size of any heading | `src/components/MenuBar/MenuBar.tsx:53`, `src/components/AboutMe/AboutMe.tsx:20`, `src/components/InfoCard/InfoCard.tsx:6`, `src/components/ContactForm/ContactForm.tsx:19`, `src/components/DefaultCard/DefaultCard.tsx:22`, `src/pages/HomePageScroll/HomePageScroll.tsx:146` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. **This task's entire risk is invariant 2: a browser's default heading sizes differ per tag, so changing `<h2>` → `<h1>` changes rendered size unless CSS compensates exactly.**
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged. **Heading *text* must not change** — only tag levels.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **`.atual-section__content h2` is a CSS/JS contract — any change to that element's tag requires a coordinated change in both `HomePageScroll.css` and any observer query.**

---

## Description

The document has **no `<h1>` at all**. Every heading is `h2` or lower, which is an outline defect for screen-reader users (no document title in the heading tree) and a mild semantic SEO signal.

The fix has two halves:

1. **Add exactly one `<h1>`.** `MenuBar.tsx:53` renders the visible site title "Renan Fellipe" — the natural document title and the natural `h1`.
2. **Re-level the remaining headings** so the outline reads `h1 → h2 → h3 → h4` without gaps.

**The whole task is one heading (`MenuBar.tsx:53`) plus tag-level corrections, and the entire risk is invariant 2.** Browsers style `h1` at roughly 2em and `h2` at 1.5em by default. If any stylesheet targets headings by **tag** rather than class, changing the tag silently changes the rendered size.

`risks.md` **R7** names five files with tag-based heading selectors: `AboutMe.css:38`, `DefaultCard.css:67`, `ContactForm.css:14`, `InfoCard.css:17`, `MenuSectionButton.css:27`.

---

## Acceptance Criteria

- [ ] **Every CSS file has been grepped for `h1`–`h6` element selectors** before any tag is changed. The result is recorded in the task's completion notes.
- [ ] Exactly one `<h1>` exists in the document on both `/` and `/scroll`.
- [ ] The `<h1>` is "Renan Fellipe" at `MenuBar.tsx:53`.
- [ ] The document outline contains no skipped levels: `h1 → h2 → h3 → h4` (each `h_n` is preceded by an `h_{n-1}`, or is the first heading in its subtree).
- [ ] Heading **text** is byte-identical to before for every heading on both routes.
- [ ] Heading **rendered size** (computed `font-size`, `line-height`, `margin`) is byte-identical to before at all 13 widths. Any tag-based CSS is compensated so the computed values match exactly.
- [ ] Screenshot diff clean at all 13 widths. **This task has zero tolerance for a visual diff** — a diff means a heading resized.
- [ ] `.atual-section__content h2` still matches its element. If any re-levelling touches that node, the CSS selector and any JS query are updated **in the same change**.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.
- [ ] Verified with an accessibility-tree tool that the document has a single top-level heading and a navigable outline.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-03
- TASK-05
- TASK-06
- TASK-07

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-03** — edits `HomePageScroll.tsx:145-148` (footer Suspense removal). This task edits `:146` (the footer heading). Adjacent lines in a max-1-concurrent-editor file → **sequential**.
- **TASK-05** — edits `MenuBar.tsx:43-46` and `:61`. This task edits `MenuBar.tsx:53`. Same file → **sequential**.
- **TASK-06** — edits `ContactForm.tsx:23-25`. This task edits `ContactForm.tsx:19`. Same file → **sequential**.
- **TASK-07** — edits `HomePageScroll.tsx:146-147` and `MenuBar.tsx:26`. This task edits `HomePageScroll.tsx:146` (the footer heading) and reads `MenuBar.tsx:53`. Overlapping ranges in a max-1-concurrent-editor file → **sequential**. Batch ordering S1-C → S1-D.

**Why these dependencies are unavoidable:** W1.17 is inherently cross-cutting across 6 files. Splitting it would leave the document outline invalid between the two halves. Four of those files (`HomePageScroll.tsx`, `MenuBar.tsx`, `ContactForm.tsx`, and `HomePage.tsx` via the outline owner) are owned by earlier Stage 1 tasks, and one (`ContactForm.tsx`) has two work items from different stages. Sequentially ordering behind the file owners is the only conflict-free option.

---

## File Overlap Warning

| File | Other tasks | Lines | Merge strategy |
|---|---|---|---|
| `src/pages/HomePageScroll/HomePageScroll.tsx` | TASK-03 (`:145-148`), TASK-07 (`:146-147`) | `:146` | **Sequential.** Three tasks, three adjacent line ranges in one max-1-concurrent-editor file. Verify after each that all three changes survived. |
| `src/components/MenuBar/MenuBar.tsx` | TASK-05 (`:43-46`, `:61`), TASK-07 (`:26`) | `:53` | **Sequential.** |
| `src/components/ContactForm/ContactForm.tsx` | TASK-06 (`:23-25`), TASK-11 (`:27,32`), TASK-20 (`:21-26,30,31,32`) | `:19` | **Sequential** — TASK-06 now; TASK-11 and TASK-20 later. |
| `src/components/AboutMe/AboutMe.tsx` | TASK-12 (W2.8, W2.9, W2.14) | `:20` | **Sequential** — Stage 2. TASK-12 rewrites `:21-29` (the description paragraph), adjacent to `:20`. **High merge risk.** |
| `src/components/InfoCard/InfoCard.tsx` | TASK-05 (reads only), TASK-15 (CSS only) | `:6` | **Parallel-safe** — this is the only file in the plan with a single work item. |
| `src/components/DefaultCard/DefaultCard.tsx` | TASK-17 (W3.6 `:15-31`), TASK-20 (W4.7 `:18`, W4.10 `:15-31`) | `:22` | **Sequential.** All three tasks touch `:15-31`. **Highest merge-risk file in the plan after `HomePage.tsx`.** |
| `src/components/InfoCard/InfoCard.css` | TASK-05 (W1.16 `:17-19`), TASK-15 (W3.2) | `:17-19` (read as evidence) | **Read-only for this task**, but see below. |

**Critical read-only dependency:** `InfoCard.tsx:6` renders `<h2>` today. W1.16 (TASK-05) decides whether `InfoCard.css:17-19`'s dead `.info-card h3` rule is retargeted to `h2` **or** deleted. If TASK-05 retargeted it to `h2` and this task changes `<h2>` to `<h3>`, the rule silently dies again. **Check TASK-05's recorded decision before re-levelling `InfoCard.tsx:6`.**

---

## Technical Context

### Relevant Components

- `MenuBar` — renders the site title, the natural `<h1>`
- `AboutMe` — bio section heading
- `InfoCard` — card with an `<h2>` at `:6`
- `ContactForm` — form heading at `:19`
- `DefaultCard` — card heading at `:22`
- `HomePageScroll` — footer heading at `:146`

### Relevant Modules

- `src/components/MenuBar/MenuBar.tsx:53`
- `src/components/AboutMe/AboutMe.tsx:20`
- `src/components/InfoCard/InfoCard.tsx:6`
- `src/components/ContactForm/ContactForm.tsx:19`
- `src/components/DefaultCard/DefaultCard.tsx:22`
- `src/pages/HomePageScroll/HomePageScroll.tsx:146`

### Relevant APIs

- HTML heading semantics and the document outline
- CSS UA default heading sizes (`h1` ≈ 2em, `h2` ≈ 1.5em, `h3` ≈ 1.17em)
- Accessibility-tree heading navigation

### Relevant Types

- None (TSX/HTML structural change)

---

## Implementation Guidance

### Expected Changes

1. **Grep first.** For every CSS file under `src/`, list every selector containing `h1`, `h2`, `h3`, `h4`, `h5`, `h6`. Record the result. This is `risks.md` R7's mitigation and it is mandatory.
2. Add `as h1` (or change `<h2>` → `<h1>`) at `MenuBar.tsx:53` for "Renan Fellipe". If it is currently an `<h2>` and its size comes from a class, verify the computed size is unchanged; if from a tag selector, add the compensating rule.
3. Re-level the other five headings so the outline has no gaps. For each, check whether a tag selector styles it; if so, compensate so the computed size and margins are identical.
4. Verify `.atual-section__content h2` still matches. If any re-levelled node is that element, update the CSS selector and any JS query in the same change (invariant 5).

### Constraints

- **Never change heading text.** Only tags.
- **Never accept a size change.** If compensating CSS is needed, add it — do not accept a different size.
- **Do not add a second `<h1>`.** Both routes must have exactly one.
- **Do not add a visually hidden `<h1>`** as an escape hatch. The site title is a real, visible `<h1>`; introducing a screen-reader-only heading would be a copy and semantics change.
- **Do not rename classes** on any heading.
- **Do not restyle headings "while you're here."** If a heading's font-size looks wrong, that is not this task.
- **Do not use a skip level.** If the natural structure has no `h3`, do not invent one.
- **Do not touch `MenuBar.css` heading rules** unless strictly required to preserve computed size. Colour tokenisation is Stage 3.
- Do not reorder headings in the DOM.

### Validation Rules

- **Screenshot diff at all 13 widths is blocking.** This is the task's only real failure mode. Expect **zero** diff.
- Grep every CSS file for tag-based heading selectors **before** editing any TSX. Record the grep output.
- Verify with an accessibility tool that the outline is `h1 → h2 → h3 → h4` with no skipped levels on both `/` and `/scroll`.
- Verify computed `font-size`, `line-height`, and margins for every heading, before and after, via DevTools.
- Confirm `.atual-section__content h2` still resolves (DevTools: does the selector match?).
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **Tag-based CSS is the whole risk.** `MenuBar.css` may style `.menu-bar h2`, `AboutMe.css:38` may style `h2`, `DefaultCard.css:67` may style `h2`. A tag change silently orphans those rules and falls back to UA defaults. Compute styles before and after for **every** heading — do not eyeball it.
- **UA default margins differ per tag** (`h1` has larger margins than `h2`), so a size-only compensation is insufficient. Check `margin-block-start` / `margin-block-end` too, and any `line-height` set by tag.
- **`MenuBar.tsx:53`'s `<h2>` → `<h1>` is the most visible change.** It is the site title in the navigation bar. If it is styled by a class only, no compensation is needed and no diff appears. If styled by tag, this is where a diff will show.
- **The footer heading at `HomePageScroll.tsx:146` sits in the footer.** Footers often have tag-independent styles, so this is likely the safest re-levelling — but TASK-03 removed the `Suspense` wrapper at `:145-148` in the same region. Confirm the merge.
- **`InfoCard.tsx:6` is entangled with W1.16 (TASK-05).** If TASK-05 retargeted `.info-card h3` to `h2`, changing the tag to `h3` re-breaks it. Check TASK-05's recorded decision first.
- **`DefaultCard.tsx:22` is entangled with W3.6 and W4.10** (both in `:15-31`). Those tasks re-wrap the card in `<article>` and add `<time>`. Keep this task's change to the single tag.
- **`.atual-section__content h2` is a contract.** Invariant 5 lists it explicitly. If the element that carries this class is one of the six headings, re-levelling it from `h2` breaks the CSS selector. This is the single most likely way to violate invariant 5 in this task — check it explicitly.
- **Duplicate `<h1>` across routes.** `/` and `/scroll` render different component sets. Verify the count on **both**, and verify that `/scroll` does not render `MenuBar`'s title twice.
- **`MenuBar` may render on both routes** — if so, `MenuBar.tsx:53` gives one `h1` on each. Good. But confirm `HomePageScroll` does not add its own site title.
- **Outline gaps in nested contexts.** A card's `h3` inside a section's `h2` is correct. A card's `h3` rendered directly under the `h1` with no intervening `h2` is a gap. Map the actual DOM tree rather than the file list — `DefaultCard` is used in several contexts (`HomePage` and `HomePageScroll`, inside `DefaultCardGroup`).
- **A heading inside a `<dialog>` (`ContactForm.tsx:19`).** Its level must fit the outline *when the dialog is open*. Since the dialog is rendered in the DOM (not conditionally mounted), the heading is always in the outline.
- **Screen-reader-only verification.** Use an accessibility tree inspector, not just DevTools markup, to confirm the outline — DevTools shows tags, not the computed accessibility roles.
- **Regressing the audit that found this.** Grep for any tooling or test that asserts heading counts before changing tags.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep all CSS for `h1`–`h6` selectors; record the output **before** editing.
- [ ] Accessibility-tree check on `/`: exactly one `h1`, text "Renan Fellipe"; outline `h1 → h2 → h3 → h4` with no gaps.
- [ ] Accessibility-tree check on `/scroll`: same.
- [ ] DevTools computed styles for every heading, before and after: `font-size`, `line-height`, `margin-block-*`.
- [ ] Screenshot diff at all 13 widths — expect **zero** diff.
- [ ] DevTools: confirm `.atual-section__content h2` still matches an element.
- [ ] Check TASK-05's recorded W1.16 decision before changing `InfoCard.tsx:6`.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; the CSS grep + computed-style comparison is ~15 min, the 13-width zero-diff check ~7 min.

---

## Definition of Done

- [ ] CSS grep for tag-based heading selectors completed and recorded
- [ ] Exactly one `<h1>` on both routes, text "Renan Fellipe"
- [ ] Outline is `h1 → h2 → h3 → h4` with no skipped levels
- [ ] All heading text byte-identical
- [ ] All heading computed sizes and margins byte-identical
- [ ] `.atual-section__content h2` contract preserved
- [ ] Screenshot diff clean at all 13 widths (zero tolerance)
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.17; invariants 2 and 5
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 row for W1.17 ("Must grep all CSS for `h1`–`h4` selectors before changing any tag")
- `.opencode/plan/codebase-quality-seo/risks.md` — R7, § *Accessibility Regression Risk* (W1.17 entry)
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — § *Verification Plan* (heading outline check), Tier 2/Tier 3 overlap maps
- `src/components/MenuBar/MenuBar.tsx:53`
- `src/components/AboutMe/AboutMe.tsx:20`
- `src/components/InfoCard/InfoCard.tsx:6`
- `src/components/ContactForm/ContactForm.tsx:19`
- `src/components/DefaultCard/DefaultCard.tsx:22`
- `src/pages/HomePageScroll/HomePageScroll.tsx:146`