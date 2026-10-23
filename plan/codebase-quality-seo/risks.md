# Risk Assessment — `codebase-quality-seo`

## Overview

### Objective

Identify every way the 68 work items can damage the approved invariants (same functionality, same branding) or fail in production, and define the mitigation that must be in place *before* the work starts.

### Scope

All 4 stages. Risk is dominated by Stage 3 (CSS consolidation) and by the currently-unverified toolchain.

### Recommendation

Approach A with a **mandatory screenshot baseline captured before Stage 3 begins** and a **verified green lint/build baseline before Stage 1 begins**. These two baselines are the primary controls for this entire plan.

---

## Risk Matrix

| # | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R1 | **Visual regression from token migration.** ~24 colour literals, 10 layout literals, 28 media queries, 18 `:hover` blocks rewritten | **High** | **High** | Capture a 13-width screenshot baseline before Stage 3. Diff after every file, not just at stage exit. Every token value must be byte-identical to the literal it replaces. |
| R2 | **Order-dependent media queries change behaviour when the 6 `(orientation: landscape)` blocks are removed** (`AboutMe.css:76`, `ContactForm.css:111`, `DefaultCard.css:188`, `HomePage.css:76`, `MenuBar.css:117`, `MenuSectionButton.css:79`) | **High** | **High** | These currently win over `min-width:600px` purely by source order. Replace with intent-equivalent `--bp-sm` + `max-height` rules and verify widths **4 (667×375), 5 (844×390) and 10 (1180×820)** explicitly, in addition to all 13. |
| R3 | **`100vh` → `100dvh` root fix regresses** — this exact change was attempted in commit `1fed2df` and reverted in `1e7e6e3` | **High** | **High** | Sequence W1.7 **before** W1.8. Declare `100dvh` *after* a `100vh` fallback so non-supporting browsers degrade. Verify widths 1–5 and 6–8 on a real device, not just DevTools. Do not touch `HomePage.css` in the same change. |
| R4 | **`HomePage` height math overflows** — `height:100vh` + `padding-top:40px` at `min-width:600px` = `100vh + 40px` always (`HomePage.css:16,68`). This is the `403ab6b` ↔ `986d27d` ping-pong | **High** | Medium | W1.7 and W1.8 must land together as one reviewed change. Verify no horizontal/vertical scrollbar at widths 6–13. |
| R5 | **Unverified lint/build baseline.** Validation was blocked by pnpm's `ERR_PNPM_IGNORED_BUILDS` policy | **Certain** (already occurred) | Medium | W1.1 (remove `"portfolio": "file:"`) is the probable root cause and is Critical. W1.3 must record the exact baseline counts before any refactor starts, so pre-existing and newly-introduced failures are distinguishable. |
| R6 | **Baseline screenshot set is never captured**, making R1–R4 undetectable | Medium | **High** | Hard gate: Stage 3 entry criterion. No Stage 3 task may start without it. |
| R7 | **Heading re-levelling changes rendered size** — CSS may target `h1`–`h4` by tag rather than class (`AboutMe.css:38`, `DefaultCard.css:67`, `ContactForm.css:14`, `InfoCard.css:17`, `MenuSectionButton.css:27`) | Medium | Medium | Grep every CSS file for element selectors before changing any heading tag. Compensate in CSS so rendered size is unchanged. Verify all 13 widths. |
| R8 | **W1.14 focus-outline change alters the focus ring appearance** | Medium | Low | Intentional a11y improvement; scope it to `:focus-visible` only so pointer users see no change. Use existing branding colours. |
| R9 | **`LazyMotion` / `domAnimation` silently disables animations** if features are not registered correctly | Medium | **High** | W4.13 is conditional. All animations used (opacity, x, y) are in the `dom` subset, but `initial`/`animate` failing silently produces a *plausible-looking* page with no motion. Visual check of every animation on both routes is mandatory. Fallback: revert to the full import. |
| R10 | **Code splitting adds user-visible Suspense latency on the main route** | Low | Medium | Verify cold-load timing on the deployed preview. `"Carregando..."` (`HomePageScroll.tsx:121`) becomes briefly visible — confirm acceptable, or scope W4.12 to `/scroll` only. |
| R11 | **`@types/react-router-dom@^5.3.3` replaced (W2.15) surfaces type errors** across `main.tsx`, `MenuBar.tsx` | Medium | Medium | Typecheck immediately after. v5→v7 type differences around `Route`/`Routes` may require adjustments; those are compile-time only, no behaviour change. |
| R12 | **New lint plugins (W2.16) surface many pre-existing violations**, mistaken for regressions | **High** | Low | Baseline the new plugin output separately at Stage 2 entry. Pre-existing violations are not Stage 2 regressions. |
| R13 | **`object-fit: fill` → `contain` (W1.21) changes logo rendering** | Low | Medium | Explicitly gated: change **only** if pixel-verified equivalent on the shipped logos. Otherwise document and leave. |
| R14 | **W1.15 divider repositioning changes a rendering that is itself the current design** | Medium | Low | Confirm intent before changing. If ambiguous, document and leave rather than guess. |
| R15 | **W2.10 changes first-render animation** — `mainMotionValue` initial is `200` (`HomePage.tsx:23`) but `MOTION_VALUE` is `100` (`:28`) | Medium | Low | `feasibility.md` Open Question 2 must be answered before this item is implemented. Do not silently reconcile. |
| R16 | **`W1.15`/`W3.5` undecided brand decisions** (`#646cff`, divider layout) | **Certain** (open) | Low | Both are listed as Open Questions in `feasibility.md`. Block the specific task, not the stage. |
| R17 | **Error boundary (W2.6) swallows errors** that previously surfaced to the console, hiding regressions | Low | Medium | Log caught errors to the console in development. The boundary must render a visible fallback. |
| R18 | **Catch-all route (W2.6) masks genuine routing bugs** by rendering a fallback instead of failing | Low | Medium | Log unmatched paths. The rewrite in `vercel.json` stays untouched. |
| R19 | **Security headers (W4.11) break the SPA** — a misconfigured `Content-Security-Policy` or `X-Frame-Options` can block assets | Low | **High** | Start with the safe subset only (`X-Content-Type-Options`, `Referrer-Policy`, `Permissions-Policy`). Verify on the deployed preview before merging. No `CSP` in this plan. |
| R20 | **A future prerender is attempted against an app still touching `window`/`document` during module init** | Medium | Medium | W4.14 audits module-scope DOM access. Until W4.14 lands, prerendering is expected to fail — that is documented, not hidden. |
| R21 | **Regression from reworking `data.ts` (6 work items)** — the single most-edited file | Medium | **High** | Strictly serialise W2.1, W2.3, W4.3, W4.6, W4.10, W4.16. `data.ts` is the source of truth for 14 cards and all JSON-LD. |
| R22 | **13-width matrix is aspirational and gets skipped under time pressure** | **High** | **High** | It is encoded as an explicit per-task acceptance criterion for W3.1–W3.7 (and Stage 1 CSS items). Skipping it voids the plan's primary control. |
| R23 | **No test framework exists** — no automated regression net for any of the 68 items | **Certain** | Medium | Accepted. Verification is manual. If a test harness is added (Stage 1.5 custom option), it becomes a new dependency and is out of the approved scope. |

