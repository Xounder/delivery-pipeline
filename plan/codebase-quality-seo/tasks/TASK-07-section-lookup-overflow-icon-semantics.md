# Task — Total Section Lookup, Symmetric Overflow Restore & Icon Semantics

## Task Information

### ID

TASK-07

### Title

Make the section→icon lookup total, restore `overflow` symmetrically, fix the menu-button `alt`, mark decorative icons

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.6** | Critical | Restore `body`/`documentElement` `overflow` to the values they actually had, not `"visible"`/`"auto"` | `src/pages/HomePage/HomePage.tsx:72-83` |
| **W1.11** | High | Make the section→icon lookup total (`sections.indexOf(currentSection)` → `-1` → `undefined` → `icons.map` throws) | `src/pages/HomePage/HomePage.tsx:101`, `src/pages/HomePageScroll/HomePageScroll.tsx:96`, `src/components/MenuBar/MenuBar.tsx:26`, `src/data/sections.ts:1` |
| **W1.18** | Medium | Correct the wrong `alt` on the menu button ("Ícone de um fone de ouvido" for a hamburger asset) | `src/pages/HomePage/HomePage.tsx:156` |
| **W1.20** | Medium | Add `aria-hidden="true"` + `alt=""` to purely decorative arrow/menu icons | `src/pages/HomePage/HomePage.tsx:141,163`, `src/pages/HomePageScroll/HomePageScroll.tsx:146-147` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged — **with one approved exception: W1.18 corrects a wrong `alt`** (that is the fix). W1.20's `alt=""` on decorative icons is also part of the fix; all *meaningful* alt text is preserved.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **`data-section` is read by the observer and must keep its exact values.**

**Protected by OOS-3:** `HomePage`'s scroll-restoration fake `<div>` at `HomePage.tsx:36-53` and its 500ms timer are out of scope. This task touches `:72-83` only, never `:36-53`.

---

## Description

Four defects, all latent or silent, spread across the four hottest files in the codebase.

**W1.6 — asymmetric `overflow` restore.** `HomePage.tsx:72-83` sets `overflow: hidden` on both `document.body` and `document.documentElement`, then restores them to `"visible"` and `"auto"` — values that were **never set beforehand**. After unmount, the document carries a permanent inline `overflow` mutation for the rest of its life. The fix captures the actual prior inline values and restores them verbatim. Strictly safer than today.

**W1.11 — non-total lookup.** Four sites do `iconsData[sections.indexOf(currentSection)]`. `Array.prototype.indexOf` returns `-1` for an unknown value; `iconsData[-1]` is `undefined`; `undefined.map` throws `TypeError`. This is `risks.md` **T7** — a latent crash surface. Today it is unreachable because `currentSection` is always a valid section name; a single bad `dataset.section` value makes it live (see `HomePageScroll.tsx:48`, which silently falls back to `"About Me"` for any unknown value).

Stage 1 delivers a **safe fallback** for the `-1` case. `feasibility.md` is explicit: *"In Stage 1, add a safe fallback for the `-1` case; make it exhaustive in Stage 2."* The exhaustive typed registry is W2.2/W2.3 in TASK-10 (Stage 2).

**W1.18 — wrong `alt`.** `HomePage.tsx:156` labels the hamburger button "Ícone de um fone de ouvido" (a headphone) while the asset is `menu-hamburguer.svg`. A screen-reader user is told the wrong thing.

**W1.20 — decorative icons announced.** Arrow and menu icons are announced as content. Adding `aria-hidden="true"` + `alt=""` removes them from the accessibility tree. **Meaningful alts are preserved** — `me.webp` and the project logos keep (and later gain unique) alt text under W4.6.

---

## Acceptance Criteria

