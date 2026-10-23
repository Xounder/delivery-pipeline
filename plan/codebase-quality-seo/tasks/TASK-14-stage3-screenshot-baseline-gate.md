# Task — HARD GATE: Capture the 13-Width Screenshot Baseline

## Task Information

### ID

TASK-14

### Title

Capture and publish the 13-width screenshot baseline from the pre-Stage-3 build

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 3 — Responsive & Design-System Consolidation

### Work Items Covered

**None — this is a gate, not a work item.** It is the mandatory Stage 3 entry criterion. It carries zero of the 68 W-IDs; the W-ID coverage table in `index.md` records it separately as `[GATE]`.

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Hard constraint for this task: ZERO source files are modified.** This task produces an *artifact*, not a code change. If implementing it requires touching `src/`, the artifact does not belong here — escalate.

---

## Description

`risks.md` **R6**: *"Baseline screenshot set is never captured, making R1–R4 undetectable"* — likelihood Medium, **impact High**. Its mitigation is stated as a **hard gate**: *"No Stage 3 task may start without it."*

Stage 3 rewrites ~24 colour literals, 10 layout literals, 28 media queries and 18 `:hover` blocks, and deletes 6 order-dependent `max-width: 1000px and (orientation: landscape)` blocks that currently win over `min-width: 600px` **purely by source order** (`risks.md` **R2**). Without a captured "before" state, a regression is indistinguishable from an intentional change. `risks.md` **R22** adds: *"the 13-width matrix is aspirational and gets skipped under time pressure… Skipping it voids the plan's primary control."*

**This task is the reason Stage 3 is safe. It must not be compressed, sampled, or skipped.**

---

## Acceptance Criteria

- [ ] Screenshots captured from the **pre-Stage-3** build (the Stage 2 exit state), for **both routes** (`/` and `/scroll`), at **all 13 widths**.
- [ ] The 13 widths, exactly:

| # | Viewport | Target | Route |
|---|---|---|---|
| 1 | 360×640 | small Android | both |
| 2 | 390×844 | iPhone 14 | both |
| 3 | 430×932 | iPhone Pro Max | both |
| 4 | 667×375 | phone landscape (short) | **priority** |
| 5 | 844×390 | phone landscape (tall) | **priority** |
| 6 | 768×1024 | tablet portrait — currently undesigned | both |
| 7 | 820×1180 | iPad Air portrait — currently undesigned | both |
| 8 | 834×1194 | iPad Pro portrait — currently undesigned | both |
| 9 | 1024×1366 | iPad Pro portrait (large) | both |
| 10 | 1180×820 | iPad Air landscape | **priority** |
| 11 | 1280×800 | small desktop | both |
| 12 | 1440×900 | desktop | both |
| 13 | 1920×1080 | large desktop | both |

- [ ] **26 base images captured** (13 widths × 2 routes). Hover-state images captured at all 13 widths for `/` (W3.9 modifies all 18 `:hover` blocks).
- [ ] Each capture is deterministic: fonts loaded, lazy content settled, scroll position set, animations in their **final** state.
- [ ] Scroll-reveal content on `/scroll` is captured in its **revealed** state (scrolled through once before capture) so the baseline reflects the settled page, not the pre-reveal state.
- [ ] The artifact set is stored **outside** `src/` and is not committed to the app bundle.
- [ ] The capture procedure is documented so TASK-15, TASK-16 and TASK-17 can reproduce it identically (same tool, same DPR, same scroll offset, same settle delay).
- [ ] **A "no-change" control diff is run and passes** — capture twice with no code change in between and confirm the diff is zero or explain every non-zero pixel. This validates the capture pipeline itself.
- [ ] `npm run build` and `npm run lint` are green at capture time (Stage 2 exit state).

---

## Dependencies

### Required Tasks

- Stage 1 exit criteria
- Stage 2 exit criteria (TASK-09, TASK-10, TASK-11, TASK-12, TASK-13 complete)

### Dependency Notes

`index.md` Stage 3 entry criteria:
- All Stage 2 exit criteria met.
- **A baseline screenshot set at all 13 widths has been captured from the pre-Stage-3 build and is available for diffing.**

