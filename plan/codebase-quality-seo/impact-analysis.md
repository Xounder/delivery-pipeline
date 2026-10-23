# Impact Analysis — `codebase-quality-seo`

## Overview

### Objective

Map every file touched by the 68 work items so the Tech Lead can sequence tasks and parallelise safely, and confirm that the blast radius on behaviour is zero.

### Scope

26 files under `src/`, 4 root config files, 1 new `public/` directory, 1 new component directory. **No dependency upgrades. No framework change. No build-tool change.**

### Recommendation

Approach A. Stage boundaries in `index.md` are also the safe parallelism boundaries — no two tasks from different stages may run concurrently, because every stage transitions are touching the same files.

---

## Affected Areas

| Area | Impact | Notes |
|---|---|---|
| Frontend | **High** | 26 of 26 source files touched. Behaviour-neutral by construction. |
| Backend | None | No server, no API, no data store exists in this repo. |
| Database | None | — |
| Infrastructure | **Low** | `vercel.json` gains security headers (W4.11); `public/` gains 2 static files (W4.4). Build output gains route chunks (W4.12). Deploy flow unchanged. |
| Testing | **Medium** | No test framework exists. Verification is manual: 13-width screenshot diff + lint + build. W2.16 adds lint plugins. |

---

## File Overlap Map — CRITICAL FOR SEQUENCING

Files are grouped by **contention tier**. Tasks within a tier touch the same file and **must be serialised**. Tasks in different tiers can run in parallel only if they have no dependency path.

### Tier 0 — Toolchain / root config (no `src/` overlap)

| File | Work items |
|---|---|
| `package.json` | W1.1, W2.15 |
| `pnpm-workspace.yaml` (delete) | W1.2 |
| `pnpm-lock.yaml` (delete) | W1.2 |
| `eslint.config.js` | W2.16 |
| `vercel.json` | W4.11 |
| `index.html` | W4.1, W4.2, W4.3, W4.5 |
| `public/robots.txt` (new) | W4.4 |
| `public/sitemap.xml` (new) | W4.4 |

No overlap with Tier 1+. Tier 0 is fully parallel-safe with Stages 2–4.

### Tier 1 — Entry points / registry (highest contention)

| File | Work items | Why hot |
|---|---|---|
| `src/main.tsx` | W2.6, W4.12 | Routing + entry; small file, many cross-cutting concerns |
| `src/data/sections.ts` | W1.11, W2.2 | Becomes the single source of truth — every refactor reads from it |
| `src/data/data.ts` | W2.1, W2.3, W4.3, W4.6, W4.10, W4.16 | Types + 14 card entries + JSON-LD source + alt text. **6 work items — serialise all of them.** |
| `src/data/iconsData.ts` | W4.6, W4.9 | Alt text + `lang` marking |

### Tier 2 — The two pages (contention with everything visual)

| File | Work items | Count |
|---|---|---|
| `src/pages/HomePage/HomePage.tsx` | W1.6, W1.7, W1.8, W1.11, W1.18, W1.20, W2.2, W2.5, W2.13, W3.8, W3.10, W4.12, W4.13 | **13** |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | W1.4, W1.5, W1.11, W1.12, W2.2, W2.12, W3.10, W4.13, W4.17 | **9** |
| `src/pages/HomePage/HomePage.css` | W1.7, W1.8, W3.2, W3.3, W3.4, W3.11 | **6** |
| `src/pages/HomePageScroll/HomePageScroll.css` | W3.2 | 1 |

### Tier 3 — Components