- [ ] `HomePage.tsx:72-83` captures the pre-existing inline `overflow` value of both `document.body` and `document.documentElement` before mutating them, and restores exactly those values on cleanup.
- [ ] A target with no prior inline `overflow` is left with no inline `overflow` after cleanup (not `"visible"`, not `"auto"`).
- [ ] `sections.indexOf(currentSection) === -1` no longer produces `undefined` at any of the four lookup sites (`HomePage.tsx:101`, `HomePageScroll.tsx:96`, `MenuBar.tsx:26`, and wherever `sections.ts` is consumed).
- [ ] A deliberate fallback exists for the unknown-section case, and it is documented rather than silent. The fallback must **not** crash and must **not** invent a section.
- [ ] `src/data/sections.ts` exposes a lookup that cannot return `-1` (a typed registry, map, or `find`-with-fallback) while remaining the single source of the 5 section names.
- [ ] `HomePage.tsx:156`'s `alt` no longer says "fone de ouvido" and correctly describes `menu-hamburguer.svg`.
- [ ] Decorative icons at `HomePage.tsx:141,163` and `HomePageScroll.tsx:146-147` carry `aria-hidden="true"` and `alt=""`.
- [ ] Meaningful alts are unchanged: `me.webp` (`AboutMe.tsx:17`) and all 14 project logos keep their current alt strings — W4.6 in TASK-19 rewrites them for uniqueness, not this task.
- [ ] `data-section` attribute values are unchanged.
- [ ] Screenshot diff clean at all 13 widths. The `alt`/aria changes have zero visual effect; the `overflow` restore must not alter scroll behaviour on `/`.
- [ ] `/` → `/scroll` → `/` → `/` navigation leaves no permanent scroll lock (O3 resolved).
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-03
- TASK-05

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-03** — `HomePageScroll.tsx` is a max-1-concurrent-editor file. TASK-03 fixes the observer effect and deletes the dead block; this task then edits `:96` and `:146-147`. Both are in the same file, and `:146-147` is adjacent to the footer region TASK-03 touches at `:145-148`. **Sequential execution required.**
- **TASK-05** — `MenuBar.tsx` is edited by TASK-05 (W1.9 timer at `:43-46`, W1.19 `type` at `:61`). This task edits `MenuBar.tsx:26`. Same file → **sequential**.

**Why these dependencies are unavoidable:** `HomePageScroll.tsx` (9 plan items) and `MenuBar.tsx` (5 plan items) are max-1-concurrent-editor files. W1.11 is inherently cross-cutting — it must fix all four lookup sites at once, or the crash surface merely moves. There is no way to split W1.11 by file without leaving it half-fixed.

**This is the last Stage 1 task to touch `MenuBar.tsx` and `HomePageScroll.tsx` before the Stage 2 tail (TASK-08).**

---

## File Overlap Warning

Three of the four files here are **max-1-concurrent-editor** files.

| File | Plan items | This task's lines | Other tasks touching it | Merge strategy |
|---|---|---|---|---|
| `src/pages/HomePage/HomePage.tsx` | **13** (W1.6, W1.7, W1.8, W1.11, W1.18, W1.20, W2.2, W2.5, W2.13, W3.8, W3.10, W4.12, W4.13) | `:72-83`, `:101`, `:141`, `:156`, `:163` | TASK-08? no. TASK-10 (W2.2 `:55-67`, W2.5 `:142-149,165-172`, W2.10 `:23,28`, W2.11 `:102`, W2.13 `:55-67,126`), TASK-17 (W3.8 `:136`, W3.10 `:23,28,38,52,77,97,124,135`), TASK-22 (W4.12 `:4-19`, W4.13 `:19`) | **Sequential, always.** TASK-10 (Stage 2) overlaps this task at `:101-102`. TASK-22 (Stage 4) at `:4-19`. |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | **9** | `:96`, `:146-147` | TASK-03 (`:43-83`, `:58-68`, `:145-148`), TASK-08 (`:146`), TASK-10 (`:79`, `:104-110`), TASK-17 (`:92,117,143`), TASK-21 (`:76`), TASK-22 (`:12`) | **Sequential.** TASK-03 runs before; TASK-08 runs after. |
| `src/components/MenuBar/MenuBar.tsx` | 5 | `:26` | TASK-05 (`:43-46`, `:61`), TASK-08 (`:53`), TASK-10 (`:8-13,40,61`), TASK-12 (`:35`), TASK-13 (`:15-19`), TASK-17, TASK-20 (`:29,32`) | **Sequential.** TASK-05 before; TASK-08 and TASK-10 after. |
| `src/data/sections.ts` | 2 (W1.11, W2.2) | `:1` | TASK-10 (W2.2 — rewrites this file into the full registry) | **Sequential.** TASK-10 (Stage 2) builds on this task's lookup helper. |

`src/data/sections.ts` is the lowest-contention file in the plan (2 items) and is **not** in the max-1-concurrent-editor list.

---

## Technical Context

### Relevant Components

- `HomePage` — primary page, mutates and restores document `overflow`
- `HomePageScroll` — scroll page, second lookup site
- `MenuBar` — renders `icons.map(...)` on the looked-up icons (the actual crash site)

### Relevant Modules

