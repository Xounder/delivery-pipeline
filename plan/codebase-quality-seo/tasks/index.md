# Implementation Tasks — Codebase Quality & SEO

**Pipeline context:** `codebase-quality-seo`
**Template:** `.opencode/template/task-template.md`
**Task count:** 22
**Owners:** `senior-frontend` (all tasks — pure frontend SPA; no backend task exists)

---

## Overview

The plan defines **68 work items (W-IDs)** across four hard-separated stages. This index maps every W-ID to exactly one task, records the execution order, the dependency graph, per-file ownership, and the recommended parallelisation.

**Coverage: 68 / 68 W-IDs, each owned by exactly one task. TASK-14 is the sole zero-W-ID task (a verification gate).**

### Stage structure

| Stage | Goal | Tasks | W-IDs |
|---|---|---|---|
| **Stage 1** | Correctness & Foundation | 8 | W1.1–W1.21 (21) |
| **Stage 2** | Types, Architecture & Tooling | 5 | W2.1–W2.17 (17) |
| **Stage 3** | Responsive & Design-System Consolidation | 4 (incl. gate) | W3.1–W3.13 (13) |
| **Stage 4** | SEO & Performance | 5 | W4.1–W4.17 (17) |

**Stage boundaries are hard.** A stage may not begin until the previous stage's exit criteria are met. Within a stage, the batches below may run concurrently.

---

## Task Index

| Task | Title | Stage | W-IDs | Owner | Batch |
|---|---|---|---|---|---|
| [TASK-01](./TASK-01-toolchain-unblock-baseline.md) | Toolchain unblock & visual baseline setup | 1 | — | senior-frontend | S1-A |
| [TASK-02](./TASK-02-viewport-units-height-math.md) | Viewport units & height math | 1 | W1.7, W1.8 | senior-frontend | S1-B |
| [TASK-03](./TASK-03-observer-lifecycle-dead-dom.md) | Observer lifecycle & dead DOM | 1 | W1.4, W1.5, W1.12 | senior-frontend | S1-B |
| [TASK-04](./TASK-04-focus-outlines-divider-positioning.md) | Focus outlines & divider positioning | 1 | W1.14, W1.15 | senior-frontend | S1-B |
| [TASK-05](./TASK-05-button-timer-dead-css-logo-fit.md) | Button timer, dead CSS & logo fit | 1 | W1.9, W1.19, W1.16, W1.21 | senior-frontend | S1-B |
| [TASK-06](./TASK-06-form-pattern-cardgroup-guard.md) | Form pattern & card-group guard | 1 | W1.13, W1.10 | senior-frontend | S1-B |
| [TASK-07](./TASK-07-section-lookup-overflow-icon-semantics.md) | Section lookup, overflow, icon semantics | 1 | W1.6, W1.11, W1.18, W1.20 | senior-frontend | S1-C |
| [TASK-08](./TASK-08-heading-outline-h1.md) | Heading outline & `<h1>` | 1 | W1.17 | senior-frontend | S1-D |
| [TASK-09](./TASK-09-data-types-discriminated-union.md) | Data types & discriminated union | 2 | W2.1, W2.3, W2.7 | senior-frontend | S2-A |
| [TASK-10](./TASK-10-section-registry-single-source.md) | Section registry & single source of truth | 2 | W2.2, W2.5, W2.10, W2.11, W2.12, W2.13 | senior-frontend | S2-A |
| [TASK-11](./TASK-11-button-consolidation.md) | Button component consolidation | 2 | W2.4 | senior-frontend | S2-B |
| [TASK-12](./TASK-12-stable-keys-paragraph-dead-code.md) | Stable keys, paragraph semantics & dead code | 2 | W2.8, W2.9, W2.14 | senior-frontend | S2-C |
| [TASK-13](./TASK-13-error-boundary-tooling.md) | Error boundary & tooling | 2 | W2.6, W2.15, W2.16, W2.17 | senior-frontend | S2-B |
| [TASK-14](./TASK-14-stage3-screenshot-baseline-gate.md) | **Stage 3 screenshot baseline gate** | 3 | — *(gate)* | senior-frontend | S3-A |
| [TASK-15](./TASK-15-tokens-breakpoint-migration.md) | Tokens & breakpoint migration | 3 | W3.1, W3.2, W3.3, W3.4 | senior-frontend | S3-B |
| [TASK-16](./TASK-16-css-literal-consolidation.md) | CSS literal consolidation | 3 | W3.5, W3.7, W3.9, W3.11, W3.12, W3.13 | senior-frontend | S3-C |
| [TASK-17](./TASK-17-semantic-landmarks-motion-constants.md) | Semantic landmarks & motion constants | 3 | W3.6, W3.8, W3.10 | senior-frontend | S3-C |
| [TASK-18](./TASK-18-static-seo-head-crawl-files.md) | Static SEO head, crawl files & `<noscript>` | 4 | W4.1, W4.2, W4.4, W4.5 | senior-frontend | S4-A |
| [TASK-19](./TASK-19-jsonld-alt-lang-metadata-module.md) | JSON-LD, alt text, lang semantics & metadata module | 4 | W4.3, W4.6, W4.9, W4.14, W4.16 | senior-frontend | S4-B |
| [TASK-20](./TASK-20-image-cls-form-labels-article-time.md) | Image CLS, form labels, `<article>`/`<time>` | 4 | W4.7, W4.8, W4.10 | senior-frontend | S4-C |
| [TASK-21](./TASK-21-security-headers-dead-font-threshold-note.md) | Security headers, dead font & threshold note | 4 | W4.11, W4.15, W4.17 | senior-frontend | S4-A |
| [TASK-22](./TASK-22-code-splitting-lazymotion.md) | Code splitting & `LazyMotion` | 4 | W4.12, W4.13 | senior-frontend | S4-C |