This is the **only** dependency in the plan that exists purely to make another task's verification possible. It is justified by `risks.md` R6 (hard gate) and R22.

**Every Stage 3 task (TASK-15, TASK-16, TASK-17) depends on this task.**

---

## File Overlap Warning

**No source file is modified by this task**, so there is no merge risk. However:

| Consideration | Detail |
|---|---|
| Capture artifacts | Store under a path that is **not** part of the Vite build (e.g. `.opencode/plan/codebase-quality-seo/baseline/`). Do **not** put them in `public/` — that would ship them to production and change the deployed asset set (`impact-analysis.md` § *Expected Changes* lists exactly which files change). |
| `public/` | **Untouched.** W4.4 adds `robots.txt` and `sitemap.xml` in TASK-18. |
| `dist/` | If capturing from a production build, do not commit `dist/`. Confirm it is gitignored. |
| Concurrent task | TASK-13 (Stage 2 batch S2-B) may still be running when this task starts if the orchestrator overlaps stage boundaries. `index.md` Stage boundaries are hard — **do not start this task until Stage 2 exit criteria are met**, or the baseline captures the wrong state. |

---

## Technical Context

### Relevant Components

- Both routes: `/` and `/scroll`

### Relevant Modules

- No source modules. The build output (`dist/`) is the capture target.

### Relevant APIs

- Headless browser / DevTools device emulation / screenshot tooling (whatever is available — **no test framework is configured, OOS-7**)
- Viewport emulation at exact pixel dimensions
- Image diffing (pixel comparison)

### Relevant Types

- None

---

## Implementation Guidance

### Expected Changes

1. Verify Stage 2 exit criteria: `npm run lint` ≤ baseline, `npm run build` passes, W2.x exit criteria met.
2. `npm run build` to produce `dist/`, then `npm run preview` (or `npm run dev` — but a **production build** is preferred because Vite emits a single CSS bundle in prod and dev-mode CSS ordering differs).
3. For each of the 13 widths and each of the 2 routes: set the viewport exactly, wait for fonts and images, set the scroll offset, wait for animations to settle, capture.
4. On `/scroll`, scroll through the full page once so every `visibleSections` reveal has fired, then return to the capture scroll position.
5. Capture hover states at all 13 widths on `/` (W3.9 changes all 18 `:hover` blocks).
6. Capture **twice with no code change** and diff — this validates the pipeline (fonts, timing, animation settle).
7. Write the capture procedure down: tool, version, DPR, scroll offsets, settle delay, colour profile.

### Constraints

- **Do not modify any source file.** Zero `src/` changes.
- **Do not capture from a state that has not settled.** `HomePage` renders `null` for 100ms (`HomePage.tsx:75-77,86-88`) and `HomePageScroll` reveals sections progressively. Capturing too early produces a baseline of a loading state.
- **Do not capture with a different DPR than the diff tool will use.** A DPR mismatch produces a uniform scale difference that swamps real regressions.
- **Do not capture the open `<dialog>`, hover states, or focus states into the same baseline as the default state** — they are separate captures, separately diffed.
- **Do not put artifacts in `public/`.** That changes the deployed asset set and would be picked up by TASK-18's file list.
- **Do not proceed to Stage 3 with a partial baseline.** 26 images (or 13 widths × 2 routes + 13 hover = 39). Missing widths are missing coverage.
- Record the exact commit SHA the baseline was captured at. Without it, a later diff cannot be attributed to a specific change.

### Validation Rules

- The **double-capture control diff** must pass before the baseline is accepted. If two captures of an unchanged page differ, the pipeline is non-deterministic and every later diff is meaningless.
- Spot-check widths 4, 5 and 10 (667×375, 844×390, 1180×820) — `risks.md` **R2** identifies these as where removing the 6 order-dependent landscape blocks is most likely to produce a visible diff.
- Spot-check widths 6, 7, 8 (768×1024, 820×1180, 834×1194) — the **currently undesigned** tablet-portrait range that `--bp-md` will change.
- Confirm images and fonts are loaded (no layout-shift artefacts baked into the baseline).

---

## Edge Cases

