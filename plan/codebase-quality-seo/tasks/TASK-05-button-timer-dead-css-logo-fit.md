# Task — Lifecycle Hygiene, Button Type Hardening, Dead CSS & Logo Fit Gate

## Task Information

### ID

TASK-05

### Title

Clear the MenuBar fade timer, harden button types, resolve dead InfoCard CSS, gate the card logo object-fit

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.9** | High | Clear the `MenuBar` fade timer on cleanup (`clearTimeout`) | `src/components/MenuBar/MenuBar.tsx:43-46` |
| **W1.19** | Medium | Add `type="button"` to every non-submit `<button>` | `src/components/DefaultSectionButton/DefaultSectionButton.tsx:11`, `src/components/MenuSectionButton/MenuSectionButton.tsx:33`, `src/components/MenuBar/MenuBar.tsx:61` |
| **W1.16** | Medium | Remove dead CSS targeting non-existent `.info-card h3` — retarget or delete | `src/components/InfoCard/InfoCard.css:17-19` vs `src/components/InfoCard/InfoCard.tsx:6` |
| **W1.21** | Low | `object-fit: fill` distorting non-square logos — gated | `src/components/DefaultCard/DefaultCard.css:143` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Explicitly protected by OOS-4:** nested interactive content — `<Link>` wrapping `<button>` at `MenuBar.tsx:61` — is **accepted out of scope**. W1.19 adds `type="button"` to that button and **does not** restructure the nesting.

**Explicitly protected by OOS-1:** the contact form's simulated submit. W1.19's `type="button"` must never be applied to the form's submit button.

---

## Description

Four independent, low-risk correctness items grouped by their "hygiene" nature — each is a small, local, behaviour-neutral repair.

**W1.9 — uncleared `MenuBar` fade timer.** `MenuBar.tsx:43-46` schedules a fade via `setTimeout` but has no cleanup. If the component unmounts (or the effect re-runs) before the timeout fires, the callback runs against a dead component. Stage 1's exit criteria explicitly require "no uncleared timers in `src/`".

**W1.19 — accidental-submit hardening.** HTML `<button>` defaults to `type="submit"`. Three non-submit buttons lack an explicit `type`, so any of them inside a `<form>` would submit it. Adding `type="button"` is behaviour-neutral today (none are inside a form) and prevents a future footgun.

**W1.16 — dead CSS rule.** `InfoCard.css:17-19` targets `.info-card h3`, but `InfoCard.tsx:6` renders an `<h2>`. `text-align:center` never applies. Two valid resolutions: retarget to `h2` (reproduces the apparent intent) or delete (documents that no centering was intended). **Choose by evidence, not preference** — if `h2` already has a `text-align` from elsewhere, retargeting would create a *new* visual change and deletion is correct.

**W1.21 — `object-fit: fill` on non-square logos.** `DefaultCard.css:143` stretches logos to their container, distorting non-square assets. The fix (`contain`) is **explicitly gated**: change it only if the current rendering is pixel-verified equivalent for the shipped logos. Otherwise document and leave. `risks.md` **R13** treats this as gated for exactly that reason.

---

## Acceptance Criteria

- [ ] `MenuBar.tsx:43-46`'s effect returns a cleanup that calls `clearTimeout` on the scheduled timer.
- [ ] `DefaultSectionButton.tsx:11`, `MenuSectionButton.tsx:33` and `MenuBar.tsx:61` each declare `type="button"`.
- [ ] The contact form's submit button is **not** given `type="button"` — its implicit `type="submit"` is preserved. Grep `ContactForm.tsx` to confirm.
- [ ] No `<button>` in `src/` lacks an explicit `type` attribute, except form submits (which must be `type="submit"` or untyped-by-design and verified as such).
- [ ] `InfoCard.css:17-19` is either retargeted to the element that actually renders, or deleted — with the decision and its evidence recorded in the task's completion notes.
- [ ] If retargeted, `text-align:center` produces no screenshot diff (i.e. the rendered result is unchanged).
- [ ] `DefaultCard.css:143` is either changed to `contain` **and** verified pixel-equivalent for all 14 shipped logos, or left as `fill` with the decision documented.
- [ ] If `object-fit` is changed, the 14-card grid screenshot diff at all 13 widths is clean.
- [ ] `src/assets/images/me.png` is untouched (its deletion is W2.14 in TASK-12).
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.
- [ ] Stage 1 exit criterion satisfied: no uncleared timers in `src/`.