---

## Technical Risks

### T1 — Viewport units (highest historical-risk area)

`index.css:26-27` uses `height:100vh; width:100vw`. `100vh` is the *large* viewport on mobile (URL bar collapsed) → content taller than the visible area. `100vw` includes the desktop scrollbar → horizontal overflow. `HomePage.css:16` compounds it with `height:100vh` and `HomePage.css:68` adds `padding-top:40px` at `min-width:600px`, making the element permanently `100vh + 40px`.

Git history confirms this is the app's chronic failure point:

| Commit | Message |
|---|---|
| `1fed2df` | modified vh to dvh in home-page |
| `1e7e6e3` | **Revert** "modified vh to dvh in home-page" |
| `7e8cc0c` | added padding-bottom cards mobile |
| `3abc297` | fixed home-page scroll-to-end-page home-page mobile |
| `668502b` | fixed home-page scroll-to-end-page home-page |
| `3233126` | **Revert** "fixed home-page scroll-to-end-page home-page mobile" |
| `403ab6b` | added padding-top min-width 600 |
| `986d27d` | modified padding-top to 0 |

Two reverts in the same area. **Mitigation:** W1.7 lands alone, before W1.8, with a device check — not a DevTools-only check.

### T2 — Scroll-restoration hack

`HomePage.tsx:36-53` appends a `200vh` `div` to `document.body`, calls `window.scrollTo({behavior:"smooth"})` to `fakeDiv.offsetTop`, then removes it after a **fixed 500ms** `setTimeout`. It exists solely to work around the browser restoring scroll on reload, because `HomePage` disables scrolling (`overflow:hidden` at `:72-73`) and renders `null` for 100ms (`:75-77,86-88`).