- **No screenshot tooling is available.** OOS-7: no Playwright, no test framework. Fallbacks in order of preference: (a) an existing global screenshot utility, (b) a browser's DevTools "Capture screenshot" per width, (c) manual OS-level screen capture. **Record which method was used and its known limitations** — if the pipeline is unreliable, say so, because that determines how much confidence the Stage 3 diffs carry.
- **Manual capture at 26+ states is error-prone.** A single mistyped width invalidates one comparison. Double-check each viewport dimension; consider capturing the dimensions into the filename (`390x844__scroll.png`) so a typo is visible.
- **Animation settle time varies by device.** `framer-motion` transitions and the progressive reveal have real durations. Too short a wait captures a mid-transition frame — which will diff against every future capture. Too long wastes time. Measure the longest transition and wait generously.
- **The baseline captures current *defects*.** Widths 1–5 have mobile overflow (W3.7 fixes it), and width 1 specifically has `.menu-section__items` overflowing both edges. **The baseline is not the target state** — it is the "before" state. A diff at those widths is *expected* and must be reviewed, not auto-rejected. Record which diffs are intentional.
- **Hover-state capture is unreliable in headless mode.** Synthetic hover may not trigger a CSS `:hover` that depends on real pointer state. If hover cannot be captured reliably, capture the **computed styles** of every hover rule as a text artifact instead — a weaker but deterministic control.
- **The 200vh fake `<div>`** appended to `document.body` by `HomePage.tsx:36-53` for 500ms could appear in a capture taken during that window. Ensure the settle delay exceeds it.
- **Scroll position affects capture.** `HomePage` sets `overflow: hidden`; `HomePageScroll` forces `scrollTo(0,0)`. Document the exact scroll offset used for each route.
- **WebKit vs Chromium rendering.** `risks.md` **T8**: Chrome-only verification is insufficient — `DefaultCard.css:79-84` relies on `-webkit-box`/`box-orient`/`-webkit-line-clamp`, which TASK-16 (W3.12) removes. **A Chromium-only baseline cannot validate a WebKit-specific change.** If Safari is available, capture the affected widths there too; if not, record that W3.12's verification is limited to Chromium and flag it as a residual risk.
- **Font loading shifts layout.** If `font-display: swap` is in play anywhere, the first paint uses a fallback. Wait for `document.fonts.ready`.
- **Colour profile differences** between the capture and diff tools produce a global tint difference. Use the same tool for both.
- **Baseline must not be captured from a dirty working tree.** An uncommitted change invalidates it. Verify `git status` shows only intended modifications, and record the SHA.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] `npm run build` green; `npm run lint` ≤ Stage 2 baseline.
- [ ] Baseline captured at all 13 widths × 2 routes, from the pre-Stage-3 build.
- [ ] Hover-state baseline captured at all 13 widths on `/`.
- [ ] Double-capture control diff passes (pipeline validated).
- [ ] Every capture recorded with its exact viewport, DPR, route, scroll offset and settle delay.
- [ ] Commit SHA recorded.
- [ ] Capture tool and version recorded; limitations documented.
- [ ] Artifacts stored outside `src/` and `public/`; `git status` confirms nothing was added to the app bundle.

### E2E Test Authoring Guidance

Not applicable — no test framework exists, and **adding one is explicitly out of scope (OOS-7)**. Do not use this task as a justification to install Playwright. If a headless browser is already available globally, use it; otherwise use manual/DevTools capture and document the limitation.

**Cost note:** 39 captures at ~20s each ≈ 15 min, plus the double-capture control run ≈ 30 min total.

---

## Definition of Done

- [ ] Stage 1 and Stage 2 exit criteria met
- [ ] Baseline captured at all 13 widths × both routes
- [ ] Hover-state baseline captured at all 13 widths
- [ ] Double-capture control diff passes
- [ ] Capture procedure documented and reproducible
- [ ] Commit SHA recorded
- [ ] Zero source files modified
- [ ] Build passes
- [ ] Lint does not regress above the Stage 2 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — Stage 3 **entry criteria**; the 13-width verification matrix
- `.opencode/plan/codebase-quality-seo/risks.md` — **R6** (hard gate), R1, R2, R8, T8, R22, OOS-7
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — § *Verification Plan*; § *Non-performance risks introduced*
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 3 row for W3.2 (*"Must be done with a baseline screenshot set captured first"*)