---

## Dependencies

### Required Tasks

- TASK-01

### Dependency Notes

TASK-01 is required so `npm run lint` / `npm run build` can run.

No dependency on any other Stage 1 task — this task's files are disjoint from TASK-02 (`index.css`, `HomePage.css`), TASK-03 (`HomePageScroll.tsx`), TASK-04 (`MenuSectionButton.css`, `Button.css`), and TASK-06 (`ContactForm.tsx`, `DefaultCardGroup.tsx`).

**Downstream overlaps (all sequential by stage):**
- **TASK-07** (W1.11) — `MenuBar.tsx:26`
- **TASK-08** (W1.17) — `MenuBar.tsx:53`
- **TASK-10** (W2.11, W2.14) — `MenuBar.tsx:8-13,40,61`
- **TASK-12** (W2.8, W2.14) — `MenuBar.tsx:35,40,61`
- **TASK-13** (W2.17) — `MenuBar.tsx:15-19`
- **TASK-17** (W3.6, W3.10) — `DefaultSectionButton.tsx`, `HomePage.tsx`, `MenuBar.tsx`
- **TASK-20** (W4.7) — `MenuBar.tsx:29,32`, `DefaultSectionButton.tsx:12`
- **TASK-11** (W2.4) — `DefaultSectionButton.tsx`, `MenuSectionButton.tsx` (Stage 2)

---

## File Overlap Warning

`src/components/MenuBar/MenuBar.tsx` (5 work items: W1.9, W2.8, W2.11, W2.14, W2.17) and `src/components/DefaultCard/DefaultCard.css` (7 work items: W1.21, W3.2, W3.3, W3.5, W3.9, W3.11, W3.12) are **max-1-concurrent-editor** files.

| File | Other task | W-IDs / lines | Merge strategy |
|---|---|---|---|
| `MenuBar.tsx` | **TASK-07** | W1.11 at `:26` | **Sequential** — TASK-07 runs after this task. |
| `MenuBar.tsx` | **TASK-08** | W1.17 at `:53` | **Sequential** — TASK-08 runs after this task. |
| `MenuBar.tsx` | **TASK-10** | W2.11 `:8-13,40,61`, W2.14 `:40,61` | **Sequential** — Stage 2. **High merge risk at `:61`**: this task adds `type="button"` there, TASK-10 and TASK-12 both rewrite the surrounding handler. Apply this task's change first and preserve it. |
| `MenuBar.tsx` | **TASK-12** | W2.8 at `:35` | **Sequential** — Stage 2. |
| `MenuBar.tsx` | **TASK-13** | W2.17 at `:15-19` | **Sequential** — Stage 2. May run **parallel with non-overlapping sections** (`:15-19` type export vs `:43-46` timer vs `:35` key vs `:61` button). |
| `MenuSectionButton.tsx` | **TASK-04** | W1.19 vs W1.14/W1.15 (`.css`) | **Parallel-safe** — different file extension. |
| `MenuSectionButton.tsx` | **TASK-11** | W2.4 (whole file) | **Sequential** — Stage 2. |
| `DefaultSectionButton.tsx` | **TASK-11** | W2.4 | **Sequential** — Stage 2. |
| `DefaultSectionButton.tsx` | **TASK-20** | W4.7 at `:12` | **Sequential** — Stage 4. |
| `DefaultCard.css` | **TASK-16** | W3.2, W3.3, W3.5, W3.9, W3.11, W3.12 | **Sequential** — Stage 3. **This task owns `DefaultCard.css` completely for Stage 1.** |
| `InfoCard.css` | **TASK-15** | W3.2 | **Sequential** — Stage 3. |
| `InfoCard.tsx` | **TASK-08** | W1.17 at `:6` | **Parallel-safe** — this task does **not** modify `InfoCard.tsx`, only reads it as evidence. |

Within Stage 1 this task is parallel-safe with TASK-02, TASK-03, TASK-04, TASK-06.