Risks: `offsetTop` depends on a live layout box; the 500ms removal races the smooth scroll on slow devices; it mutates `document.body` (which is also the parent of `#root`, `index.html:10`), so a style recalc can occur mid-animation.

**Mitigation:** this hack must be **replaced by a mechanism, not tuned** — `history.scrollRestoration = "manual"` (set in an effect) or `window.scrollTo(0,0)` with `behavior:"auto"`. Do not adjust the 500ms value; that is the tuning trap that produced commits `668502b` → `3abc297` → `3233126`. **Not planned as a change in this iteration** (the fake div is at `HomePage.tsx:36-53`; it is out of the 68 items) — recorded here so a future iteration starts from this analysis rather than re-deriving it.

### T3 — Observer lifecycle

`HomePageScroll.tsx:83` — `useEffect(..., [currentSection])` where `currentSection` is set **inside the observer callback** (`:51`). Each section change tears down and re-creates the `IntersectionObserver` and re-runs `document.querySelectorAll("section")` (`:79`). Self-re-triggering.

**Mitigation:** W1.4 (empty dependency array) must land with W1.5 in the same change — removing the dead `querySelector` block (`:58-68`) is part of the same file and the same concern.

### T4 — Global selector coupling

`HomePageScroll.tsx:79` observes **every** `<section>` in the document; `dataset.section || "About Me"` (`:48`) silently mislabels any foreign section. Mitigation: W2.12 scopes to a container ref.

### T5 — `threshold: 0.5` and tall sections

`HomePageScroll.tsx:76` — a section taller than the viewport can never reach 50% intersection → `visibleSections` never gains the key → `animate={... ? ... : {}}` (`:116`) holds `opacity: 0, y: 300` forever. Silent content loss. **The progressive reveal itself is out of scope**; only the risk is recorded (W4.17) so it is not mistaken for a new defect.

### T6 — Global style mutation

`HomePage.tsx:72-82` sets `overflow:hidden` on **both** `document.body` and `document.documentElement`, then restores them to `"visible"` / `"auto"` — values that were never set beforehand. The document keeps a mutated inline `overflow` for the rest of its life.

**Mitigation:** W1.6 captures and restores the actual prior values.

### T7 — Latent crash surfaces

`DefaultCardGroup.tsx:35` dereferences `selectedData.cards.length` before the `selectedData &&` guard at `:39`, hidden by an unsafe `as keyof typeof` cast at `:33`. `iconsData[sections.indexOf(currentSection)]` (`HomePage.tsx:101`, `HomePageScroll.tsx:96`) yields `undefined` on `-1` → `icons.map` throws in `MenuBar.tsx:26`. Currently unreachable; a single bad `dataset.section` value makes it reachable. Mitigation: W1.10 (reorder guard) + W1.11/W2.3 (typed registry).

### T8 — Legacy CSS in WebKit

`DefaultCard.css:79-84` relies on `display: -webkit-box` / `box-orient` / `-webkit-line-clamp`; `Button.css:20` uses the removed `-webkit-focus-ring-color`. Chrome-only verification is insufficient. Mitigation: W3.12 verification must include WebKit/Safari.

