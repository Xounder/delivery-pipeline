# Task — Image Dimensions & Loading Priority, Contact Form Labels, `<article>`/`<time>` Semantics

## Task Information

### ID

TASK-20

### Title

Add explicit image dimensions and loading priorities, add a real `<label>` to the contact form, and add `<time datetime>` + `<article>` semantics to the experience entries

### Owner

senior-frontend

### Status

 Source implementation complete; browser-only acceptance pending (user prohibited browser execution)

### Stage

Stage 4 — SEO & Performance

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W4.7** | Medium | Add explicit `width`/`height` to images, correct `loading`/`decoding`, and mark above-the-fold images `fetchpriority="high"` | `src/components/AboutMe/AboutMe.tsx:13`, `src/components/DefaultCard/DefaultCard.tsx:18`, `src/components/MenuBar/MenuBar.tsx:29,32`, `src/components/DefaultSectionButton/DefaultSectionButton.tsx:8` |
| **W4.8** | Medium | Give the contact form's input a real `<label>` instead of relying on `placeholder` + `pattern` | `src/components/ContactForm/ContactForm.tsx:28-33` |
| **W4.10** | Medium | Experience entries get `<time datetime="...">` around the date string, and the experience card becomes an `<article>` | `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx:19`, `src/components/DefaultCard/DefaultCard.tsx:15-31`, `src/data/data.ts` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **W4.8 adds a `<label>`; the input's `name`, `type`, `pattern`, `required`, submit handler and the simulated submit (OOS-1) must all be unchanged.**
2. **Visual branding** — rendered output must be pixel-identical. **W4.7's `width`/`height` attributes must match the images' rendered dimensions exactly, or the layout shifts. W4.8's `<label>` must be visually hidden or styled to match the current placeholder-only appearance.**
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged. **W4.8's `<label>` text is a new (invisible) string — derive it from the existing `placeholder`. W4.10's `<time>` wraps the existing date string; do not reformat it.**
4. **Routing** — `/` and `/scroll` only, `vercel.json` untouched.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **W4.10 adds `<article>` and `<time>` inside `.group-cards` cards — `data-section` attributes and the `.group-cards` class must survive intact, and TASK-03's observer depends on `.atual-section__content h2` still resolving.**