---

## Execution Order

### Stage 1 — Correctness & Foundation

**Entry:** none.
**Exit:** `npm run lint` and `npm run build` green (post-TASK-01); the four Stage 1 invariants verified; no new console errors.

```
S1-A   TASK-01
         |
S1-B   TASK-02  TASK-03  TASK-04  TASK-05  TASK-06      (5 concurrent)
         |
S1-C   TASK-07  (requires TASK-03, TASK-05)
         |
S1-D   TASK-08  (requires TASK-03, TASK-05, TASK-06, TASK-07)
```

### Stage 2 — Types, Architecture & Tooling

**Entry:** Stage 1 exit met.
**Exit:** discriminated union in place, single-source registry, route tree rewritten with an error boundary, tooling green.

```
S2-A   TASK-09  TASK-10                                (2 concurrent)
         |
S2-B   TASK-11  TASK-13  (TASK-13 requires TASK-10)    (2 concurrent)
         |
S2-C   TASK-12  (requires TASK-09, TASK-10, TASK-11)
```

### Stage 3 — Responsive & Design-System Consolidation

**Entry:** Stage 2 exit met.
**Exit:** **13-width screenshot diff clean at every Stage 3 task boundary** (R6). This is the plan's primary regression gate.

```
S3-A   TASK-14  (capture baseline; modifies no source)
         |
S3-B   TASK-15  (token foundation — every later Stage 3 CSS task depends on it)
         |
S3-C   TASK-16  TASK-17                                (2 concurrent — CSS-only vs TSX-only)
```

### Stage 4 — SEO & Performance

**Entry:** Stage 3 exit met, and the TASK-14 baseline is stable and trusted.
**Exit:** Rich Results valid, external social validators pass, deployed headers verified, animations verified in a real browser.

```
S4-A   TASK-18  TASK-21                                (2 concurrent — disjoint files)
         |
S4-B   TASK-19  (requires TASK-18)
         |
S4-C   TASK-20  TASK-22                                (2 concurrent — disjoint files)
```

---

## Dependency Graph

**Legend:** `→` hard dependency (same stage) · `⇢` ordering-only (file ranges do not overlap) · `‖‖` stage gate.