---

## Delivery Risks

| Risk | Detail | Mitigation |
|---|---|---|
| **D1 — Scope creep into a redesign** | 68 items touching every file makes it tempting to "improve" visuals. The plan explicitly forbids it. | The 5 invariants in `index.md` are the gate. Any item that cannot satisfy them is re-specified, not implemented. Every task description must restate them. |
| **D2 — Unverified baseline is discovered late** | W1.3 could reveal that `tsc -b` already fails, changing Stage 1's size estimate | W1.3 is the Stage 1 gate. Do not start any refactor task before it. |
| **D3 — Pre-existing lint failures misattributed** | New lint plugins and type changes surface old problems | Baseline counts recorded in `feasibility.md`; W2.16 gets its own separate baseline. |
| **D4 — Stage boundaries blurred** | A task that "needs" a later stage's refactor will be smuggled into an earlier one | Dependencies are explicit in `feasibility.md` (e.g. W1.11 → W2.3). Split the task if it cannot be done standalone. |
| **D5 — Parallel edits to `data.ts`** | 6 work items target one file; concurrent edits lose work | `impact-analysis.md` marks `data.ts`, `HomePage.tsx`, `HomePageScroll.tsx`, `MenuSectionButton.css`, `DefaultCard.css`, `MenuBar.css` as **max 1 concurrent editor**. |
| **D6 — No tests** | No framework configured; nothing catches a regression automatically | Accepted. Manual 13-width diff is the control. A harness is an explicit custom-scope option, not a default. |
| **D7 — Review fatigue** | 68 items is a large PR surface | Stage boundaries are the review unit. Stage 1 and Stage 2 are behaviour/structure-only and reviewable without visual tooling. |
| **D8 — Untracked files polluting commits** | `pnpm-workspace.yaml` / `pnpm-lock.yaml` could be committed by accident | W1.2 deletes them; verify `git status` is clean of them before any commit. |

---

## Accessibility Regression Risk

The plan **improves** accessibility, but three items could regress it if done carelessly.

| Item | Current state | Risk if done wrong | Control |
|---|---|---|---|
| **W1.17** heading re-levelling | **No `<h1>` anywhere** | Introducing an `<h1>` with a different visual style than the old `<h2>` changes branding; changing `<h2>`→`<h1>` in `MenuBar` may trigger tag-based CSS | Grep all CSS for `h1`–`h4` selectors first; compensate in CSS; verify all 13 widths |
| **W1.14** focus outlines | `all:unset` at `MenuSectionButton.css:28` kills the UA ring; `Button.css:20` uses a removed value → **zero working focus indicators** | Using `outline: none` (a common but wrong "fix") would make it worse | Scope to `:focus-visible`; assert a visible outline in a keyboard-only walkthrough as a blocking Stage 1 exit criterion |
| **W2.6** error boundary | Any render error = blank page | A boundary that renders nothing, or that swallows errors silently | Must render a visible fallback **and** log to console in development |

Additional a11y items are pure additions with no regression surface: `W1.19` (`type="button"`), `W1.20` (`aria-hidden` on decorative icons), `W3.6` (landmark correctness, `<article>`), `W4.8` (`<label>` elements — must not shift layout at widths 1–3), `W4.10` (`<time>`), `W4.9` (`lang` marking — no copy change).

**Note on nested interactive content:** `<a>` wrapping `<button>` (`DefaultCard.tsx:15` → `ExperienceCardInfo.tsx:13`, `ProjectCardInfo.tsx:19,28`) and `<Link>` wrapping `<button>` (`MenuBar.tsx:61`) are invalid HTML and harm keyboard/AT users. They are **not** in the 68 items because fixing them requires a DOM-structure decision that touches visual rendering (the whole card is the link). Recorded here so a future iteration picks it up deliberately.

---

## Accepted Out-of-Scope Items

These were identified in the audit, are **known**, and are **deliberately excluded** from all 68 work items. They must not be "fixed" opportunistically.

