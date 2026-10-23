# Codebase Quality, Clean Code & SEO Plan — `codebase-quality-seo`

## Overview

### Objective

Refactor the portfolio SPA so the implementation is clean, DRY and KISS — same rendering, same behaviour, same branding — while closing the correctness defects, the duplicated/organisation debt and the ~70% missing SEO surface identified in the Phase 1 audit. Add a single source of truth for breakpoints (5 tokens) and layout metrics so ad-hoc CSS patching cannot recur.

### Scope

| In scope | Out of scope (see `risks.md` — Accepted Out-of-Scope Items) |
|---|---|
| All 26 files under `src/`, `index.html`, `vercel.json`, `package.json`, `eslint.config.js`, `tsconfig*.json`, new `public/robots.txt` + `public/sitemap.xml` | The contact form's simulated submit (`ContactForm.tsx:8-11,31`) — **document only, never change** |
| Toolchain unblock to establish a verified green `npm run lint` / `npm run build` baseline | `HomePageScroll`'s progressive per-section reveal (`HomePageScroll.tsx:104-135`) — **document only, never change** |
| Responsive consolidation to 5 breakpoints with a 13-width verification matrix | Visual design, copy, content, layout intent, routing — **all invariants** |
| Full client-only SEO track + a small *prerender-readiness* item (no prerender) | Actual prerendering/SSG — future, config-only, deliberately deferred |

### Recommendation

**Approach A — Staged Targeted Refactor**, 4 independently shippable and revertible stages. Correctness first, then tokens/breakpoints, then SEO, then polish. Every stage keeps `npm run lint` and `npm run build` green and is visually verified across all 13 widths.

---

## Invariants — "Must Not Change" (applies to EVERY work item in EVERY stage)

Any work item that cannot satisfy all five clauses below is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings (except where §SEO explicitly requires unique/descriptive text), `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays. (A `*` catch-all route is permitted only because it *replaces a blank page* — see W2.6.)
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` are part of the contract between TSX and CSS/observers and may not be renamed without a coordinated single change.

---

## Stages

### Stage 1 — Toolchain Baseline & Correctness Fixes

**Purpose:** establish a trusted green baseline, then eliminate the 20 correctness defects. No visual or structural change.

**Entry criteria**
- Approval of this plan.

**Exit criteria**
- `npm run lint` and `npm run build` (`tsc -b && vite build`) both pass. If lint has pre-existing failures, the exact count is recorded in `feasibility.md` as the **baseline** and lint must not regress above it.
- `pnpm-workspace.yaml` and `pnpm-lock.yaml` no longer present at the repo root.
- All Stage 1 work items done.
- Screenshot diff at all 13 widths is clean against the pre-refactor build.
- No `IntersectionObserver` re-creation on state change; no uncleared timers in `src/`.

**Work items**