```
TASK-01 ⇢ TASK-02, TASK-03, TASK-04, TASK-05, TASK-06, TASK-07, TASK-08

Stage 1
  TASK-02 ─┐
  TASK-03 ─┼→ TASK-07 → TASK-08
  TASK-05 ─┤
  TASK-06 ─┴──────────→ TASK-08

Stage 2
  TASK-09 ─┬→ TASK-11 ─┐
  TASK-10 ─┴→ TASK-13  ├→ TASK-12
            └→ TASK-12 ─┘

Stage 3
  TASK-14 → TASK-15 → TASK-16
                  └→ TASK-17

Stage 4
  TASK-18 → TASK-19 → TASK-20
  TASK-21 ⇢ TASK-22
  TASK-17, TASK-13, TASK-15 → TASK-22
```

### Non-obvious dependency justifications

| Edge | Why it cannot be parallel |
|---|---|
| TASK-01 → everything | `package.json:14`'s self-referential `"portfolio": "file:"` dependency makes `npm run build` fail. Nothing can be verified. |
| TASK-03, TASK-05 → TASK-07 | Both edit `HomePageScroll.tsx` and `MenuBar.tsx` — max-1-concurrent-editor files. |
| TASK-07 → TASK-08 | Overlapping ranges in `HomePageScroll.tsx:146-147` and `MenuBar.tsx:26`. |
| TASK-10 → TASK-13 | Both rewrite `main.tsx`'s route tree region. |
| TASK-09, TASK-10, TASK-11 → TASK-12 | `MenuBar.tsx:35,40,61` is written by TASK-10 and TASK-11. |
| TASK-14 → TASK-15 | The screenshot baseline must exist before any Stage 3 change can be diffed (R6). |
| TASK-15 → TASK-16 | `feasibility.md`: W3.5, W3.7 and W3.9 all *"depend on W3.1"*. `impact-analysis.md`: *"W3.5 must follow W3.2 per file."* |
| TASK-18 → TASK-19 | Both edit `index.html`, and adjacent line ranges (`:3-8` vs `:2`, `:12-15`). |
| TASK-19 → TASK-20 | Both edit `DefaultCard.tsx:15-31` and `data.ts`. |
| TASK-21 ⇢ TASK-22 | Both edit `HomePageScroll.tsx`, but `:76` vs `:12` — **ordering only**, driven by the stage batch structure. |

---

## Ownership Mapping

| Area | Tasks | Primary file cluster |
|---|---|---|
| Toolchain, build, lint | TASK-01, TASK-13 | `package.json`, `eslint.config.js`, `vite.config.ts` |
| Viewport math & layout | TASK-02, TASK-15, TASK-16 | `index.css`, `HomePage.css`, all component CSS |
| Observer, scroll & timing | TASK-03, TASK-07, TASK-17 | `HomePageScroll.tsx`, `MenuBar.tsx`, `ArrowSectionButton.tsx` |
| Focus, outline & semantics | TASK-04, TASK-08, TASK-17, TASK-20 | `MenuSectionButton.css`, `HomePageScroll.tsx`, `ContactForm.tsx`, `ExperienceCardInfo.tsx` |
| Data layer & registry | TASK-09, TASK-10, TASK-12, TASK-19 | `data.ts`, `iconsData.ts`, `types.ts`, `seo.ts` |
| Components & dead code | TASK-05, TASK-06, TASK-11 | `MenuBar.tsx`, `ContactForm.tsx`, `DefaultCardGroup.tsx` |
| Metadata & SEO | TASK-18, TASK-19 | `index.html`, `public/robots.txt`, `public/sitemap.xml` |
| Performance & delivery | TASK-20, TASK-21, TASK-22 | `main.tsx`, `HomePage.tsx`, `vercel.json` |

**Single-owner rule:** every file modification is owned by exactly one task at a time. Where two tasks touch the same file, the merge strategy below is **sequential**.

---

## File Overlaps

Only files touched by more than one task are listed. **Every strategy is `Sequential` unless stated otherwise** — no file in this plan is edited by two tasks in the same batch.

### Hot files (highest merge risk)