### OOS-1 — The contact form does not send anything

`ContactForm.tsx:8-11` calls `event.preventDefault()` then `dialogRef.current?.showModal()`. Nothing is transmitted. `ContactForm.tsx:31` displays "Sua mensagem foi enviada!" regardless of validity beyond the browser's own constraint validation.

**Why out of scope:** wiring a real backend, adding a mail service or a serverless function changes functionality, deployment topology and possibly the hosting model. That is a feature, not a refactor.

**What *is* in scope:** `W1.13` (usable phone `pattern` + `inputMode`), `W1.19` (`type="button"` correctness), `W4.8` (`<label>` elements), `W4.9` (`lang` on the Portuguese strings). **The submit behaviour and the modal copy are unchanged.**

### OOS-2 — `HomePageScroll`'s progressive per-section reveal

`HomePageScroll.tsx:104-135` renders each section's content only once `visibleSections.has(key)` is true, animating it in from `y: 300`. Combined with `threshold: 0.5` (`:76`) this is the app's chosen reveal design.

**Why out of scope:** removing or altering the reveal changes the site's visual identity — the exact thing the invariants forbid. It also interacts with the accepted T5 latent risk.

**What *is* in scope:** `W1.4` (observer lifecycle — the effect currently rebuilds on every section change, which is a defect *independent* of the reveal design), `W1.5` (dead DOM-poking block), `W1.12` (Suspense around static footer text), `W2.12` (scoped observer target). **The reveal design itself is unchanged.**

### OOS-3 — `HomePage`'s scroll-restoration fake `<div>`

`HomePage.tsx:36-53`. Analysed in T2. Replaced by a mechanism in a **future** iteration, not this one.

**What *is* in scope:** `W1.6` (the `overflow` restore asymmetry, `HomePage.tsx:72-83`). The fake div and its 500ms timer are unchanged.

### OOS-4 — Nested interactive content

`<a>` wrapping `<button>` (`DefaultCard.tsx:15`), `<Link>` wrapping `<button>` (`MenuBar.tsx:61`). See the Accessibility Regression Risk section.

### OOS-5 — Duplicate images for distinct projects

`logoAPI` serves 3 different projects (`data.ts:182,194,206`); `logoDio` serves 3 recognitions (`:99,117,135`). Replacing them requires new image assets — a content/design decision.

**What *is* in scope:** `W4.6` (unique descriptive `alt` so the *text* is distinct even where the image is not) and `W4.16` (document the finding).

### OOS-6 — Actual prerendering / SSG

Deferred by decision. Analysed in `feasibility.md` § *Prerender-Readiness Rationale*, including the four APIs that make it fragile today: `IntersectionObserver` (`HomePageScroll.tsx:44`), `window.scrollTo` smooth (`HomePage.tsx:45`, `HomePageScroll.tsx:40`, `MenuBar.tsx:40`), `<dialog>.showModal()` (`ContactForm.tsx:10`), `document.createElement` + `offsetTop` (`HomePage.tsx:37-48`).

**What *is* in scope:** `W4.14` — per-route SEO metadata as data, serialisable content modules, no module-scope `window`/`document`. Explicitly **no prerender tool, no build-output change.**

### OOS-7 — A test framework

No Vitest / Testing Library / Playwright is added. Verification is manual per the 13-width matrix. Available as a custom Stage 1.5 option, which would introduce new dependencies and therefore needs explicit approval.

---

## Operational Risks

