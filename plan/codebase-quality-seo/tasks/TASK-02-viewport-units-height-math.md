# Task — Root Viewport Units & HomePage Height/Padding Math

## Task Information

### ID

TASK-02

### Title

Fix `100vh`/`100vw` root overflow and the always-overflowing `HomePage` height math

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.7** | Critical | `100vh` → `100dvh` (with `100vh` fallback); `100vw` → `100%` | `src/index.css:26-27`, `src/pages/HomePage/HomePage.css:16` |
| **W1.8** | High | Reconcile `height:100vh` + `padding-top:40px` overflow at `min-width:600px` | `src/pages/HomePage/HomePage.css:16,17,68` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Hard constraint for this task:** the on-screen result must be unchanged. Both work items are overflow *repairs* — they remove scrollbars that should never have existed. Any fix that shifts content by even one pixel is a different change and is out of scope.

---

## Description

`src/index.css:26-27` sets `height:100vh; width:100vw` on the root. Two distinct defects follow:

- **`100vh`** is the *large* viewport on mobile (URL bar collapsed), so the document is taller than the visible area and the bottom of the page is unreachable.
- **`100vw`** includes the desktop scrollbar width, so the root is ~15px wider than the viewport → horizontal scrollbar on desktop.

`src/pages/HomePage/HomePage.css:16` compounds it with `height:100vh`, and `:68` adds `padding-top:40px` at `min-width:600px`. With the default `box-sizing`, that element is permanently `100vh + 40px` tall — an always-present vertical overflow.

**This is the app's documented chronic failure point.** Git history shows two reverts in exactly this area:

| Commit | Message |
|---|---|
| `1fed2df` | modified vh to dvh in home-page |
| `1e7e6e3` | **Revert** "modified vh to dvh in home-page" |
| `403ab6b` | added padding-top min-width 600 |
| `986d27d` | modified padding-top to 0 |

A third and fourth attempt (this task) must not become a fifth revert. That is why `risks.md` **R3** and **R4** are both mitigated here.

### Required internal ordering (mandatory)

`risks.md` R3 says *"Sequence W1.7 before W1.8… Do not touch `HomePage.css` in the same change."*
`risks.md` R4 says *"W1.7 and W1.8 must land together as one reviewed change."*

Both are satisfied by **two ordered commits inside one task**:

1. **Commit A — W1.7 only.** `index.css` root units plus the `HomePage.css:16` viewport-unit swap. No `padding` change. Device-verified before continuing.
2. **Commit B — W1.8.** The `HomePage.css` height/padding reconciliation only, applied *after* Commit A is verified on a real device.

Do not squash the two into a single undifferentiated diff — the revert history is precisely what a merged diff destroys.

---

## Acceptance Criteria

- [ ] `src/index.css:26-27` declares `height:100vh` immediately followed by `height:100dvh` (fallback ordering — non-supporting browsers use `100vh`), and `width:100vw` is replaced by `width:100%`.
- [ ] `src/pages/HomePage/HomePage.css:16` uses the same `100vh` → `100dvh` fallback pattern.
- [ ] **Commit A is device-verified before Commit B is written.** A real mobile device or device emulation with URL-bar collapse, not DevTools-only. Verified widths: 360×640, 390×844, 430×932, 667×375, 844×390.
- [ ] At width 360×640, 390×844 and 430×932: `document.documentElement.scrollHeight <= clientHeight` — no vertical overflow, and the last section of `/scroll` is reachable.
- [ ] At widths 1024×1366, 1180×820, 1280×800, 1440×900, 1920×1080: `document.documentElement.scrollWidth <= clientWidth` — no horizontal scrollbar.
- [ ] At widths 667×375, 844×390, 768×1024, 820×1180, 834×1194, 1180×820: no horizontal **or** vertical scrollbar on `/`.
- [ ] `HomePage.css:16,17,68` no longer produce a `100vh + 40px` box. Either `box-sizing:border-box` is applied to the element, or the `height`/`padding-top` pair is reconciled so total height equals the intended viewport height.
- [ ] The `padding-top:40px` at `min-width:600px` is either preserved as the *visible* offset or replaced by a mathematically equivalent declaration — the rendered offset at 600px+ is unchanged.
- [ ] Screenshot diff at all 13 widths is clean against the pre-Stage-1 build: 360×640 · 390×844 · 430×932 · 667×375 · 844×390 · 768×1024 · 820×1180 · 834×1194 · 1024×1366 · 1180×820 · 1280×800 · 1440×900 · 1920×1080.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.
- [ ] The 13-width diff for Commit A and Commit B are validated **separately** — a combined pass/fail hides which step regressed.

---

## Dependencies

### Required Tasks

- TASK-01

### Dependency Notes