| ID | Sev | Work item | File:line targets |
|---|---|---|---|
| **W1.1** | Critical | Remove self-referential dependency `"portfolio": "file:"` — it makes the package depend on itself and is the likely cause of the blocked install | `package.json:14` |
| **W1.2** | Critical | Delete untracked `pnpm-workspace.yaml` + `pnpm-lock.yaml` created by the blocked validation attempt; restore a clean `git status` (only the `.opencode` submodule should be untracked) | repo root |
| **W1.3** | Critical | Record the verified lint + build baseline (counts, warnings, output) into `feasibility.md`; if lint errors are pre-existing, fix only those introduced by Stage 1 and document the rest | `npm run lint`, `npm run build` |
| **W1.4** | Critical | Fix observer rebuild loop: `useEffect` dependency must not be `currentSection`; observer must be created once. Remove `currentSection` from the dependency array and keep the DOM-node identity stable | `HomePageScroll.tsx:43-83` (dep array `:83`) |
| **W1.5** | Critical | Delete the dead DOM-poking block that queries non-existent classes `.atual-section__name h2` / `.atual-section__icons`. Both return `null` today; the guard hides it. `MenuBar` already owns this state via `isVisible`. Also delete the now-pointless `document.querySelector` calls that reach outside the component | `HomePageScroll.tsx:58-68` |
| **W1.6** | Critical | Fix asymmetric `overflow` restore: restore `body`/`documentElement` to the values they actually had before, not `"visible"`/`"auto"`. Ideally capture the previous inline values and put them back | `HomePage.tsx:72-83` |
| **W1.7** | Critical | Fix root viewport units: `100vh` → `100dvh` (with `100vh` fallback) and `100vw` → `100%` to stop mobile overflow and desktop scrollbar-width overflow. This is the fix commit `1fed2df` attempted and `1e7e6e3` reverted — implement it correctly and verify | `index.css:26-27`; related `HomePage.css:16` |
| **W1.8** | High | Fix the always-overflowing height math: `height:100vh` + `padding-top:40px` at `min-width:600px` = `100vh + 40px`. Reconcile without changing the on-screen result (this is the `403ab6b`↔`986d27d` ping-pong) | `HomePage.css:16,17,68` |
| **W1.9** | High | Clear the `MenuBar` fade timer on cleanup; add `clearTimeout` in the effect's cleanup | `MenuBar.tsx:43-46` |
| **W1.10** | High | Move the guard before the dereference: `selectedData.cards.length` runs before `selectedData &&`. Reorder so the null/undefined check precedes any property access, and remove the unsafe `as keyof typeof` cast in favour of a typed lookup | `DefaultCardGroup.tsx:33,35,39` |
| **W1.11** | High | Make the section→icon lookup total: `sections.indexOf(currentSection)` can be `-1` → `iconsData[-1]` → `undefined` → `icons.map` throws. Introduce a typed section registry so the lookup cannot be `-1` (pairs with W3.4) | `HomePage.tsx:101`, `HomePageScroll.tsx:96`, `MenuBar.tsx:26`, `sections.ts:1` |
| **W1.12** | High | Remove the `Suspense` wrapper around static footer text — pointless and it delays indexable content | `HomePageScroll.tsx:145-148` |
| **W1.13** | High | Replace the unsatisfiable phone `pattern` with a usable rule and add `inputMode="tel"`. Must keep the *same accepted input space* or a superset — never narrow what users may type | `ContactForm.tsx:23-25` |
| **W1.14** | High | Restore working keyboard focus indicators: `all: unset` at `MenuSectionButton.css:28` strips the UA focus ring with no replacement; `outline: 4px auto -webkit-focus-ring-color` at `Button.css:20` uses a removed non-standard value. Replace with explicit `:focus-visible` outlines that match existing branding colours | `MenuSectionButton.css:27-44`, `Button.css:18-21` |
| **W1.15** | High | Fix the divider pseudo-element: `.menu-section__items li::after` is `position:absolute` but no `<li>` is positioned → resolves against `.menu-section__items` and all dividers stack at the bottom. Establish the intended positioning context without changing the rendered result | `MenuSectionButton.css:56-67` |
| **W1.16** | Medium | Remove dead CSS targeting a non-existent element: `.info-card h3` — the component renders `<h2>`, so `text-align:center` never applies. Decide: retarget to the real element (reproduces intent) or delete (documents intent) | `InfoCard.css:17-19` vs `InfoCard.tsx:6` |
| **W1.17** | Medium | Fix the missing `<h1>`. Add exactly one `<h1>` and re-level existing headings so the document outline is `h1 → h2 → h3 → h4`. The visible site title "Renan Fellipe" (`MenuBar.tsx:53`) is the natural `h1`. Must not change the *rendered text or size* of any heading | `MenuBar.tsx:53`, `AboutMe.tsx:20`, `InfoCard.tsx:6`, `ContactForm.tsx:19`, `DefaultCard.tsx:22`, `HomePageScroll.tsx:146` |
| **W1.18** | Medium | Correct the wrong `alt` on the menu button: it says "Ícone de um fone de ouvido" (headphone) but the asset is `menu-hamburguer.svg` | `HomePage.tsx:156` |
| **W1.19** | Medium | Add `type="button"` to every non-submit `<button>` so no button can accidentally submit a form (`DefaultSectionButton`, `MenuSectionButton` items, `MenuBar` change-view) | `DefaultSectionButton.tsx:11`, `MenuSectionButton.tsx:33`, `MenuBar.tsx:61` |
| **W1.20** | Medium | Add `aria-hidden="true"` + `alt=""` to purely decorative arrow/menu icons so screen readers do not announce them; keep meaningful alts. `ArrowSectionButton`'s `<nav>` and `HomePage`'s `<aside>` misuse is handled in W3.6 | `HomePage.tsx:141,163`, `HomePageScroll.tsx:146-147` |
| **W1.21** | Low | Fix `object-fit: fill` distorting non-square project logos. Use `object-fit: contain` only if the current rendering is visually verified equivalent for the shipped logos — otherwise document and leave | `DefaultCard.css:143` |