- `src/pages/HomePage/HomePage.tsx:72-83, 101, 141, 156, 163`
- `src/pages/HomePageScroll/HomePageScroll.tsx:48, 96, 146-147`
- `src/components/MenuBar/MenuBar.tsx:26`
- `src/data/sections.ts:1`

### Relevant APIs

- `document.body.style.overflow`, `document.documentElement.style.overflow`
- `Array.prototype.indexOf` (returns `-1` — the defect source)
- `sections` / `iconsData` lookup
- `dataset.section` (`HomePageScroll.tsx:48` — `dataset.section || "About Me"` silent mislabelling, risk **T4**)
- `<img aria-hidden alt>` — decorative image semantics

### Relevant Types

- `currentSection` — `string` today; a string that may not be a valid section name
- `Icon` — the shape at `MenuBar.tsx:15-19` and `iconsData.ts`
- `sections` — the 5 canonical section names

---

## Implementation Guidance

### Expected Changes

1. **W1.6** — before mutating, read `document.body.style.overflow` and `document.documentElement.style.overflow`; store both; restore both verbatim in the effect cleanup. If the original inline value was the empty string, restore the empty string (i.e. remove the inline declaration) rather than writing `"visible"`.
2. **W1.11** — in `sections.ts`, add a lookup that returns the icons for a section name or a well-defined safe result when the name is unknown. It must not return `-1`-indexed `undefined`. Use it at all four sites. Stage 2 (TASK-10, W2.2) will extend this into the full typed registry — design the helper so it can grow into it without being rewritten.
3. **W1.18** — correct the `alt` on the hamburger button to describe `menu-hamburguer.svg`.
4. **W1.20** — add `aria-hidden="true"` and `alt=""` to the decorative arrow/menu icons at the three named sites.

### Constraints

- **Do not change any `data-section` value.** Invariant 5. The observer and the lookup both depend on the exact strings.
- **Do not change the section names** themselves. `sections.ts:1` holds the canonical 5 names; this task adds a lookup, not a rename.
- **Do not restructure the `<aside>` at `HomePage.tsx:161`.** Landmark semantics are W3.6 (TASK-17). This task only adds `aria-hidden` to the icon inside it.
- **Do not touch `HomePage.tsx:36-53`** (the fake div / 500ms timer). OOS-3.
- **Do not touch `HomePageScroll.tsx:79`** (`document.querySelectorAll("section")`). That is W2.12 (TASK-10).
- **Do not unify the three section lists yet.** W1.11 adds totality; W2.2 (TASK-10) unifies the three sources of truth. Stage 1 must not import Stage 2's refactor.
- **Do not remove the `dataset.section || "About Me"` fallback** at `HomePageScroll.tsx:48` — that is risk T4, addressed by scoping the observer in W2.12.
- **Do not add `alt` text to decorative icons.** `alt=""` + `aria-hidden="true"` is correct for a purely decorative image; inventing descriptive text would defeat the purpose.
- Do not change any icon asset, size, or positioning.

### Validation Rules