| File | Work items | Count |
|---|---|---|
| `src/components/MenuBar/MenuBar.tsx` | W1.9, W2.8, W2.11, W2.14, W2.17 | 5 |
| `src/components/MenuBar/MenuBar.css` | W1.14, W3.2, W3.3, W3.4, W3.5, W3.13 | 6 |
| `src/components/DefaultCardGroup/DefaultCardGroup.tsx` | W1.10, W2.3, W2.8, W3.8 | 4 |
| `src/components/DefaultCardGroup/DefaultCardGroup.css` | W2.7, W3.2, W3.5 | 3 |
| `src/components/DefaultCard/DefaultCard.tsx` | W2.6, W3.6, W4.7, W4.10 | 4 |
| `src/components/DefaultCard/DefaultCard.css` | W1.21, W3.2, W3.3, W3.5, W3.9, W3.11, W3.12 | **7** |
| `src/components/ContactForm/ContactForm.tsx` | W1.13, W2.4, W4.8, W4.9 | 4 |
| `src/components/ContactForm/ContactForm.css` | W3.2, W3.3, W3.5, W3.11 | 4 |
| `src/components/AboutMe/AboutMe.tsx` | W2.8, W2.9, W2.14, W4.7 | 4 |
| `src/components/AboutMe/AboutMe.css` | W3.2, W3.3, W3.5, W3.11 | 4 |
| `src/components/ProjectCardInfo/ProjectCardInfo.tsx` | W2.4, W2.8 | 2 |
| `src/components/ExperienceCardInfo/ExperienceCardInfo.tsx` | W2.4, W4.10 | 2 |
| `src/components/MenuSectionButton/MenuSectionButton.tsx` | W1.19, W2.4 | 2 |
| `src/components/MenuSectionButton/MenuSectionButton.css` | W1.14, W1.15, W1.19, W3.2, W3.3, W3.4, W3.5, W3.7, W3.12 | **9** |
| `src/components/Button/Button.tsx` | W2.4 | 1 |
| `src/components/Button/Button.css` | W1.14, W3.5, W3.7 | 3 |
| `src/components/DefaultSectionButton/DefaultSectionButton.tsx` | W1.19, W3.6, W4.7 | 3 |
| `src/components/DefaultSectionButton/DefaultSectionButton.css` | W3.5 | 1 |
| `src/components/ArrowSectionButton/ArrowSectionButton.tsx` | W2.4, W3.6 | 2 |
| `src/components/ArrowSectionButton/ArrowSectionButton.css` | W3.2 | 1 |
| `src/components/InfoCard/InfoCard.tsx` | W1.17 | 1 |
| `src/components/InfoCard/InfoCard.css` | W1.16, W1.17, W3.2 | 3 |

### Tier 4 — Global styles / dead assets / new

| File | Work items |
|---|---|
| `src/index.css` | W1.7, W3.1, W3.5, W4.15 |
| `src/assets/images/me.png` | W2.14 (delete) |
| `public/robots.txt`, `public/sitemap.xml` (new) | W4.4 |
| `src/components/ErrorBoundary/` (new) | W2.6 |
| `src/data/seo/` (new) | W4.14 |

### Contention summary

| Files | Max concurrent editors |
|---|---|
| `src/data/data.ts` | **1** |
| `src/pages/HomePage/HomePage.tsx` | **1** |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | **1** |
| `src/components/MenuSectionButton/MenuSectionButton.css` | **1** |
| `src/components/DefaultCard/DefaultCard.css` | **1** |
| `src/components/MenuBar/MenuBar.css` | **1** |

### Safe parallel batches (within a stage, dependency-respecting)

- **Stage 1, batch A** (after W1.1–W1.3 green): W1.9 (`MenuBar.tsx`) ∥ W1.12 (`HomePageScroll.tsx`) ∥ W1.13 (`ContactForm.tsx`) ∥ W1.18 (`HomePage.tsx`) — 4 distinct files.
- **Stage 1, batch B**: W1.14/W1.15 (`MenuSectionButton.css`) ∥ W1.16 (`InfoCard.css`) — then W1.17 (needs both files settled).
- **Stage 1, serial chain**: W1.7 → W1.8 (same CSS file, same bug).
- **Stage 1, serial chain**: W1.11 → (Stage 2) W2.3.
- **Stage 3**: W3.5 (colours) ∥ W3.8 (inline styles) ∥ W3.10 (motion constants) — distinct concerns, though W3.5 and W3.2 both touch the same files, so **W3.5 must follow W3.2 per file**.
- **Stage 4, fully parallel**: W4.1 ∥ W4.2 ∥ W4.5 (`index.html` — serialise these three) ∥ W4.11 (`vercel.json`) ∥ W4.4 (`public/`) ∥ W4.15 (`index.css`).

---

## Expected Changes

### Files