| File | W-IDs | Tasks (chronological) | Strategy |
|---|---|---|---|
| `src/pages/HomePage/HomePage.tsx` | **13** | TASK-02 → TASK-07 → TASK-10 → TASK-17 → TASK-22 | **Sequential.** Ledger must be walked in order. `:23,28` (TASK-10's unresolved `MOTION_VALUE`) and `:97,124,135` (TASK-17's transitions) are the collision points. |
| `src/components/MenuSectionButton/MenuSectionButton.css` | **9** | TASK-04 → TASK-15 → TASK-16 | **Sequential.** TASK-15 owns it completely in Stage 3; TASK-16 follows. |
| `src/pages/HomePage/HomePageScroll.tsx` | **9** | TASK-03 → TASK-07 → TASK-08 → TASK-10 → TASK-17 → TASK-21 → TASK-22 | **Sequential.** Four of the nine land in Stage 3/4; verify the observer wiring after TASK-22's `m`-component switch. |
| `src/components/DefaultCard/DefaultCard.css` | **7** | TASK-05 → TASK-15 → TASK-16 | **Sequential.** TASK-16 owns colours, hover, `vh` and legacy props after TASK-15's media queries. |
| `src/data/data.ts` | **6** | TASK-05 → TASK-09 → TASK-10 → TASK-12 → TASK-20 | **Sequential.** TASK-19 is read-only here. |

### Supporting files

| File | Tasks | Strategy |
|---|---|---|
| `src/index.css` | TASK-02, TASK-15, TASK-16, TASK-19, TASK-21 | **Sequential.** Ledger: TASK-02 → TASK-15 → TASK-16 → TASK-19 → TASK-21. `:2`, `:6`, `:49` are close together — a merge can restore the dead Inter face or drop colour tokens. |
| `src/components/MenuBar/MenuBar.tsx` | TASK-05, TASK-07, TASK-08, TASK-10, TASK-12, TASK-13, TASK-17, TASK-20 | **Sequential** (8 owners — the second-most contested file). `:45` (TASK-17) and `:29,32` (TASK-20) are isolated; `:8-13,40,61` (TASK-10) vs `:35,40,61` (TASK-12) overlap. |
| `src/main.tsx` | TASK-08, TASK-13, TASK-22 | **Sequential.** `:4-19` (TASK-22) sits adjacent to `:22-50` (TASK-13) — **verify both `ErrorBoundary` and `Suspense` survive the merge.** |
| `src/components/DefaultCard/DefaultCard.tsx` | TASK-08, TASK-11, TASK-17, TASK-19, TASK-20 | **Sequential.** Four consecutive tasks touch `:15-31`/`:18`. **Re-verify after every merge: exactly one `<article>` wrapper, and both the `alt` (W4.6) and `width`/`height` (W4.7) attributes present.** |
| `index.html` | TASK-18, TASK-19 | **Sequential.** TASK-18 populates `:3-8` and adds `:9-12`; TASK-19 appends the JSON-LD script at `:12-15` and may touch `:2`. |
| `src/data/iconsData.ts` | TASK-09, TASK-10, TASK-12, TASK-19, TASK-20 | **Sequential.** TASK-19 owns the W4.16 documentation comment; TASK-20 owns W4.10. |
| `vercel.json` | TASK-21 | **No overlap** — single owner. Treat as a whole-file review (JSON validity + SPA rewrite intact). |
| `src/components/AboutMe/AboutMe.tsx` | TASK-19, TASK-20 | **Sequential.** `:10` (alt) and `:13` (dimensions) are 3 lines apart in the same `<img>` — **a merge must keep both.** |
| `src/components/ContactForm/ContactForm.tsx` | TASK-06, TASK-08, TASK-20 | **Sequential.** `:19`, `:23-25`, `:28-33` — distinct ranges, low merge risk. |
| `src/components/DefaultSectionButton/DefaultSectionButton.tsx` | TASK-11, TASK-20 | **Sequential.** Read post-TASK-11; `:8` may have moved. |
| `src/components/ArrowSectionButton/ArrowSectionButton.tsx` | TASK-11, TASK-17 | **Sequential.** **TASK-11 may have deleted this file** — TASK-17 must re-locate the `<nav>` before editing. |
| `src/components/MenuBar/MenuBar.css` | TASK-15, TASK-16 | **Sequential.** |
| `src/pages/HomePage/HomePage.css` | TASK-02, TASK-15, TASK-16 | **Sequential.** |
| `src/components/AboutMe/AboutMe.css` | TASK-15, TASK-16 | **Sequential.** |
| `src/components/ContactForm/ContactForm.css` | TASK-15, TASK-16 | **Sequential.** |
| `src/components/Button/Button.css` | TASK-04, TASK-11, TASK-15, TASK-16 | **Sequential.** |
| `src/components/InfoCard/InfoCard.css` | TASK-05, TASK-15, TASK-16 | **Sequential.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.css` | TASK-09, TASK-15, TASK-16 | **Sequential.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.tsx` | TASK-06, TASK-09, TASK-12, TASK-17 | **Sequential.** Ranges are distinct after TASK-09/12 completed. |
| `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx` | TASK-11, TASK-20 | **Sequential.** |
| `src/components/InfoCard/InfoCard.tsx` | TASK-08 (only modifier) | **No conflict** — read-only elsewhere. |