| Risk | Detail | Mitigation |
|---|---|---|
| **O1 — Blank page on bad URL** | `vercel.json:3` rewrites `/(.*)` → `/index.html`; `main.tsx:10-13` has no `*` route → `/typo` renders nothing at all | W2.6 adds a catch-all. Improves current behaviour; does not change routing of valid URLs. |
| **O2 — Blank page on any render error** | No error boundary today | W2.6. Also the single largest SEO risk: an error means a crawler gets an empty page. |
| **O3 — Permanent `overflow` mutation** | `HomePage.tsx:80-81` leaves `documentElement.style.overflow = "auto"` after unmount | W1.6. |
| **O4 — Route → route scroll leakage** | `/scroll` forces `window.scrollTo(0,0)` once on mount (`HomePageScroll.tsx:39-41`) with no cleanup; `HomePage` fights scroll restoration on mount | Out of scope to redesign; W1.6 reduces the interference. Verify `/` → `/scroll` → `/` navigation at widths 1–13. |
| **O5 — Robots meta conflicts** | W4.1 adds `meta robots`; `robots.txt` also ships (W4.4). A mismatch is a silent SEO failure | Both must agree. `robots.txt` allows all and points to the sitemap. Verify both from the deployed domain. |
| **O6 — Hardcoded domain in SEO files** | `canonical`, `og:url`, `sitemap.xml` need the production origin (`https://portfolio-renan-fellipes-projects.vercel.app`, `README.md:44`) | Derive from one place (`W4.14`'s metadata module) so there is a single constant to change on a domain migration. |
| **O7 — Security headers break assets** | See R19 | Safe subset only; verify on the deployed preview; no CSP in this plan. |
| **O8 — Social preview cache** | OG tags are cached aggressively by social platforms | After W4.1 ships, re-scrape with each platform's debugger. A stale preview is expected, not a defect. |

---

## Decision Summary

### Selected Approach

Approach A — Staged Targeted Refactor, 4 stages, 68 work items.

### Reasoning

68 items over a ~2,050-line codebase is a large surface, but every item is small, local, and constrained by five explicit invariants. Risk is concentrated in Stage 3 (CSS) and is controlled by a mandatory screenshot baseline plus the 13-width matrix. Stage 1 is behaviour-only and Stage 2 is structure-only, so the two highest-risk-looking stages are also the two most verifiable. Seven behaviours are explicitly accepted as out-of-scope so that opportunistic "fixes" do not smuggle functionality changes into a refactor.

### Expected Outcome

Zero correctness defects from the audit list. One breakpoint system with the tablet gap closed and no order-dependent rules. The full client-only SEO surface delivered and validated. Identical rendering, identical behaviour, identical branding.

### Open Questions

1. **`#646cff`** (`Button.css:15`) — deliberate brand accent or Vite template leftover? Blocks W3.5.
2. **`mainMotionValue` `200` vs `MOTION_VALUE` `100`** (`HomePage.tsx:23,28`) — which is intended? Blocks W2.10.
3. **Divider pseudo-element layout** (`MenuSectionButton.css:56-67`) — is the current stacked rendering the bug or the intent? Blocks W1.15.
4. **Lint/build baseline counts** — unknown until W1.3. Referenced by every stage's exit criteria.
5. **Real pixel dimensions of the 14 images** — required by W4.7; must be read from the files.
6. **Production origin** — confirm the canonical domain for `canonical`, `og:url` and `sitemap.xml`.

### W1.15 — Unresolved, pending human decision (recorded 2026-10-03)

**Selector / Location:** `.menu-section__items li::after` at `src/components/MenuSectionButton/MenuSectionButton.css:61`

**Context:** The pseudo-element uses `position: absolute` but no `<li>` establishes a positioning context. The nearest positioned ancestor is `.menu-section` at line 2 (`position: relative`), causing all five dividers to stack at the bottom of the list instead of separating items.

**Candidate Outcomes:**
1. **Keep current (stacked at bottom)** — add no positioning to `li`; dividers remain stacked. Preserves current pixel-identical rendering.
2. **Render between items** — add `position: relative` to `.menu-section__items li`; dividers move to separate each list item. Visibly changes rendering.

**Which preserves current branding:** Outcome 1 (keep stacked). The current rendering — however visually odd — is the shipped design. Per R14 and plan invariant 2, pixel-identical output must be preserved unless an explicit branding decision authorises a change.

**Status:** Blocked on user decision. Do not change the CSS. QA proposed Outcome 2 but it violates invariant 2. This item remains open until the user confirms which rendering is the intended brand expression.