---

## Technical Context

### Relevant Components

- `MenuBar` — fade timer, view-change link
- `DefaultSectionButton` — section navigation button
- `MenuSectionButton` — menu list item button
- `InfoCard` — `<h2>`-rendering card with dead `h3` CSS
- `DefaultCard` — logo `<img>` with `object-fit: fill`

### Relevant Modules

- `src/components/MenuBar/MenuBar.tsx:43-46, 61`
- `src/components/DefaultSectionButton/DefaultSectionButton.tsx:11`
- `src/components/MenuSectionButton/MenuSectionButton.tsx:33`
- `src/components/InfoCard/InfoCard.css:17-19`, `src/components/InfoCard/InfoCard.tsx:6`
- `src/components/DefaultCard/DefaultCard.css:143`

### Relevant APIs

- `setTimeout` / `clearTimeout`
- HTML `<button type="submit">` default behaviour
- CSS `object-fit: fill | contain`

### Relevant Types

- `Icon` (exported from `MenuBar.tsx:15-19`; its relocation is W2.17 in TASK-13)

---

## Implementation Guidance

### Expected Changes

1. **W1.9** — add a cleanup to the effect at `MenuBar.tsx:43-46` that calls `clearTimeout(<id>)`. Store the id in a variable inside the effect scope. Do not change the delay value or the fade logic.
2. **W1.19** — add `type="button"` to the three named buttons. Confirm by grep that `ContactForm.tsx`'s submit button is untouched.
3. **W1.16** — grep all CSS for `text-align` on `InfoCard` descendants and check whether `h2` already has a centering rule. If yes → delete `:17-19` and record why. If no → retarget the selector to `h2` and confirm the diff is empty (i.e. centering already happened via inheritance and the rule was genuinely dead).
4. **W1.21** — inspect all 14 shipped logo assets' intrinsic aspect ratios. If every logo is square, `fill` and `contain` are pixel-identical → change to `contain` and record the evidence. If any is non-square, `fill` is visibly distorting it → decide whether that distortion is the current accepted rendering (do not change) or a defect (change, and record the before/after screenshots as a deliberate, approved exception to invariant 2).

### Constraints

- **Do not restructure the `<Link>`-wrapping-`<button>` nesting** at `MenuBar.tsx:61`. OOS-4. Only add the attribute.
- **Do not touch `ContactForm.tsx`'s submit button.** OOS-1 — the simulated submit is out of scope.
- **Do not change the `MenuBar` fade delay, easing, or duration.**
- **Do not delete `src/assets/images/me.png`** here (W2.14, TASK-12).
- **Do not change any logo asset.** W4.16 in TASK-19 documents duplicate-image usage; replacing assets is a content decision (OOS-5).
- **Do not add a `type` attribute to non-`<button>` elements** (`<a>`, `<Link>`). `type="button"` is only meaningful on `<button>`.
- **Do not change `width`/`height`/`object-position` on the logo `<img>`** — only `object-fit`, and only if gated open.
- **Do not introduce tokens** for any value touched here. Stage 3 owns that.

### Validation Rules