### New files (single owner, no overlap)

`public/robots.txt` (TASK-18) · `public/sitemap.xml` (TASK-18) · manifest / apple-touch-icon assets (TASK-18) · `src/data/seo.ts` (TASK-19) · `src/data/types.ts` (TASK-09) · `src/components/ErrorBoundary/` (TASK-13) · TASK-14 baseline captures.

---

## Parallelization Plan

**Rules applied:**
1. Stage boundaries are hard — no two stages overlap.
2. Only tasks listed in the same batch below may run concurrently.
3. A batch containing two tasks must have **disjoint file sets** (verified against the overlap table above).
4. Maximum concurrency in this plan: **5** (S1-B).

| Batch | Tasks | Concurrency | Why these can run together |
|---|---|---|---|
| **S1-A** | TASK-01 | 1 | Everything depends on it. |
| **S1-B** | TASK-02, TASK-03, TASK-04, TASK-05, TASK-06 | 5 | Each touches a disjoint file set: `index.css`+`HomePage.css`; `HomePageScroll.tsx`+observer; `Button.css`+`MenuSectionButton.css`; `MenuBar.tsx`+`DefaultCard.css`; `ContactForm.tsx`+`DefaultCardGroup.tsx`. |
| **S1-C** | TASK-07 | 1 | Waits on `HomePageScroll.tsx` and `MenuBar.tsx` from TASK-03/TASK-05. |
| **S1-D** | TASK-08 | 1 | Waits on TASK-07's overlapping ranges. |
| **S2-A** | TASK-09, TASK-10 | 2 | `data.ts`/`iconsData.ts`/`types.ts` vs `componentsMap.ts`/`HomePage.tsx`/`HomePageScroll.tsx`. *(Both touch `data.ts` conceptually; TASK-10's `DATA_SECTIONS` reads the union TASK-09 produces — coordinate the read, do not run a second writer.)* |
| **S2-B** | TASK-11, TASK-13 | 2 | Button components/CSS vs `main.tsx` + `eslint.config.js` + `vite.config.ts`. Disjoint. |
| **S2-C** | TASK-12 | 1 | Waits on TASK-09, TASK-10 and TASK-11. |
| **S3-A** | TASK-14 | 1 | Gate. Captures the baseline; modifies no source. |
| **S3-B** | TASK-15 | 1 | Token foundation; every Stage 3 CSS task reads its `:root`. |
| **S3-C** | TASK-16, TASK-17 | 2 | **TASK-16 is CSS-only** (11 `.css` files); **TASK-17 is TSX-only** (6 `.tsx` files). Zero overlap. |
| **S4-A** | TASK-18, TASK-21 | 2 | `index.html`+`public/` vs `vercel.json`+`index.css`+`HomePageScroll.tsx`. Disjoint. |
| **S4-B** | TASK-19 | 1 | Waits on TASK-18's `index.html`. |
| **S4-C** | TASK-20, TASK-22 | 2 | `DefaultCard.tsx`+`data.ts`+4 components vs `main.tsx`+`HomePage.tsx`+`HomePageScroll.tsx`. Disjoint. |

### Critical path

`TASK-01 → TASK-03/05 → TASK-07 → TASK-08 → TASK-10 → TASK-11 → TASK-12 → TASK-14 → TASK-15 → TASK-16 → TASK-19 → TASK-20`

Every task on the critical path is single-owner (never batched), because each is blocked by a max-1-concurrent-editor file.

---

## W-ID Coverage

### Stage 1 (21 / 21)

| W-ID | Task | W-ID | Task | W-ID | Task |
|---|---|---|---|---|---|
| W1.1 | TASK-01 | W1.8 | TASK-02 | W1.15 | TASK-04 |
| W1.2 | TASK-01 | W1.9 | TASK-05 | W1.16 | TASK-05 |
| W1.3 | TASK-01 | W1.10 | TASK-06 | W1.17 | TASK-08 |
| W1.4 | TASK-03 | W1.11 | TASK-07 | W1.18 | TASK-07 |
| W1.5 | TASK-03 | W1.12 | TASK-03 | W1.19 | TASK-05 |
| W1.6 | TASK-07 | W1.13 | TASK-06 | W1.20 | TASK-07 |
| W1.7 | TASK-02 | W1.14 | TASK-04 | W1.21 | TASK-05 |

### Stage 2 (17 / 17)

| W-ID | Task | W-ID | Task | W-ID | Task |
|---|---|---|---|---|---|
| W2.1 | TASK-09 | W2.7 | TASK-09 | W2.13 | TASK-10 |
| W2.2 | TASK-10 | W2.8 | TASK-12 | W2.14 | TASK-12 |
| W2.3 | TASK-09 | W2.9 | TASK-12 | W2.15 | TASK-13 |
| W2.4 | TASK-11 | W2.10 | TASK-10 | W2.16 | TASK-13 |
| W2.5 | TASK-10 | W2.11 | TASK-10 | W2.17 | TASK-13 |
| W2.6 | TASK-13 | W2.12 | TASK-10 | | |

### Stage 3 (13 / 13)

| W-ID | Task | W-ID | Task | W-ID | Task |
|---|---|---|---|---|---|
| W3.1 | TASK-15 | W3.6 | TASK-17 | W3.11 | TASK-16 |
| W3.2 | TASK-15 | W3.7 | TASK-16 | W3.12 | TASK-16 |
| W3.3 | TASK-15 | W3.8 | TASK-17 | W3.13 | TASK-16 |
| W3.4 | TASK-15 | W3.9 | TASK-16 | | |
| W3.5 | TASK-16 | W3.10 | TASK-17 | | |

### Stage 4 (17 / 17)

| W-ID | Task | W-ID | Task | W-ID | Task |
|---|---|---|---|---|---|
| W4.1 | TASK-18 | W4.7 | TASK-20 | W4.13 | TASK-22 |
| W4.2 | TASK-18 | W4.8 | TASK-20 | W4.14 | TASK-19 |
| W4.3 | TASK-19 | W4.9 | TASK-19 | W4.15 | TASK-21 |
| W4.4 | TASK-18 | W4.10 | TASK-20 | W4.16 | TASK-19 |
| W4.5 | TASK-18 | W4.11 | TASK-21 | W4.17 | TASK-21 |
| W4.6 | TASK-19 | W4.12 | TASK-22 | | |

**Duplicate check: none. Every W-ID appears exactly once.**

---

## Out of Scope — No Task Created

Per `risks.md` OOS-1..OOS-7, **none of the following is a task.** If an implementer "improves" these while executing a task above, that is a scope violation.

| ID | Item | Why no task |
|---|---|---|
| **OOS-1** | Contact form's simulated submit | No backend; the submit stays simulated. W4.8 improves the label only. |
| **OOS-2** | Progressive per-section reveal on `/scroll` | Working as designed. TASK-17 and TASK-21 may touch its transition/threshold **comments and constants only**. |
| **OOS-3** | Scroll-restoration fake `<div>` + 500ms timer | Working as designed. TASK-17 may name the `200vh` constant; the hack stays. |
| **OOS-4** | Nested interactive content (`<Link>` wrapping a `<button>`) | Out of scope. TASK-17 and TASK-20 must not restructure it. |
| **OOS-5** | Duplicate asset usage / asset replacement | Documented only (W4.16, TASK-19). No asset, markup or content change. |
| **OOS-6** | Actual prerendering | No tooling in this plan. W4.14 (TASK-19) lands the **metadata module only**. |
| **OOS-7** | Adding a test framework | None exists. All verification is manual, plus external SEO validators. |

---

## Verification Gates

| Gate | When | Blocks |
|---|---|---|
| Build/lint green | Every task | The next task |
| **13-width screenshot diff clean** | Every Stage 3 task boundary | All of Stage 4 |
| TASK-14 baseline captured | Before TASK-15 | TASK-15 |
| Google Rich Results Test | TASK-19 (needs deployment) | Stage 4 exit |
| Facebook / LinkedIn / Twitter validators | TASK-18 (needs deployment) | Stage 4 exit |
| Deployed response headers + `/scroll` 200 | TASK-21 (needs deployment) | Stage 4 exit |
| Animation inventory in a real browser | TASK-22 | Stage 4 exit |

**13 widths:** `360×640`, `390×844`, `430×932`, `667×375`, `844×390`, `768×1024`, `820×1180`, `834×1194`, `1024×1366`, `1180×820`, `1280×800`, `1440×900`, `1920×1080`.
**Breakpoints:** `600`, `768`, `1024`, `1440`, `1920`.
**Routes:** `/` and `/scroll`.

---

## Known Issues & Open Questions

| Item | Where | Handling |
|---|---|---|
| **Build blocker** | `package.json:14` — self-referential `"portfolio": "file:"` | TASK-01. `npm run build` and `npm run test` both fail before running any script. |
| **`[ERR_PNPM_IGNORED_BUILDS]`** | `esbuild@0.25.12` build script blocked | TASK-01. Requires `pnpm approve-builds`; blocks build/test today. |
| **No `test` script** | `package.json` | OOS-7. No task adds one. |
| **Open Question 1** — is `#646cff` (`Button.css:15`) intentional? | TASK-16 | Gate. Default: declare as a token (pure rename). Do not replace. |
| **Open Question 2** — `MOTION_VALUE` `200` vs `100` (R15) | TASK-10, TASK-17 | Extract as a constant; **do not resolve.** |
| **Open Question 3** — divider positioning intent (W1.15) | TASK-04 | Resolve or document; do not silently restyle. |
| **Open Questions 4–5** | `feasibility.md` | Review at Stage 2 exit; they gate W2.10/W2.16 decisions. |
| **Canonical production origin** | TASK-18, TASK-19 (O6) | Confirm before writing. One value, one constant (`src/data/seo.ts`). |
| **`<article>` vs `<a>`-wrapping-`<button>`** | TASK-17, TASK-20 | OOS-4 forbids restructuring. Record the placement decision; escalate if unresolvable. |
| **`lang="pt-BR"` vs `:lang(pt-BR)`** | TASK-19 (W4.9) | Confirm the site's actual language. Change `:lang()`, **not** the declared language, unless it is genuinely wrong. |
| **W3.5 colour literals** | TASK-15 → TASK-16 | TASK-15 declares tokens only; TASK-16 replaces the literals. Do not conflate. |
| **Deployment-dependent validation** | TASK-18, TASK-19, TASK-21, TASK-22 | Four tasks cannot be fully verified locally. Record the pending status honestly. |

---

## Conventions Applied to Every Task

- **Must Not Change** section restates all five project invariants verbatim, with the task's own W-IDs annotated where an item is the documented exception shape.
- **Edge Cases** are task-specific: race conditions, empty/null states, API or storage failures, missing optional fields, limits, and regressions in existing behaviour. Not template boilerplate.
- **File Overlap Warning** lists every file the task modifies that another task also modifies, with the recommended merge strategy.
- **Acceptance Criteria** require `npm run lint` (no regression above the TASK-01 baseline) and `npm run build` (passes).
- **Testing** records that no test framework exists, and every task carries an explicit cost note for its manual matrix.
- Stage 3 tasks require a clean 13-width diff at their boundary; TASK-16 additionally requires a hover-state diff.