**Out of scope:** OOS-1 (the contact form's simulated submit) — W4.8 improves the label only. OOS-4 (`<a>`-wrapping-`<button>`) — see the TASK-17 note; the same tension applies here.

---

## Description

Three accessibility + performance items. `feasibility.md` estimates the group at Small–Medium effort, Low risk.

**W4.7 — image dimensions and loading priority.** Images currently have no `width`/`height`, no `loading`, no `decoding`, and no `fetchpriority`. Missing intrinsic dimensions cause **CLS**; missing priorities cause LCP images to be discovered late. `impact-analysis.md` lists the LCP element as `HomePageScroll`'s section content, so above-the-fold images are the priority targets.

**W4.8 — contact form label.** `ContactForm.tsx:28-33` has an input with `placeholder` + `pattern` and no `<label>`. A placeholder is not an accessible name — it disappears on input and is unreliable across AT. `feasibility.md`: *"Add `<label for>`; keep `placeholder` + `pattern` for the existing visual + validation."*

**W4.10 — `<time>` and `<article>`.** Experience entries have a date string with no machine-readable form, and the experience card lacks a semantic wrapper. `impact-analysis.md` flags this as a real SEO opportunity: search engines can use `<time datetime>` for date-range understanding.

---

## Acceptance Criteria

- [ ] Every `<img>` in the four target components has explicit `width` and `height` matching its **rendered** dimensions.
- [ ] Above-the-fold images (per `impact-analysis.md`'s LCP element) carry `fetchpriority="high"` and `loading="eager"`.
- [ ] Below-the-fold images carry `loading="lazy"` and `decoding="async"`.
- [ ] **No layout shift.** Screenshot diff clean at all 13 widths; CLS does not increase.
- [ ] **Alt text is not touched here** — that is W4.6 (TASK-19). This task must not add, remove or alter any `alt` value.
- [ ] The contact form's input has a real `<label for="...">` whose `for` matches the input's `id`.
- [ ] The `<label>` is either visually hidden (preferred) or styled so the rendered result is pixel-identical to today.
- [ ] `placeholder`, `pattern`, `type`, `name`, `required` and the submit handler are unchanged.
- [ ] The input now has an accessible name; DevTools Accessibility pane confirms it.
- [ ] Focus order and the visible focus ring are unchanged.
- [ ] Experience date strings are wrapped in `<time datetime="YYYY-MM-DD">`.
- [ ] The `datetime` value is a valid ISO 8601 date parsed from the existing display string — **the displayed text is unchanged.**
- [ ] Experience cards are wrapped in `<article>`.
- [ ] `.group-cards` and `data-section` attributes survive intact.
- [ ] `.atual-section__content h2` still resolves (TASK-03's observer).
- [ ] Screenshot diff clean at all 13 widths, both routes.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-03
- TASK-11
- TASK-14
- TASK-17
- TASK-19

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-14** — the **hard gate**. R6.
- **TASK-03** — W1.4's observer queries `.atual-section__content h2`. W4.10 inserts `<article>` and `<time>` into the card subtree; **verify the observer's selectors still resolve after the insertion.** Sequential; the observer must be live before you can prove it.
- **TASK-11** — collapsed the button components. `DefaultSectionButton.tsx:8` and `MenuBar.tsx:29,32` may have changed shape. Read them post-TASK-11.
- **TASK-17** — **technically unavoidable**: W4.10's `<article>` wrapper on `DefaultCard.tsx:15-31` is the **same** wrapper W3.6 introduces. Sequential; do not add a second wrapper.
- **TASK-19** — **technically unavoidable**: `DefaultCard.tsx:18` (W4.7's image) is where TASK-19 sets alt text (W4.6), and `data.ts` / `iconsData.ts` are shared. Sequential. **W4.7 and W4.6 are adjacent concerns on the same `<img>` — do not let one task's attributes get reverted by the other.**

**Parallelism:** runs in S4-C alongside TASK-22. `DefaultCard.tsx`/`data.ts` are shared with TASK-19 (done) but **not** with TASK-22 (`main.tsx`, `HomePage.tsx`, `HomePageScroll.tsx`) — disjoint, so S4-C is parallel-safe per `impact-analysis.md` Tier 0.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/components/DefaultCard/DefaultCard.tsx` | 4 | `:18` (W4.7), `:15-31` (W4.10) | TASK-08, TASK-11, **TASK-17** (`:15-31`), **TASK-19** (`:18`) | **Sequential, highest merge risk in the plan.** Ledger: TASK-08 → TASK-11 → **TASK-17** → **TASK-19** → **TASK-20**. **`:15-31` and `:18` are touched by three consecutive tasks — verify no double `<article>` and no lost `alt`/`width`/`height`.** |
| `src/data/data.ts` | 6 | W4.10 (date strings) | TASK-05, TASK-09, TASK-10, TASK-12, **TASK-19** (read-only) | **Sequential.** Read-only in TASK-19. Merge ledger: TASK-09 → TASK-10 → TASK-12 → TASK-19 → **TASK-20**. |
| `src/components/AboutMe/AboutMe.tsx` | 2 | `:13` (W4.7) | **TASK-19** (`:10` W4.6 alt) | **Sequential, adjacent lines.** `:10` and `:13` are 3 lines apart in the same `<img>`. **A merge must keep both the `alt` and the `width`/`height`/`fetchpriority`.** |
| `src/components/ContactForm/ContactForm.tsx` | 1 | `:28-33` (W4.8) | — | No overlap. But `:28-33` sits in the same file as TASK-15's media-query work at `:111`. **Different ranges — low risk.** |
| `src/components/MenuBar/MenuBar.tsx` | 5 | `:29,32` (W4.7) | TASK-05, TASK-07, TASK-08, TASK-10, TASK-12, TASK-13, **TASK-17** (`:45`) | **Sequential.** `:29,32` vs TASK-17's `:45` — isolated. |
| `src/components/DefaultSectionButton/DefaultSectionButton.tsx` | 2 | `:8` (W4.7) | **TASK-11** (W2.4, done) | **Sequential.** Read post-TASK-11. |
| `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx` | 2 | `:19` (W4.10 `<time>`) | TASK-11 (W2.4, done) | **Sequential.** |

---

## Technical Context

### Relevant Components

- `AboutMe` — content-bearing `me.webp`, above the fold
- `DefaultCard` — 14 card images, plus the card wrapper
- `MenuBar` — logo/avatar images
- `DefaultSectionButton` — arrow/section image
- `ContactForm` — the unlabelled input
- `ExperienceCardInfo` — the date string

### Relevant Modules

- `src/components/AboutMe/AboutMe.tsx:13`
- `src/components/DefaultCard/DefaultCard.tsx:15-31`
- `src/components/MenuBar/MenuBar.tsx:29,32`
- `src/components/DefaultSectionButton/DefaultSectionButton.tsx:8`
- `src/components/ContactForm/ContactForm.tsx:28-33`
- `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx:19`
- `src/data/data.ts`

### Relevant APIs

- HTML `width`/`height` attributes on `<img>` (intrinsic aspect-ratio reservation)
- `loading="eager" | "lazy"`, `decoding="async" | "sync"`, `fetchpriority="high" | "low" | "auto"`
- `<label for>` / input `id` — the accessible-name mechanism
- `<time datetime>` — machine-readable date range
- `<article>` — self-contained composition
- Visually-hidden CSS (`.sr-only` or equivalent)

### Relevant Types

- `Experience` entries in `data.ts` (the date fields — TASK-09's discriminated union)
- The image aspect ratios, needed to compute `width`/`height`

---

## Implementation Guidance

### Expected Changes

1. **W4.7** — add `width`/`height` to each `<img>` using its real intrinsic dimensions. Set `fetchpriority="high"` + `loading="eager"` on above-the-fold images; `loading="lazy"` + `decoding="async"` below the fold. **Do not add `alt`** (TASK-19 owns that).
2. **W4.8** — give the input an `id`, add a `<label for>` with visually-hidden text derived from the `placeholder`. Keep `placeholder` and `pattern` exactly as they are.
3. **W4.10** — wrap each experience date string in `<time datetime="YYYY-MM-DD">`, parsing the existing display format. **Reuse TASK-17's `<article>` wrapper — do not add a second one.**

### Constraints

- **W4.7: do not change any `alt` value.** W4.6 belongs to TASK-19. Two tasks, one `<img>` — attribute ownership must be explicit.
- **`width`/`height` must match the rendered dimensions**, not the file's pixel dimensions if CSS resizes it. If CSS constrains the rendered size, either use the rendered size or rely on `aspect-ratio`; **verify no layout shift at all 13 widths.**
- **Never add both `width`/`height` attributes and a CSS `width`/`height` that conflict** without checking the outcome. The attributes set the intrinsic ratio; CSS wins on rendered size.
- **W4.8: do not remove the `placeholder`.** It is the visible affordance today; removing it is a visual change.
- **W4.8: do not change `pattern` or the submit handler.** OOS-1 — the submit stays simulated.
- **W4.8: the `<label>` must be visually hidden**, not `display: none` (which removes it from the accessibility tree) and not visible (which changes layout). Use a clip-based visually-hidden pattern and confirm the existing CSS does not already have one.
- **W4.10: do not reformat the displayed date string.** Only the `datetime` attribute is machine-readable; the visible text is invariant 3.
- **W4.10: do not add a second `<article>`.** TASK-17 already introduced it on the card.
- **W4.10: `<time>` inside an `<a>`-wrapping-`<button>`** is the same structural tension TASK-17 documents. Place it where the structure permits and **record the resolution**; escalate if it cannot be done without restructuring (OOS-4).
- **Do not add a new CSS file for the visually-hidden label** unless one exists; prefer the project's existing utility. If none exists, TASK-15's `index.css` is the established home — **but that file is an overlap file, so confirm no concurrent task owns it.**

### Validation Rules

- **CLS must not increase.** Compare before/after with DevTools' Layout Shift regions; a `width`/`height` mismatch makes this worse, not better.
- **Verify each image's rendered size** at all 13 widths before writing the attributes.
- **DevTools Accessibility pane**: the contact input shows its accessible name; the experience date is exposed as a `<time>` with a machine-readable value.
- **Focus order and focus ring unchanged** after adding the label.
- Screenshot diff at all 13 widths, both routes.
- Grep the four components to confirm every `<img>` has `width`/`height`/`loading` and an `alt` (the latter from TASK-19).
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **`width`/`height` on an image with `max-width: 100%` in CSS.** The attributes set the intrinsic ratio; CSS still constrains the rendered width, and the browser derives height from the ratio. **If the CSS sets only `width` and no `height`, adding a `height` attribute can conflict.** Verify rendered height matches today's.
- **`object-fit` / `object-position` images** (e.g. a circular avatar) may have a rendered box that differs from the file's aspect ratio. Use the **rendered** aspect ratio, not the file's.
- **SVG images and `width`/`height`.** SVGs without intrinsic dimensions ignore the attributes for aspect-ratio reservation. If an icon is an inline SVG rather than an `<img>`, W4.7 does not apply — check the element type per target.
- **Above vs below the fold is route-dependent.** `/` and `/scroll` have different layouts, and the fold is viewport-dependent. **"Above the fold" for `fetchpriority="high"` means above the fold on `/` at the most common desktop viewport** — do not mark every image high priority, or the benefit is lost. `fetchpriority="high"` on many images is worse than none.
- **`loading="lazy"` on an above-the-fold image delays the LCP element.** Getting this wrong is a regression. Verify the LCP element identified by `impact-analysis.md` is eager.
- **A `<label>` wrapping vs referencing.** `for`/`id` is required; a wrapping label without `id` also works but is less robust with conditional rendering. **The input must have a unique `id` — if the form can render more than once, `id` collides.** Use a stable unique id.
- **`ContactForm`'s input may be conditionally rendered** (e.g. only after a click). If so, the `<label>` must render with it, or the association breaks.
- **A visually-hidden label and the focus ring.** Some visually-hidden patterns (e.g. `clip` without `position: absolute`) can affect layout or create a scrollbar. Verify no layout shift.
- **A `display: none` label is invisible to AT** — the exact bug W4.8 fixes. Do not use it.
- **Two labels on one input** (if the plan's other tasks added one) produce an ambiguous accessible name concatenation. Grep first.
- **`<time datetime>` requires a parseable date.** If `data.ts` stores dates as `"Jan 2024 – Present"` or `"Present"`, there is no `datetime` value to emit. **A `<time>` without a valid `datetime` (or with `datetime="Present"`) is invalid** — for an open-ended range, wrap only the start date and record the omission. This is the most likely failure in W4.10; **inspect the actual date strings first.**
- **Present/future dates change over time**, so a hardcoded `datetime` goes stale. Record how it is maintained.
- **A `<time>` element changes the DOM shape inside `.group-cards`.** TASK-03's observer and any `:nth-child`/`:first-child` CSS selector on the card subtree may shift. Grep for structural selectors on cards.
- **`data-section` attribute placement** must not move. If `<article>` or `<time>` is inserted between the element carrying `data-section` and its parent, a `[data-section] > x` selector breaks.
- **The `DefaultCard.tsx` merge ledger is four tasks long.** A conflict resolution that "keeps ours" on `:15-31` can silently drop TASK-17's `<article>` **or** TASK-19's `alt`. After the merge, re-verify all three properties explicitly.
- **OOS-1 and OOS-4 both constrain this task.** Neither may be "fixed" as a side effect.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Record each image's **rendered** dimensions at widths 1, 7, 9, 12, 13; confirm the attributes match.
- [ ] DevTools Layout Shift regions before/after → no new shift entries.
- [ ] Confirm the LCP element is `loading="eager"` + `fetchpriority="high"`; confirm below-fold images are lazy.
- [ ] Grep: every `<img>` in the four components has `width`, `height`, `loading`, and an `alt` (alt from TASK-19).
- [ ] DevTools Accessibility pane: contact input shows an accessible name; keyboard focus order and ring unchanged.
- [ ] Confirm `placeholder`, `pattern`, `type`, `name`, `required` unchanged in `ContactForm.tsx:28-33`.
- [ ] Confirm exactly one `<article>` per experience card (no double wrapper from TASK-17).
- [ ] Confirm displayed date strings are byte-identical; `datetime` values are valid ISO 8601.
- [ ] Confirm `.group-cards` and `data-section` intact; `.atual-section__content h2` still resolves (scroll `/` and confirm section activation still works).
- [ ] Screenshot diff at all 13 widths, both routes.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual only. Budget ~45 min: image dimension audit (~15 min), a11y pane checks (~10 min), 13-width diff (~7 min), date/`datetime` audit (~10 min).

---

## Definition of Done

- [ ] Every target `<img>` has `width`, `height`, `loading`, `decoding`; LCP image has `fetchpriority="high"`
- [ ] No CLS increase; rendered dimensions verified at 5 representative widths
- [ ] No `alt` value changed (W4.6 remains TASK-19's)
- [ ] Contact input has a real, visually-hidden `<label for>`; accessible name confirmed
- [ ] `placeholder`, `pattern` and the simulated submit unchanged (OOS-1)
- [ ] Experience dates wrapped in `<time datetime>` with valid ISO 8601 values
- [ ] Displayed date strings byte-identical
- [ ] `<article>` present exactly once per experience card (reused from TASK-17)
- [ ] `.group-cards`, `data-section`, `.atual-section__content h2` all intact
- [ ] `<a>`-wrapping-`<button>` unchanged (OOS-4); resolution recorded
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes; `npm run lint` green
- [ ] Validation completed

### Source-only implementation evidence (attempt 2)

- Measured source WebP dimensions with installed Pillow (no browser): portrait `me.webp` 400×402; card assets `ifma` 242×208, `maratona-logo` 250×225, `voxy-logo` 250×250, `dio` 250×250, `solyd` 200×200, `api-logo` 250×250, `buletin-blog` 500×314, `buzzfeed` 500×244, `angular-pstore` 500×249, `bokumon` 992×737, `shoes-ecommerce` 500×246; header `logo.webp` 100×102. Inspected SVG viewBoxes: section arrows 24×24, menu icons 24×24 except basketball 512×512, all square. CSS already specifies rendered image boxes: header 70×70 (50×50 compact), menu icons 20×20 / 24×24 / 32×32, section icon 40×40. Portrait and card image boxes use viewport-dependent `clamp`/percent and cannot be encoded exactly for every viewport in constant HTML attributes: source metadata encodes true file aspect ratios; existing CSS retains authority over rendered width and height (including existing image stretching). Browser visual proof required before claiming CLS/no layout shift.
- Portrait is eager/high/async; header logo, menu icons, and section arrows eager/async; cards lazy/async. Existing TASK-19 alt text and TASK-17's sole `<article>` inside the anchor remain untouched. Existing `<a>` containing nested button remains as in TASK-17 (OOS-4); no new wrapper or interaction change.
- Added a clip-hidden `Contact Number*` label referencing a `useId()` input id (unique across multiple form instances). Phone placeholder, pattern, type, name, required and submit handler retained; other fields remain baseline.
- Date inventory: one experience displays `2021 - 2022`, represented by two `<time dateTime="2021">` and `<time dateTime="2022">` endpoints with original separator; recognitions display `2024` (five entries) and `2021` (one entry), represented by year-only `<time dateTime="YYYY">`. HTML time supports valid year-only machine values; adding `-01-01` would invent unsupported month/day precision. Year values are derived from displayed data, not hardcoded. Date button retains button/class/type, and anchor/button structure unchanged.
- `npm run build` passed. `npm run lint` reports the existing six errors / zero warnings baseline: four `jsx-a11y/control-has-associated-label` (including the newly labeled phone input; the rule does not recognize dynamic `useId()` association) and two existing MenuBar clickable-image a11y errors. No test framework configured. No browser, GUI, headless, CDP or test server used, per user restriction. Rendered dimensions at representative widths, CLS, screenshots, DevTools accessible name, focus ring and observer behavior **NOT VERIFIED**; manual browser acceptance remains pending. `.group-cards` and `data-section` were not edited in this task.

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W4.7, W4.8, W4.10; Stage 4 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 4 rows for all three items
- `.opencode/plan/codebase-quality-seo/risks.md` — T5, OOS-1, OOS-4, § *Performance & Core Web Vitals* (CLS, LCP)
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 1/2/3 overlap maps; § *Expected Changes*; LCP element identification
- `src/components/AboutMe/AboutMe.tsx:13`
- `src/components/DefaultCard/DefaultCard.tsx:15-31`
- `src/components/MenuBar/MenuBar.tsx:29,32`
- `src/components/DefaultSectionButton/DefaultSectionButton.tsx:8`
- `src/components/ContactForm/ContactForm.tsx:28-33`
- `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx:19`
- `src/data/data.ts`