- The `overflow` restore must be verified empirically: navigate `/` → `/scroll` → `/` and inspect `document.body.style.overflow` and `document.documentElement.style.overflow` in DevTools. Both must be empty strings (or whatever they were before).
- The `-1` lookup must be verified by **temporarily** forcing an unknown `currentSection` value, confirming no crash, then reverting the instrumentation. Record the result.
- Screenshot diff clean at all 13 widths. `alt` and `aria-hidden` produce zero visual change; any diff means something else was touched.
- Confirm meaningful alts survived: `me.webp` and all 14 logos.
- Confirm `data-section` values are byte-identical (diff the rendered DOM before/after).
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **`overflow` had a *stylesheet* value, not an inline one.** `document.body.style.overflow` returns `""` when the value comes from CSS. Restoring `""` correctly re-exposes the stylesheet value. Restoring `"auto"` would **override** it — which is exactly today's bug.
- **`overflow` had a meaningful prior inline value.** If some other code set an inline `overflow` before `HomePage` mounts, the correct restore is that value, not `visible`. Capture-and-restore is the only correct behaviour.
- **The effect may run twice in React 18 StrictMode** (development). Capture must happen inside the effect, and cleanup must restore symmetrically, or the second run captures the already-mutated `"hidden"` and "restores" it. Test in the dev build as well as the production build.
- **The effect's cleanup ordering matters.** If the component unmounts while `overflow:hidden` is applied and cleanup restores `""`, any scroll position held under `overflow:hidden` may be lost. Verify the scroll position is preserved across `/` → `/scroll`.
- **`currentSection` is `undefined` or `null`** — not just an unknown string. `indexOf(undefined)` returns `-1`; the helper must handle `undefined`/`null`/empty string without throwing.
- **`sections.indexOf` is case- and whitespace-sensitive.** `"about me"` vs `"About Me"` both return `-1`. The fallback should be case-insensitive *only* if the data actually can produce that; do not add normalisation as a "fix" — the real fix (TASK-10) is a typed registry that makes the value impossible.
- **Choosing the fallback value.** Options: (a) return an empty icon list, (b) return the first section's icons, (c) return no `MenuBar`. Option (b) would *mislabel* the user — the same class of defect as `dataset.section || "About Me"` (T4). Prefer (a) or (c): render nothing rather than something wrong. Record the choice.
- **`MenuBar.tsx:26` is the actual crash site** (`icons.map` on `undefined`). The other three sites are lookups that feed it. Fixing only `MenuBar` would leave the wrong-value propagation in place; fixing only the lookups would leave `MenuBar` able to throw if any future caller passes a bad value. Fix the lookup in `sections.ts` *and* make `MenuBar`'s consumption defensive.
- **`HomePage.tsx:141,163` icons may be inside a `<button>` with its own `aria-label`.** If the button already has an accessible name, `aria-hidden` on the inner `<img>` is safe and correct. If the button relies on the `<img>`'s `alt` for its name, adding `aria-hidden` would remove the button's accessible name entirely — a regression. Check the button's accessible name before and after.
- **`HomePageScroll.tsx:146-147`** is the footer region that TASK-03 also edits (`:145-148`). Confirm the merge did not lose either change.
- **The `alt` on `HomePage.tsx:156` is Portuguese** while `index.html` declares `lang="en"`. W4.9 (TASK-20) fixes the language semantics. The corrected `alt` must be written in Portuguese to match the surrounding copy — do not translate it to English.
- **`W1.11`'s `sections.ts` helper must be forward-compatible with W2.2.** If TASK-10 rewrites `sections.ts` into a registry with name + icons + render target, a throwaway helper here would be deleted and re-created. Write it so TASK-10 extends it rather than replaces it.
- **Route-to-route scroll leakage (O4).** `/scroll` forces `window.scrollTo(0,0)` on mount with no cleanup. This task's `overflow` change interacts with it. Verify `/` → `/scroll` → `/` leaves both routes scrollable.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] DevTools: `document.body.style.overflow` and `document.documentElement.style.overflow` are empty after `/` → `/scroll` → `/`.
- [ ] Temporarily force `currentSection` to `"__unknown__"`, `"about me"` (lowercase), `undefined`, and `""` — confirm no crash in every case; revert instrumentation.
- [ ] Accessibility tree: confirm the hamburger button reports "menu"/"hamburger" (not "headphone"), and that arrow icons are absent from the tree while every button still has an accessible name.
- [ ] Screenshot diff at all 13 widths — expect zero diff.
- [ ] Rendered DOM diff of all `data-section` values — expect byte-identical.
- [ ] Scroll-position check across `/` → `/scroll` → `/`.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; ~5 min for the lookup instrumentation + accessibility tree checks, ~7 min for the 13-width diff.

---

## Definition of Done

- [ ] Prior inline `overflow` values captured and restored verbatim
- [ ] No permanent `overflow` mutation after unmount (O3 resolved)
- [ ] Section→icon lookup cannot produce `undefined` at any of the four sites
- [ ] Unknown-section fallback defined, documented, and non-misleading
- [ ] Hamburger `alt` corrected
- [ ] Decorative icons marked `aria-hidden` + `alt=""`
- [ ] Meaningful alts preserved
- [ ] `data-section` values unchanged
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.6, W1.11, W1.18, W1.20; invariants 4 and 5
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 rows for W1.11 ("add a safe fallback for the `-1` case; make it exhaustive in Stage 2"), W1.18
- `.opencode/plan/codebase-quality-seo/risks.md` — T4, T6, T7, O3, O4, OOS-3, OOS-4
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 1 and Tier 2 overlap maps
- `src/pages/HomePage/HomePage.tsx:72-83, 101, 141, 156, 163`
- `src/pages/HomePageScroll/HomePageScroll.tsx:48, 96, 146-147`
- `src/components/MenuBar/MenuBar.tsx:26`
- `src/data/sections.ts:1`