**Modified (26 existing + 4 config):**
`index.html`, `vercel.json`, `package.json`, `eslint.config.js`,
`src/main.tsx`, `src/index.css`,
`src/pages/HomePage/{HomePage.tsx,HomePage.css}`,
`src/pages/HomePageScroll/{HomePageScroll.tsx,HomePageScroll.css}`,
`src/components/MenuBar/{MenuBar.tsx,MenuBar.css}`,
`src/components/AboutMe/{AboutMe.tsx,AboutMe.css}`,
`src/components/ContactForm/{ContactForm.tsx,ContactForm.css}`,
`src/components/DefaultCard/{DefaultCard.tsx,DefaultCard.css}`,
`src/components/DefaultCardGroup/{DefaultCardGroup.tsx,DefaultCardGroup.css}`,
`src/components/ExperienceCardInfo/{ExperienceCardInfo.tsx,ExperienceCardInfo.css}`,
`src/components/ProjectCardInfo/{ProjectCardInfo.tsx,ProjectCardInfo.css}`,
`src/components/MenuSectionButton/{MenuSectionButton.tsx,MenuSectionButton.css}`,
`src/components/ArrowSectionButton/{ArrowSectionButton.tsx,ArrowSectionButton.css}`,
`src/components/DefaultSectionButton/{DefaultSectionButton.tsx,DefaultSectionButton.css}`,
`src/components/Button/{Button.tsx,Button.css}`,
`src/components/InfoCard/{InfoCard.tsx,InfoCard.css}`,
`src/data/{data.ts,sections.ts,iconsData.ts}`

**Deleted (2):** `pnpm-workspace.yaml`, `pnpm-lock.yaml` (both untracked)

**Deleted (1 asset):** `src/assets/images/me.png`

**New (5):** `public/robots.txt`, `public/sitemap.xml`, `src/components/ErrorBoundary/ErrorBoundary.tsx`, `src/data/seo/` (route metadata module), manifest + apple-touch-icon asset

### Modules

| Module | Change |
|---|---|
| Routing / entry | Error boundary + catch-all route (W2.6); route-level lazy loading (W4.12) |
| Section registry | `sections.ts` becomes the single typed source of truth (W2.2, W2.3) |
| Data layer | Dependency inversion removed (W2.1); discriminated union (W2.3); JSON-LD source (W4.3) |
| Component set | 4 button components consolidated (W2.4); `Article` semantics (W3.6); `<time>` (W4.10) |
| Styling | 5 breakpoint tokens + layout/colour/radius tokens in `:root` (W3.1); 28 media queries migrated (W3.2); 6 order-dependent blocks removed (W3.3) |
| Animations | Shared variants + duration tokens (W3.10); `LazyMotion` evaluation (W4.13) |
| SEO | Full head block + JSON-LD + `robots.txt` + `sitemap.xml` + `<noscript>` (W4.1–W4.5) |
| Tooling | Self-dependency removed (W1.1); router types corrected (W2.15); `react` + `jsx-a11y` lint plugins added (W2.16) |

### Dependencies

**Runtime — unchanged.** No runtime package is added, removed or upgraded anywhere in the plan.

**Dev — 2 additions:**
- `eslint-plugin-react`, `eslint-plugin-jsx-a11y` (W2.16) — expect them to surface pre-existing violations; baseline separately.
- W2.15 either adds the v7 router types or removes the stale `@types/react-router-dom@^5.3.3`. No new runtime cost either way.

**Removed:** the self-referential `"portfolio": "file:"` (W1.1).

**Build output:** gains route-level chunks (W4.12); `public/` gains 2 static files (W4.4). The deploy contract on Vercel is unchanged.

---

## Breaking Changes

**None.**

Zero behaviour change, zero API change, zero routing change, zero visual change. Explicitly:

| Surface | Status |
|---|---|
| Routes `/` and `/scroll` | Unchanged. The `*` catch-all (W2.6) replaces a blank page with a fallback — additive, and `vercel.json`'s SPA rewrite is untouched. |
| Contact form | **Unchanged, including its simulated submit.** Explicitly out of scope. |
| `HomePageScroll` progressive reveal | **Unchanged.** Out of scope. |
| Rendered output | Pixel-identical. Every token introduced is a rename of an existing value. |
| Copy and content | Unchanged, except `alt` text, which the user explicitly approved for uniqueness/descriptiveness (W4.6). |
| Domain names / links | Unchanged. |
| Deploy | Unchanged. |

**Deployment-side changes (not app behaviour):** new response headers (W4.11) and 2 new static files (W4.4). These are additive and do not affect the app.

---

## Performance & Core Web Vitals Impact

### Bundle