TASK-01 is a hard prerequisite: `npm run build` cannot run until the self-referential `"portfolio": "file:"` dependency is removed, so the fix cannot be compiled or verified.

This task is deliberately a **single task covering both W1.7 and W1.8** rather than two dependent tasks. `risks.md` R4 requires them to land as one reviewed change, and R3 requires strict internal ordering — one task with two ordered commits expresses both, and avoids adding a frontend→frontend dependency edge.

---

## File Overlap Warning

| File | Shared with | Merge strategy |
|---|---|---|
| `src/index.css` | **TASK-16** (W3.5 — colour tokens at `index.css:6`); **TASK-19** (W4.9 — `:lang()` at `index.css:49`); **TASK-21** (W4.15 — dead `Inter` declaration at `index.css:2`) | **Sequential.** Stage 1 vs Stage 3/4, different lines (`:26-27` vs `:2`, `:6`, `:49`). No conflict possible in practice, but do not run them concurrently — Stage 4 begins after Stage 3 exit. |
| `src/pages/HomePage/HomePage.css` | **TASK-15** (W3.2, W3.3, W3.4); **TASK-16** (W3.5, W3.11) | **Sequential, always.** This task owns `HomePage.css` completely for Stage 1. TASK-15 and TASK-16 are Stage 3 and run after Stage 2 exit. |
| `src/pages/HomePage/HomePage.tsx` | Not modified by this task | — |

**No other Stage 1 task touches `index.css` or `HomePage.css`**, so this task is fully parallel-safe with TASK-03, TASK-04, TASK-05 and TASK-06.

---

## Technical Context

### Relevant Components

- `HomePage` (`src/pages/HomePage/`) — the page whose height math is fixed
- Root layout (`src/index.css`)

### Relevant Modules

- `src/index.css:26-27` — `:root` / `html` / `body` sizing
- `src/pages/HomePage/HomePage.css:16,17,68`

### Relevant APIs

- CSS viewport units: `vh`, `dvh`, `vw`
- CSS `box-sizing`
- DevTools: `document.documentElement.scrollHeight / scrollWidth / clientHeight / clientWidth`

### Relevant Types

- None (CSS-only)

---

## Implementation Guidance

### Expected Changes

**Commit A (W1.7)**
1. `src/index.css:26-27`:
   ```css
   /* fallback first, enhancement second — non-supporting browsers keep 100vh */
   height: 100vh;
   height: 100dvh;
   width: 100%;   /* was 100vw: included scrollbar width → horizontal overflow */
   ```
2. `src/pages/HomePage/HomePage.css:16`: same `100vh` → `100dvh` fallback pair. **Do not touch any `padding` value in this commit.**

**Commit B (W1.8)**
3. Reconcile the `height:100vh` + `padding-top:40px` at `min-width:600px` combination at `HomePage.css:16,17,68`. Preferred resolution: make the box model explicit so total height equals the viewport height. Alternatives (negative margin, `calc()`, `box-sizing`) are acceptable **only if** the rendered offset is unchanged.
4. Do **not** tune any timing or magic number to "make it look right". `risks.md` T2 identifies constant-tuning as the trap that produced commits `668502b` → `3abc297` → `3233126`. The fix must be a *mechanism*, not a tuned value.

### Constraints

- **The `100vh` fallback declaration must come first.** Reversing the order disables the fallback entirely — this is the most likely way to reproduce the `1e7e6e3` revert.
- **No value tuning.** Do not change `40px`, `100px`, or any literal to a different number to mask overflow.
- **No token introduction.** Breakpoint and layout tokens are Stage 3 (TASK-15). Stage 1 does CSS *repair*, Stage 3 does CSS *consolidation*. Introducing tokens here would create a Stage 3 dependency.
- **Do not touch the scroll-restoration fake `<div>`** at `HomePage.tsx:36-53` or its 500ms timer. That is OOS-3 and explicitly out of scope.
- **Do not touch `HomePage.tsx`.** This task is CSS-only. The `overflow` restore asymmetry (W1.6) is TASK-07.
- Do not modify any media query. `min-width:600px` stays exactly as-is — `--bp-sm` is Stage 3.
- Verify on a real device, not DevTools only (`risks.md` R3).

### Validation Rules

- The 13-width screenshot diff is **blocking** for this task, not optional. `risks.md` R22: skipping it voids the plan's primary control.
- Widths 1–5 (360×640, 390×844, 430×932, 667×375, 844×390) gate W1.7 — these are the phone widths where `dvh` matters.
- Widths 4, 5 and 10 (667×375, 844×390, 1180×820) are explicitly called out in `risks.md` R2 as the highest-risk regression widths. Check them explicitly even though they are in the matrix.
- Widths 6–13 gate W1.8 — no scrollbar at any of them.
- Horizontal-overflow check (`scrollWidth > clientWidth`) is blocking at widths 1, 2, 3, 4, 5.
- Run `npm run lint` and `npm run build` after each commit, not once at the end.