- W1.16 and W1.21 are both **evidence-gated**: the decision must be recorded with the evidence that justified it, not just the outcome. If the evidence is unavailable, the safe default for both is *document and leave*.
- W1.9 requires a functional check: navigate `/` → `/scroll` → `/` rapidly and confirm no error and no post-unmount state update warning in the console.
- W1.19 requires a grep-based audit of **every** `<button>` in `src/`, not just the three named ones. A button missed here will not be caught by any later stage.
- Screenshot diff required if `InfoCard.css` is retargeted or `object-fit` is changed.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **`clearTimeout` with a stale id.** If the effect re-runs before the timeout fires, the cleanup must clear the *newest* id, not a captured outer-scope one. Declare the id inside the effect. Getting this wrong leaves the timer running — the exact defect being fixed.
- **The fade timer may be created conditionally.** If the `setTimeout` at `:43-46` only runs when a state flag is true, the cleanup must handle the "no timer scheduled" case (guard on `undefined`, or initialise the id to `undefined`). A `clearTimeout(undefined)` is harmless but a lint rule may flag it.
- **Rapid navigation can unmount `MenuBar` mid-fade.** `/` → `/scroll` → `/` in under the fade duration. Without the cleanup this triggers a React "state update on unmounted component" warning; with it, nothing happens. Verify in the console.
- **Adding `type="button"` could theoretically change behaviour** if any of the three buttons is inside a `<form>`. Grep to confirm none is. If one *is* inside a form, `type="button"` would **remove** a submit behaviour — in that case stop and escalate, because that would be a functionality change (invariant 1).
- **`MenuBar.tsx:61` merges into Stage 2 rewrites.** TASK-10 and TASK-12 both rework the same line. If `type="button"` is dropped during one of those merges, the W1.19 work is silently lost. Verify after every Stage 2 task that all `<button>` elements still declare a type.
- **`.info-card h3` may have been intentional.** The component once rendered an `<h2>` and may have been migrated. If the intent was always "centre this heading", retargeting restores intent — but that is a *new* visual state, not the current one. Read the git history for `InfoCard.tsx` before deciding. If `h2` is already centred by another rule, the honest action is to delete and note that no centering was intended.
- **Adding `h2` to the selector may create a specificity conflict.** `.info-card h2` (0,1,1) could outrank a `.info-card__title` class rule (0,1,0) on `text-align`. Check the cascade.
- **`object-fit: fill` vs `contain` with `border-radius`.** Switching to `contain` leaves the box the same size but shows the asset's true aspect ratio, centred — which changes the visible logo shape for any non-square asset. That is a visual change and needs explicit approval, not a silent "fix".
- **Non-square logos that look square.** Some logos are visually square but have non-square pixel dimensions with transparent padding. `fill` and `contain` then differ only in the transparent region — invisible, but the diff tool will flag it. Inspect the assets, do not trust the eyeball test alone.
- **`object-fit` interacts with `object-position`.** If `object-position` is set, `contain` positions within the box differently than `fill`. Check it exists before changing.
- **`DefaultCard.css` is a Stage 3 hotspot (7 items).** Any edit here increases the merge surface for TASK-15 and TASK-16. Keep this task's change to the single `object-fit` declaration and nothing else.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep `src/` for `<button` — confirm every one declares an explicit `type` (or is a verified form submit).
- [ ] Grep `src/` for `setTimeout` — confirm each has a matching `clearTimeout` in an effect cleanup.
- [ ] Navigate `/` → `/scroll` → `/` rapidly; confirm no console warning about state updates after unmount.
- [ ] Inspect all 14 logo assets' intrinsic dimensions; record the aspect ratios.
- [ ] Screenshot diff of the 14-card grid at all 13 widths if `object-fit` changed.
- [ ] Screenshot diff of the InfoCard section at all 13 widths if `InfoCard.css` was retargeted.
- [ ] Record the W1.16 and W1.21 decisions with their evidence.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; cost ≈ 30s per width for the card-grid diff (13 widths ≈ 7 min), ~2 min for the greps and navigation check.

---

## Definition of Done

- [ ] `MenuBar` fade timer cleared on cleanup
- [ ] `type="button"` added to all three named non-submit buttons
- [ ] Contact form submit behaviour unchanged (OOS-1 respected)
- [ ] `<Link>`-wrapping-`<button>` nesting unchanged (OOS-4 respected)
- [ ] `.info-card h3` dead rule resolved with a recorded, evidence-based decision
- [ ] `object-fit` resolved with a recorded, evidence-based decision
- [ ] No uncleared timers in `src/` (Stage 1 exit criterion)
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.9, W1.16, W1.19, W1.21
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 rows for W1.9 / W1.16 / W1.19 / W1.21 (W1.21 marked **Gated**)
- `.opencode/plan/codebase-quality-seo/risks.md` — R13, § *Accessibility Regression Risk*, OOS-1, OOS-4, OOS-5
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 3 overlap map
- `src/components/MenuBar/MenuBar.tsx:43-46, 61`
- `src/components/DefaultSectionButton/DefaultSectionButton.tsx:11`
- `src/components/MenuSectionButton/MenuSectionButton.tsx:33`
- `src/components/InfoCard/InfoCard.css:17-19`
- `src/components/DefaultCard/DefaultCard.css:143`