---

### Stage 2 — Organisation, DRY & KISS

**Purpose:** remove duplication and inverted dependencies. Still zero visual change.

**Entry criteria**
- All Stage 1 exit criteria met.

**Exit criteria**
- `sections` array has exactly **one** source of truth; the inline duplicates are gone.
- `data.ts` no longer imports types from component files.
- `cardType` is a typed union, not `string`.
- The `.group-cards` duplicate rule block and all duplicate index keys are gone.
- All 15 widths still pixel-identical.

**Work items**

| ID | Sev | Work item | File:line targets |
|---|---|---|---|
| **W2.1** | High | Break the inverted dependency: `data.ts` imports `AboutMeProps`/`ExperienceCardInfoProps`/`ProjectCardInfoProps`/`DefaultCardProps` from components. Move the data-facing shapes into a types module (or define them in `data/`) and have components import *from* it | `data.ts:1-4` |
| **W2.2** | High | Unify the 3 duplicated section lists into one registry. `sections.ts`, `HomePage.tsx`'s `componentsMap`, and `HomePageScroll`'s inline array are 3 sources of truth for the same 5 strings | `sections.ts:1`, `HomePage.tsx:55-67`, `HomePageScroll.tsx:104-110` |
| **W2.3** | High | Collapse `ExperienceCardsProps` + `ProjectCardsProps` into a discriminated union (e.g. `kind: 'project' \| 'experience'`) so `DefaultCardGroup`'s `cardDataMap` needs no cast and no unsafe index access | `data.ts:32-35,171-174`, `DefaultCardGroup.tsx:11-33` |
| **W2.4** | High | Collapse the 4 button components. `ArrowSectionButton` is an 11-line wrapper that only adds a `<nav>`; `Button` forces every caller to pass a no-op `onClick`. Make `onClick` optional and reduce to the minimum set of genuinely distinct components | `Button.tsx:3-15`, `ArrowSectionButton.tsx:5-11`, `DefaultSectionButton.tsx:9-15`, `MenuSectionButton.tsx:13-51` + call sites `ContactForm.tsx:27,32`, `ExperienceCardInfo.tsx:13-17`, `ProjectCardInfo.tsx:19-23,28-32` |
| **W2.5** | High | Extract the duplicated arrow-navigation arithmetic into one helper. Two near-identical blocks differ only by `±1` and both hand-roll wrap-around instead of modulo | `HomePage.tsx:142-149`, `HomePage.tsx:165-172` |
| **W2.6** | High | Add an error boundary and a `*` catch-all route. Today `vercel.json`'s rewrite means `/typo` matches no route → permanently blank page, and any render error is a blank page | `main.tsx:8-15`, new `src/components/ErrorBoundary/` |
| **W2.7** | Medium | Remove the duplicate `.group-cards` rule block — `scrollbar-width`/`scrollbar-color` are declared verbatim twice | `DefaultCardGroup.css:1-13` vs `:15-18` |
| **W2.8** | Medium | Replace index keys with stable domain keys (`link` or `title` for cards, icon identity for menu icons, text for description paragraphs). Guard against duplicate keys: `logoDio` appears 3× and `logoAPI` 3×, so use `link`+`title` | `DefaultCardGroup.tsx:42`, `ProjectCardInfo.tsx:18`, `MenuBar.tsx:35`, `AboutMe.tsx:23` |
| **W2.9** | Medium | Replace the array-of-one-plus-`<br><br>` hack with a proper paragraph render. `description: string[]` holds exactly one string and each is followed by two `<br>`s | `AboutMe.tsx:1,21-29`, `data.ts:27-29` |
| **W2.10** | Medium | Hoist `MOTION_VALUE` out of the component body to a module constant, and reconcile the inconsistent initial value (`200` vs the constant's `100`) | `HomePage.tsx:23,28` |
| **W2.11** | Medium | Fix stringly-typed `atualPage` and its mixed equality (`=== '/'` at one site, `== '/'` at another). Derive the destination from the route instead of passing a string and re-comparing it in two places; eliminate the no-op `menuSetCurrentSection={() => {}}` by making the prop optional | `MenuBar.tsx:8-13,40,61`, `HomePage.tsx:102`, `HomePageScroll.tsx:97-98` |
| **W2.12** | Medium | Replace the globally-scoped `document.querySelectorAll("section")` with a scoped container ref so a foreign `<section>` can never be observed or silently mislabelled as "About Me" | `HomePageScroll.tsx:79`, `:48` |
| **W2.13** | Medium | Stop rebuilding `componentsMap` on every render — hoist the static element map or render directly from the section registry | `HomePage.tsx:55-67,126` |
| **W2.14** | Low | Delete dead code and dead assets: `me.png` (never imported), `React` default import under the automatic JSX runtime, the unused `React.Fragment`, the stray `</Link>;` semicolon, the non-exported interfaces in `data.ts`, the `return`-of-`void` in `navigateToStart` | `src/assets/images/me.png`, `AboutMe.tsx:1,23`, `MenuBar.tsx:40,61`, `data.ts:32,171` |
| **W2.15** | Low | Install the correct router types: `@types/react-router-dom@^5.3.3` is v5 types against `react-router-dom@^7`. Either migrate to the v7 types package or remove the stale `@types` and rely on the bundled types | `package.json:17,23` |
| **W2.16** | Low | Add `eslint-plugin-react` and `eslint-plugin-jsx-a11y` to `eslint.config.js` so `react/jsx-key` and the a11y rules are enforced. This is what made the index-key and a11y defects invisible | `eslint.config.js:16-26` |
| **W2.17** | Low | Fix the `react-refresh/only-export-components` warning by moving the `Icon` type out of the component module | `MenuBar.tsx:15-19` |

---

### Stage 3 — Responsive & Design-System Consolidation

**Purpose:** one breakpoint system, one set of layout tokens, no cascade-order-dependent layout. **Highest visual-regression risk stage.**

**Entry criteria**
- All Stage 2 exit criteria met.
- A baseline screenshot set at all 13 widths has been captured from the pre-Stage-3 build and is available for diffing.

**Exit criteria**
- Exactly 5 breakpoint tokens exist, declared once, and are used by every media query in the app.
- **Zero** `max-width … orientation: landscape` blocks remain.
- Zero hardcoded header/footer height literals; all derive from tokens.
- Zero hardcoded brand colour literals outside `:root`; all derive from tokens.
- Screenshot diff at all 13 widths is pixel-identical to the Stage 2 baseline.
- **`min-width: 768px` (tablet portrait) is now defined and exercised** — it currently does not exist anywhere in the codebase.

**Breakpoint tokens (approved)**

| Token | Value | Replaces / covers |
|---|---|---|
| `--bp-sm` | `600px` | existing `min-width: 600px` |
| `--bp-md` | `768px` | **new — the missing tablet-portrait case (768/820/834)** |
| `--bp-lg` | `1024px` | existing `min-width: 1024px` |
| `--bp-xl` | `1440px` | existing `min-width: 1440px` |
| `--bp-2xl` | `1920px` | new — add only if a real defect is found at ≥1920 |

Mobile-first authoring. The 6 order-dependent `max-width: 1000px and (orientation: landscape)` blocks are replaced by `--bp-sm` width rules plus `max-height` queries, which removes the cascade-order conflict entirely. (Phone-landscape shortest common width ≈ 667px, so `--bp-sm` already covers those devices.)

**Verification matrix (13 widths — mandatory acceptance criterion for W3.1–W3.7)**

| # | Viewport | Target |
|---|---|---|
| 1 | 360×640 | small Android |
| 2 | 390×844 | iPhone 14 |
| 3 | 430×932 | iPhone Pro Max |
| 4 | 667×375 | phone landscape (short) |
| 5 | 844×390 | phone landscape (tall) |
| 6 | 768×1024 | tablet portrait — **currently undesigned** |
| 7 | 820×1180 | iPad Air portrait — **currently undesigned** |
| 8 | 834×1194 | iPad Pro portrait — **currently undesigned** |
| 9 | 1024×1366 | iPad Pro portrait (large) |
| 10 | 1180×820 | iPad Air landscape |
| 11 | 1280×800 | small desktop |
| 12 | 1440×900 | desktop |
| 13 | 1920×1080 | large desktop |

**Work items**

| ID | Sev | Work item | File:line targets |
|---|---|---|---|
| **W3.1** | High | Declare the 5 breakpoint tokens and all layout/colour/radius tokens once in `:root`: header height, footer height, card width, breakpoint widths, the 4 purples, surface/overlay radii, the neutral text colour | `index.css:1-15` |
| **W3.2** | High | Migrate all 28 `@media` blocks to the token system. Introduce `--bp-md: 768px` handling for tablet portrait. Replace the 2 overlapping `max-width: 600px` / `min-width: 600px` pairs so they no longer both match at exactly 600px | 12 CSS files; the 28 blocks listed in the audit inventory |
| **W3.3** | High | Delete the 6 order-dependent `max-width:1000px and (orientation: landscape)` blocks and re-express their intent with `max-height` queries + `--bp-sm`. These currently win over `min-width:600px` purely by source order | `AboutMe.css:76`, `ContactForm.css:111`, `DefaultCard.css:188`, `HomePage.css:76`, `MenuBar.css:117`, `MenuSectionButton.css:79` |
| **W3.4** | High | Tokenise the 10 duplicated layout literals. Header `90px`/`100px`, footer `60px`, and the `calc(100vh - 100px - 60px)` expression are hardcoded in 10 places across 3 files plus the `top: 75px/85px/45px` button offsets | `HomePage.css:7,43,64,72,78`, `MenuBar.css:10,72,100,113,119,133`, `MenuSectionButton.css:14` |
| **W3.5** | Medium | Tokenise the ~24 hardcoded colour literals, keeping values byte-identical. Replace the Vite-template leftover `#646cff` at `Button.css:15` with the correct branding hover colour **only if** the current blue is confirmed as a leftover (this is the one tokenisation that could change a pixel — verify first) | `index.css:6`, `Button.css:15`, `DefaultCard.css:13,18,89`, `AboutMe.css:48`, `MenuBar.css:79`, `MenuSectionButton.css:39,47,53,65`, `DefaultSectionButton.css:23`, `InfoCard.css:13`, `DefaultCardGroup.css:11,17,29` |
| **W3.6** | Medium | Correct semantic landmark misuse: `<nav>` around a single arrow button, and `<aside>` for the primary navigation arrows. Every card becomes an `<article>`. No visual effect | `ArrowSectionButton.tsx:7`, `HomePage.tsx:138,161`, `DefaultCard.tsx:15-31` |
| **W3.7** | Medium | Fix small-screen horizontal overflow: `.menu-section__items` is `width: 28rem; max-width: 350px`, `position:absolute; left:50%; transform:translateX(-50%)` → overflows both edges below ~390px and causes horizontal page scroll. Same class of issue at `.menu-section__items li button` (`21.85rem`) and `.button` (`padding: 9.6px 96px`) | `MenuSectionButton.css:13-22,41`, `Button.css:2` |
| **W3.8** | Medium | Remove inline styles that override the stylesheet and duplicate CSS rules | `HomePage.tsx:136`, `DefaultCardGroup.tsx:38` |
| **W3.9** | Medium | Consolidate the 18 duplicated `:hover` blocks that animate `height`/`max-height`/`font-size`. Drive the expansion from a single CSS custom property (or `transform`) so one declaration controls it. Rendering must be identical — this is a performance improvement (no layout thrash), not a visual one | `DefaultCard.css:17-21,74-76,94-97,106-108,116-118,129-131,146-149,164-166,174-177,183-185,197-199,205-207,217-219` |
| **W3.10** | Medium | Replace the 6 hardcoded framer-motion `transition` objects with shared variants + a duration token, and reduce magic animation values (`200vh` fake div, `500ms`, `100ms`, `200ms`) to named constants. Timings must not change | `HomePage.tsx:97,124,135,38,52,77`, `HomePageScroll.tsx:92,117,143`, `MenuBar.tsx:45` |
| **W3.11** | Low | Remove the `vh`-unit sprawl: 71 `clamp()`/`vh` occurrences across 4 files, several of which are unsatisfiable (e.g. `width: clamp(12em, 90vw, 35rem)` inside a parent capped at `20rem`/`90%` — the `90vw` term can never win). Simplify only where provably equivalent | `AboutMe.css:19-20`, `DefaultCard.css` (31 occurrences), `ContactForm.css` (22), `HomePage.css:16,72,100` |
| **W3.12** | Low | Remove legacy duplicated CSS properties kept alongside modern ones (`display: -webkit-box` + `display: box`, `box-orient`, `-webkit-line-clamp` + `line-clamp` pairs) and the `content: ' '` hack | `DefaultCard.css:79-84,95-96,170-176`, `MenuSectionButton.css:66` |
| **W3.13** | Low | Replace the "magic-number centering" hacks with robust positioning | `MenuBar.css:73-74` (`right: calc(50% - 80px)` + `width:160px`) |

---

### Stage 4 — SEO & Performance

**Purpose:** close the ~70% missing SEO surface, plus the Core Web Vitals work.

**Entry criteria**
- All Stage 3 exit criteria met (visual baseline is stable and trusted).

**Exit criteria**
- Every item in the approved SEO checklist is present and validated (Rich Results Test + social-preview debuggers).
- `robots.txt` and `sitemap.xml` are served from `public/` at the real domain.
- Exactly one `<h1>`; landmarks valid; 14 images have `alt` + intrinsic `width`/`height` + `loading`/`decoding`.
- Per-route SEO metadata exists as **data**, and no module performs `window`/`document` access during initialisation.
- Build output confirms route-level code splitting.

**Work items**

| ID | Sev | Work item | File:line targets |
|---|---|---|---|
| **W4.1** | High | Add the full head block: `meta description`, `link rel=canonical`, `meta robots`, complete Open Graph (`og:title/description/image/type/url/site_name/locale`), complete Twitter card (`twitter:card/title/description/image`), `meta theme-color`, manifest link, `apple-touch-icon` | `index.html:3-8` |
| **W4.2** | High | Fix the favicon MIME mismatch — `type="image/svg+xml"` declared for a `.webp` asset, so browsers may ignore it | `index.html:5` |
| **W4.3** | High | Add JSON-LD: `Person` (name, description, `sameAs` for the 4 socials already in `iconsData.ts:51-54`, `alumniOf`), `WebSite`, and one `CreativeWork` per project derived from `ProjectsCards` (8 entries, `data.ts:177-277`) | `index.html`, `data.ts:177-277`, `iconsData.ts:51-54` |
| **W4.4** | High | Create `public/robots.txt` (allow-all + sitemap reference) and `public/sitemap.xml` listing `/` and `/scroll` with `lastmod`. Vercel serves `public/` directly, so these work independently of any future prerender | new `public/robots.txt`, new `public/sitemap.xml` |
| **W4.5** | High | Add `<noscript>` fallback content so non-JS crawlers and users see something meaningful instead of an empty `<div id="root">` | `index.html:9-12` |
| **W4.6** | High | Unique, descriptive `alt` text. Today `"Logo API"` appears 3× and `"Logo of Dio"` 3×; `"Buletin Blog - Home Page"` / `"Angular Store - Home Page"` describe screenshots rather than the project. Every alt must describe its own image | `data.ts:99,117,135,182,194,206,221,245,268`, `iconsData.ts:27-55` |
| **W4.7** | High | Image performance + CLS: add `loading="lazy"`, `decoding="async"` and intrinsic `width`/`height` to all 14 `<img>` elements. The LCP image (`me.webp`) must be prioritised (`loading="eager"` + `fetchpriority="high"`) | `AboutMe.tsx:17`, `DefaultCard.tsx:18`, `MenuBar.tsx:29,32`, `DefaultSectionButton.tsx:12` |
| **W4.8** | High | Add proper `<label>` elements for all four form inputs (today `placeholder` is the only label — WCAG 1.3.1/3.3.2 failure) and an accessible name on the `<dialog>` | `ContactForm.tsx:21-26,30` |
| **W4.9** | High | Correct `lang` semantics. `<html lang="en">` while alt text, the modal copy and the phone `title` are Portuguese. Mark the Portuguese strings with `lang="pt-BR"` rather than changing the copy | `index.html:2`, `iconsData.ts:27-55`, `ContactForm.tsx:24,31,32` |
| **W4.10** | High | Wrap the 14 project/experience/recognition cards in `<article>` and convert `yearPeriod` strings (`"2024"`, `"2021 - 2022"`) into `<time datetime>` | `DefaultCard.tsx:15-31`, `ExperienceCardInfo.tsx:11-18`, `data.ts:53,75,94,111,129,147,166` |
| **W4.11** | Medium | Add security headers to `vercel.json`: `X-Content-Type-Options`, `Referrer-Policy`, `X-Frame-Options`, `Strict-Transport-Security`, `Permissions-Policy`. Must not affect the SPA rewrite | `vercel.json:1-5` |
| **W4.12** | Medium | Introduce route-level code splitting. `HomePageScroll` already lazy-loads 3 components — extend the pattern: lazy-load both pages in `main.tsx` and lazy-load the section components used by `HomePage`, so `framer-motion` and all card data leave the initial chunk | `main.tsx:5-6`, `HomePage.tsx:4-19`, pattern already at `HomePageScroll.tsx:14-30` |
| **W4.13** | Medium | Reduce `framer-motion` weight: the full `motion` entry is imported today. Evaluate `LazyMotion` + `domAnimation` features (or the `motion` package) for the same visual result | `HomePage.tsx:19`, `HomePageScroll.tsx:12` |
| **W4.14** | Medium | **Prerender-readiness (no prerender now).** Three small, low-risk sub-items so a future prerender becomes a config change rather than a rewrite: (a) move per-route SEO metadata into a typed data module instead of hardcoding it in one `index.html`; (b) ensure content/data modules are serialisable — no component instances or DOM references in the data layer; (c) audit `src/` for any `window`/`document` access at module scope and move it into effects. Explicitly **not** adding a prerender tool, and not changing build output | new `src/data/seo/`; audit of all `src/data/*` and module-init in `src/main.tsx` |
| **W4.15** | Low | Remove the dead `Inter` font declaration — it is referenced but never loaded (no `@font-face`, no link, no package), so the app silently falls back to `system-ui` | `index.css:2` |
| **W4.16** | Low | Document the duplicate-image/content findings (`logoAPI` used for 3 different projects, `logoDio` for 3 recognitions) so the SEO impact is recorded. **Do not** change the images — that is a content decision | `data.ts:182,194,206,99,117,135` |
| **W4.17** | Low | Evaluate `threshold: 0.5` for tall sections. Flagged as a latent content-visibility risk, but the progressive reveal is **accepted out-of-scope** — only add a regression test/note, change nothing | `HomePageScroll.tsx:76` |

---

## Prioritized Work-Item Index

| Severity | Count | IDs |
|---|---|---|
| **Critical** | 7 | W1.1 – W1.7 |
| **High** | 24 | W1.8 – W1.15, W2.1 – W2.6, W3.1 – W3.4, W4.1 – W4.6 |
| **Medium** | 25 | W1.16 – W1.20, W2.7 – W2.13, W3.5 – W3.10, W4.7 – W4.13 |
| **Low** | 12 | W1.21, W2.14 – W2.17, W3.11 – W3.13, W4.14 – W4.17 |
| **Total** | **68** | |

### Stage distribution

| Stage | Critical | High | Medium | Low | Total |
|---|---|---|---|---|---|
| 1 — Baseline & correctness | 7 | 8 | 5 | 1 | 21 |
| 2 — Organisation & DRY | 0 | 6 | 7 | 4 | 17 |
| 3 — Responsive & tokens | 0 | 4 | 6 | 3 | 13 |
| 4 — SEO & performance | 0 | 6 | 7 | 4 | 17 |
| **Total** | **7** | **24** | **25** | **12** | **68** |