| Item | Today | After | Note |
|---|---|---|---|
| Initial JS | All routes + all components + full `framer-motion` | Routes and section components split (W4.12) | Pattern already proven at `HomePageScroll.tsx:14-30` |
| `framer-motion` | Full `motion` entry (~110 KB gz) | `LazyMotion` + `domAnimation` subset, or unchanged (W4.13) | **Conditional.** All animations used (opacity/x/y) are in the `dom` subset. Fallback = keep current import. |
| Data graph | All 14 images referenced from the eager `data.ts` import | Split with the consuming route | Images stay separate assets; only the URL strings move out of the initial chunk |
| Dead weight | `me.png` (unused), dead `Inter` declaration, `.group-cards` duplicate block | Removed (W2.14, W4.15) | Negligible byte win; correctness win |

### Core Web Vitals

| Metric | Direction | Driver |
|---|---|---|
| **LCP** | **Improve** | `loading="eager"` + `fetchpriority="high"` on `me.webp` (W4.7); code splitting (W4.12); `<noscript>` (W4.5) |
| **CLS** | **Improve** | Intrinsic `width`/`height` on all 14 images (W4.7); `dvh` root fix (W1.7) removes the mobile layout shift that caused commits `1fed2df`/`1e7e6e3` |
| **INP** | **Improve** | 18 `:hover` blocks that animate `height`/`max-height`/`font-size` → one custom-property-driven mechanism (W3.9) removes layout thrash on every card hover |
| **Scroll smoothness** | **Improve** | Observer no longer rebuilt on every section change (W1.4) |
| **Memory** | **Improve** | Cleared timer (W1.9); no repeated observer construction (W1.4); no permanent `overflow` mutation (W1.6) |
| **Font loading** | Neutral | `Inter` is never loaded, so removing the dead declaration changes nothing (`index.css:2`) |
| **Render-blocking CSS** | Neutral | Vite already emits a single CSS bundle in prod; the 15-request cost is dev-only |

### CSS efficiency

| Metric | Today | After |
|---|---|---|
| `@media` blocks | 28 across 12 files | Same count, but 5 tokens and **0 order-dependent** blocks |
| Hardcoded brand colours | ~24 | ~0 outside `:root` |
| Duplicated layout literals | 10 | 0 |
| Duplicated `:hover` blocks | 18 | 1 mechanism |
| Tablet breakpoint | **absent** | `--bp-md: 768px` |

### Non-performance risks introduced

- **W3.2** removes 6 `(orientation: landscape)` blocks. Because those currently win by source order, a naive replacement produces a visible diff at verification-matrix widths 4, 5 and 10 (667×375, 844×390, 1180×820). This is the single most likely source of regression in the whole plan.
- **W4.12** adds network requests for the lazy chunks. On a cold, slow connection the Suspense fallback (`"Carregando..."`, `HomePageScroll.tsx:121`) is user-visible text. Verify it does not introduce a visible delay on the main route.
- **W4.13** (`LazyMotion`) can change first-paint animation if features are registered incorrectly — `initial`/`animate` would silently not run. Requires a visual check of every animation on both routes.

---

## Verification Plan

| Check | When | Blocking? |
|---|---|---|
| `npm run lint` (≤ baseline count) | Every stage exit | Yes |
| `npm run build` (`tsc -b && vite build`) | Every stage exit | Yes |
| Screenshot diff at 13 widths | Every stage exit (baseline captured before Stage 3) | Yes |
| Horizontal-overflow check (`document.scrollWidth > clientWidth`) | Widths 1, 2, 3, 4, 5 | Yes |
| Hover-state screenshot diff | Widths 1–13 (W3.9) | Yes |
| Focus-visible walkthrough, keyboard only | Stage 1 exit (W1.14) | Yes |
| Rich Results Test + Facebook/LinkedIn/Twitter preview debuggers | Stage 4 exit | Yes |
| `robots.txt` / `sitemap.xml` fetch from the deployed domain | Stage 4 exit | Yes |
| Heading outline check (exactly one `h1`) | Stage 1 exit (W1.17), Stage 4 exit | Yes |

**Verification matrix (13 widths), mandatory wherever CSS or layout is touched:**

360×640 · 390×844 · 430×932 · 667×375 · 844×390 · 768×1024 · 820×1180 · 834×1194 · 1024×1366 · 1180×820 · 1280×800 · 1440×900 · 1920×1080