---

## Edge Cases

- **Browser without `dvh` support.** The `100vh` fallback declaration handles this, but only if it precedes the `dvh` declaration. Any browser ignoring the unknown `dvh` value falls back to `vh` — verify the fallback is *not* dropped by an over-eager CSS minifier or by `vite build`'s CSS pipeline.
- **iOS Safari URL bar collapsing/expanding mid-scroll.** This is the exact failure the `dvh` change addresses. With `100dvh`, the root shrinks and grows as the bar moves — content reflows on every bar transition. Test by scrolling slowly to the bottom and back with the bar partially expanded, not just at rest.
- **iOS Safari `100dvh` and `position:fixed` interaction.** `dvh` tracks the *dynamic* viewport, which excludes the collapsed bar. If `HomePage` relies on `position:fixed` descendants sized against the root, a `dvh` change can move them. Verify the `MenuBar` and the section arrows stay anchored at widths 1–5.
- **Desktop scrollbar removal changes `100vw` → `100%` semantics.** With `width:100%` the root is the containing-block width, which excludes the scrollbar only when the scrollbar is the classic (space-consuming) kind. On overlay-scrollbar platforms (macOS default, some Linux) `100vw` and `100%` already agreed. Confirm no horizontal scrollbar appears *because* `100%` now resolves against a narrower box in a browser with classic scrollbars.
- **Removing the vertical overflow changes the scrollbar's own width**, which feeds back into horizontal layout at `100vw`. Sequence matters: apply W1.7 fully, verify, then apply W1.8.
- **`padding-top:40px` at `min-width:600px` may be load-bearing for visual alignment.** It was added deliberately in `403ab6b`. If the reconciled result moves the first section by any amount at 768×1024 or wider, the fix is wrong — revert to declaring the same padding via a box-model-equivalent mechanism.
- **`min-width:600px` matches exactly at 600px.** There is no viewport in the 13-width matrix at exactly 600px, so a bug at the boundary would be invisible to the matrix. Spot-check 599px and 600px manually.
- **Commit A passes but Commit B reintroduces overflow.** This is the most likely regression. Validate each commit independently, at all 13 widths, and capture separate baseline images.
- **Deep-link directly to `/scroll` at a phone width.** The root height fix interacts with the scroll container. Verify the last section is reachable without a manual scroll-to-bottom.
- **`prefers-reduced-motion` and smooth scroll are irrelevant here** — but `HomePage`'s 500ms `setTimeout` fake-div hack (OOS-3) *does* interact with the new height. On a slow device the removal timer may now fire at a different point in the reflow. Observe the landing position after a reload at width 390×844 and record the result even if it is unchanged.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Commit A: 13-width screenshot diff clean.
- [ ] Commit B: 13-width screenshot diff clean.
- [ ] Widths 1, 2, 3 (360×640, 390×844, 430×932): scroll to the bottom of `/scroll`; confirm the last section is fully reachable and the URL-bar collapse does not hide content.
- [ ] Widths 1–5: `document.documentElement.scrollWidth <= clientWidth`.
- [ ] Widths 6–13: no vertical scrollbar on `/`; no horizontal scrollbar anywhere.
- [ ] Widths 4, 5, 10 (667×375, 844×390, 1180×820): explicit diff — highest regression risk per R2.
- [ ] Width 600px boundary (599px and 600px): no overflow on either side.
- [ ] Real device or device emulation with URL-bar collapse, per R3.
- [ ] `npm run lint` and `npm run build` green after each commit.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification only; cost ≈ 30s per width per commit (13 widths × 2 commits ≈ 13 min). Budget accordingly.

---

## Definition of Done

- [ ] W1.7 implemented with correct `vh` → `dvh` fallback ordering
- [ ] W1.8 implemented with an unchanged rendered offset
- [ ] Two ordered commits, each independently verified
- [ ] No overflow at any of the 13 widths
- [ ] Real-device check completed (R3)
- [ ] Screenshot diff clean at all 13 widths for both commits
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.7, W1.8; the 13-width verification matrix
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 rows for W1.7 / W1.8 ("Sequence W1.7 → W1.8")
- `.opencode/plan/codebase-quality-seo/risks.md` — R1, R2, R3, R4, R22, T1
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 2 and Tier 4 overlap maps
- `src/index.css:26-27`
- `src/pages/HomePage/HomePage.css:16,17,68`
- git history: `1fed2df`, `1e7e6e3`, `403ab6b`, `986d27d`