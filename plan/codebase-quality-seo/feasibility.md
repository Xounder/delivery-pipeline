# Feasibility Analysis — `codebase-quality-seo`

## Overview

### Objective

Confirm that all 68 work items in `index.md` are technically achievable in the current React 18.3 + TypeScript 5.6 + Vite 6 SPA without changing rendered output, and record the toolchain baseline situation the Tech Lead must establish first.

### Scope

Whole `src/` tree (26 files) plus `index.html`, `vercel.json`, `package.json`, `eslint.config.js`, new `public/` files. No new framework, no bundler change, no state-management library, no UI library, no CSS framework.

### Recommendation

Approach A — Staged Targeted Refactor. All 68 items are feasible with the existing dependency set. Only W2.16 (`eslint-plugin-react` + `eslint-plugin-jsx-a11y`) and W2.15 (router types) add dev dependencies; no runtime dependency is added anywhere in the plan.

---

## Approach A (Recommended)

### Description

Four independently shippable and revertible stages. Stage 1 establishes a verified green lint/build baseline and removes the 20 correctness defects. Stage 2 removes duplication and inverted dependencies. Stage 3 consolidates 28 media queries into 5 tokens and ~24 colour literals into a token set. Stage 4 delivers the full client-only SEO track plus code splitting. Each stage has explicit entry/exit criteria in `index.md`.

### Advantages

- Every item is local and small; no item requires a rewrite. The largest single item is W3.2 (28 media queries) and it is mechanical.
- Zero visual change is achievable because every refactor is a *rename* of an existing value or a *move* of an existing declaration — never a value edit. The one exception is W3.5's `#646cff`, which is explicitly gated on visual verification first.
- Stages are revertible independently: Stage 1 is behaviour-only, Stage 2 is structure-only, Stage 3 is CSS-only, Stage 4 is additive (head metadata + new public files).
- The 13-width verification matrix makes Stage 3 objectively checkable.
- Matches the observed failure mode in git history (10+ commits of CSS ping-pong, including a `vh`→`dvh` fix that was reverted) — the token system is the structural fix for that, not another patch.

### Disadvantages

- Highest total effort of the three approaches.
- Stage 3 carries the only meaningful visual-regression risk and therefore needs a captured baseline before it starts.
- `lint`/`build` are currently **unverified**, so the true size of Stage 1's first task is unknown until W1.3 runs.

### Estimated Effort

**Medium.** Approx. 3 of 68 items are Large (W3.2, W3.4, W4.3); ~55 are Small or Medium.

---

## Per-Work-Item Feasibility

### Stage 1 — Baseline & correctness

| ID | Feasible? | Effort | Depends on | Notes |
|---|---|---|---|---|
| W1.1 | Yes | S | — | Trivial deletion. High-confidence root cause of the blocked install: pnpm resolved `portfolio 0.0.0` as a dependency of itself. |
| W1.2 | Yes | S | — | Trivial deletion of two untracked files. `git status` returns to only the `.opencode` submodule. |
| W1.3 | Yes | S | W1.1, W1.2 | Blocked today — see *Toolchain Baseline* below. |
| W1.4 | Yes | S | W1.3 | Empty dep array is sufficient; `currentSection` is written by the callback and read via closure on each tick. No ref needed. |
| W1.5 | Yes | S | — | Pure deletion (11 lines). Verify no CSS depends on a `.visible` class being force-added — `MenuBar.tsx:53-54` already applies it via React state. |
| W1.6 | Yes | S | — | Capture the pre-existing inline values and restore them verbatim. Strictly safer than today. |
| W1.7 | Yes | M | W1.3 | `100dvh` with a `100vh` preceding declaration gives progressive enhancement. Verify widths 1–3 and 4–5 of the matrix (this is the reverted commit's territory — treat as the highest-risk item in Stage 1). |
| W1.8 | Yes | M | W1.7 | Must be reconciled with W1.7 — doing W1.8 before W1.7 risks fighting the same bug twice. Sequence W1.7 → W1.8. |
| W1.9 | Yes | S | — | Standard `clearTimeout` in cleanup. |
| W1.10 | Yes | S | W2.3 | Reordering the guard is trivial; removing the cast depends on W2.3's discriminated union. Land the reorder in Stage 1, the cast removal in Stage 2. |
| W1.11 | Yes | M | W2.3 | Cannot be fully solved in Stage 1 alone — a typed registry is Stage 2 work. In Stage 1, add a safe fallback for the `-1` case; make it exhaustive in Stage 2. |
| W1.12 | Yes | S | — | Unwrap; no lazy import involved. |
| W1.13 | Yes | M | — | Constraint: must accept a **superset** of today's input. Verifying the accepted set needs a browser check of the native constraint-validation UI on widths 1–3. |
| W1.14 | Yes | M | — | Replaces `-webkit-focus-ring-color` (removed non-standard value) and the `all: unset` focus-ring removal. Outline colour must come from existing branding tokens. Visual change limited to focus state only — acceptable and required for a11y. |
| W1.15 | Yes | M | — | Requires a browser check: confirm the *intended* divider positions before changing positioning context. Risk that "current rendering" is itself the bug. |
| W1.16 | Yes | S | — | Two valid resolutions; pick by intent. |
| W1.17 | Yes | M | — | Re-leveling headings risks changing rendered size if any stylesheet selector targets a specific tag. Must grep all CSS for `h1`–`h4` selectors before changing any tag and compensate in CSS. |
| W1.18 | Yes | S | — | Text-only. Note: this alt is Portuguese while `lang="en"`; W4.9 handles the language marking. |
| W1.19 | Yes | S | — | Behaviour-neutral except preventing accidental submit. |
| W1.20 | Yes | S | — | Behaviour-neutral. |
| W1.21 | Yes | S | — | **Gated**: only change if pixel-verified equivalent on the shipped logos. Otherwise document and leave. |

### Stage 2 — Organisation & DRY

| ID | Feasible? | Effort | Depends on | Notes |
|---|---|---|---|---|
| W2.1 | Yes | M | — | Pure type relocation. No runtime change. |
| W2.2 | Yes | M | — | The registry must expose name + icons + render target so all three sites derive from it. |
| W2.3 | Yes | M | W2.1 | Enables W1.10's cast removal and W1.11's total lookup. |
| W2.4 | Yes | M | W2.1 | Touches 8 files. Safe because every `<Button>` call site passes a no-op handler today — no behaviour is lost by making `onClick` optional. |
| W2.5 | Yes | S | — | Modulo arithmetic; verify wrap-around at both ends for all 5 sections. |
| W2.6 | Yes | M | — | Class component or a small library-free boundary. The catch-all route *replaces a blank page*, so it is an improvement, not a routing change. |
| W2.7 | Yes | S | — | Delete 4 duplicated lines. |
| W2.8 | Yes | S | W2.3 | Must choose a key that is unique: `link` is unique for all 14 cards, so `link` is safe. |
| W2.9 | Yes | S | — | Renders the same text with the same spacing. Verify line breaks at widths 1–3. |
| W2.10 | Yes | S | — | Value must be reconciled, not just moved — `200` vs `100` is a behaviour question. Confirm which is intended. |
| W2.11 | Yes | M | W2.2 | Deriving destination from the route removes the stringly-typed prop and the loose/strict equality mix. |
| W2.12 | Yes | M | — | Container ref replaces the global query. |
| W2.13 | Yes | S | W2.2 | |
| W2.14 | Yes | S | — | 6 independent deletions. |
| W2.15 | Yes | S | — | Check whether `react-router-dom@7` ships its own types before adding/removing packages. |
| W2.16 | Yes | S | — | **Adds dev dependencies.** Rule additions may surface *new* pre-existing violations; that count must be recorded as a second baseline, not silently fixed inside Stage 2. |
| W2.17 | Yes | S | — | |

### Stage 3 — Responsive & tokens

| ID | Feasible? | Effort | Depends on | Notes |
|---|---|---|---|---|
| W3.1 | Yes | S | — | Pure declarations in `:root`. Note: CSS custom properties cannot be used inside `@media` conditions without `@custom-media` (not natively supported). **Therefore the 5 breakpoint values must also be documented in a comment block as the canonical source, while media queries reference the literal values.** This is a known platform limitation, not a blocker. |
| W3.2 | **Large** | L | W3.1, W1.7 | 28 blocks across 12 files. Must be done with a baseline screenshot set captured first. Includes introducing the currently-absent `--bp-md: 768px` tablet-portrait handling. |
| W3.3 | Yes | M | W3.1 | Removes 6 order-dependent blocks. Highest chance of a visible diff — the 6 files currently rely on source order. |
| W3.4 | Yes | M | W3.1 | 10 literals + the `calc(100vh - 100px - 60px)` expression. Interacts with W1.7/W1.8 — sequence after both. |
| W3.5 | Yes | M | W3.1 | Byte-identical token values. `#646cff` is the single gated exception. |
| W3.6 | Yes | S | — | Semantic only; verify no CSS selector depends on `nav`/`aside` element names. |
| W3.7 | Yes | M | W3.1 | Real bug at widths 1 (360px) and below. Verify all 13 widths for horizontal overflow. |
| W3.8 | Yes | S | — | 2 sites. |
| W3.9 | Yes | M | W3.1 | 18 blocks → 1 mechanism. Must preserve the exact hover dimensions; screenshot-diff the hover state at all 13 widths. |
| W3.10 | Yes | M | — | Pure extraction of existing values. Timings unchanged. |
| W3.11 | Yes | M | — | 71 occurrences. Only simplify where provably equivalent (e.g. the unsatisfiable `clamp(12em, 90vw, 35rem)`). |
| W3.12 | Yes | S | — | Legacy-property removal; verify in WebKit, not just Chrome. |
| W3.13 | Yes | S | — | |

### Stage 4 — SEO & performance

| ID | Feasible? | Effort | Depends on | Notes |
|---|---|---|---|---|
| W4.1 | Yes | S | — | Static markup. Values must be drawn from existing copy — no new marketing text is authored by this plan. |
| W4.2 | Yes | S | — | |
| W4.3 | Yes | M | W2.3 | 8 `CreativeWork` entries derived from `ProjectsCards`. Must stay consistent with `data.ts` (single source of truth — do not duplicate the data). |
| W4.4 | Yes | S | — | Static files in `public/`, served by Vercel independently of any prerender. |
| W4.5 | Yes | S | — | Short summary + link to the live profile. Keep minimal. |
| W4.6 | Yes | M | — | 14+ alt strings rewritten to be unique and descriptive. Copy-adjacent → this is the one SEO item that changes text; approved explicitly. |
| W4.7 | Yes | M | — | Requires intrinsic dimensions for 14 images. `me.webp` and the logos need their real pixel dimensions read from the files. |
| W4.8 | Yes | M | — | Visible labels must not change layout — verify widths 1–3. |
| W4.9 | Yes | S | — | Adds `lang` attributes; no copy change. |
| W4.10 | Yes | M | W2.1 | `yearPeriod` is `"2024"` / `"2021 - 2022"` — needs a mapping to valid `datetime` values. `datetime` accepts `YYYY`, `YYYY-MM`, or a range via separate elements/text. |
| W4.11 | Yes | S | — | Verify headers do not conflict with the SPA rewrite. |
| W4.12 | Yes | M | — | Pattern already proven at `HomePageScroll.tsx:14-30`. |
| W4.13 | Yes | M | W4.12 | **Conditional**: `LazyMotion` + `domAnimation` supports only the `dom` renderer subset. Every animation used here (opacity, x, y) is in that subset, so it is compatible — but verify no future/exit-only feature is relied upon. Fallback: keep the full import. |
| W4.14 | Yes | **S** | W2.1, W2.3 | Deliberately small. Three sub-items only: metadata-as-data, serialisable content, no module-scope DOM access. **Explicitly adds no prerender tool and changes no build output.** |
| W4.15 | Yes | S | — | Pure deletion of a declaration with no effect. |
| W4.16 | Yes | S | — | Documentation only. |
| W4.17 | Yes | S | — | Note/test only — no behaviour change (progressive reveal is accepted out-of-scope). |

---

## Toolchain Baseline — Current Situation

**Status: VERIFIED — baseline recorded.**

### Baseline History

| Date | Lint | Build | Typecheck | Notes |
|---|---|---|---|---|
| Initial (W1.3) | **17 errors, 0 warnings** | **pass** (7m 48s) | **pass** | All 17 errors in `.opencode/` tooling, not `src/` |
| Post-W1.3 fix | **0 errors, 0 warnings** | **pass** | **pass** | Added `.opencode` to `eslint.config.js` ignores; ceiling drops from 17 → 0 |

```
LINT BASELINE (initial):  17 errors, 0 warnings   (command: npm run lint)
BUILD BASELINE (initial): pass                                    (command: npm run build)
TYPECHECK (initial):      pass                                    (command: npx tsc -b)

LINT BASELINE (post-W1.3, pre-plugin): 0 errors, 0 warnings (command: npm run lint)
BUILD BASELINE (post-W1.3): pass                             (command: npm run build)
TYPECHECK (post-W1.3):      pass                             (command: npx tsc -b)
```

**Cause of change:** The initial `eslint .` invocation linted the `.opencode/` directory (plugins, tools, skills) as if it were application code, producing 17 errors unrelated to the project. Adding `.opencode` to the `ignores` array in `eslint.config.js` restricts linting to `src/` and root config files only, yielding a clean 0-error baseline.

**Effect:** The pre-plugin Stage 1 ceiling of 17 was superseded by 0. Stage 2's plugin-enabled baseline is recorded separately below; its seven existing accessibility errors are not Stage 2 regressions.

### Why it was blocked (now resolved)

1. The only command runner available in this environment is pnpm-based. Its implicit `pnpm install` fails with `ERR_PNPM_IGNORED_BUILDS` — pnpm 11 blocks dependency build scripts by default and `esbuild@0.25.12` was not approved. This is an **environment policy**, not a project defect.
2. Direct `npm run …` and `node_modules/.bin/*.cmd` invocations are blocked by the sandbox command allowlist.
3. `package.json:14` `"portfolio": "file:"` made the package depend on itself. pnpm resolved it as `portfolio 0.0.0` and it was the **root cause** of the install failure. Removed in W1.1.

### Side effects already on disk

The blocked validation attempt created two **untracked** files at the repo root. Deleted by W1.2:

- `pnpm-workspace.yaml` — auto-generated by pnpm's `approve-builds` flow, content: `allowBuilds:\n  esbuild: set this to true or false`
- `pnpm-lock.yaml`

`git status --short` now reports exactly: `?? .gitmodules`, `?? .opencode/` (plus `M package.json` for the self-dependency removal). Only the `.opencode` submodule (and `.gitmodules`) remain untracked.

### Baseline recording requirement

### TASK-13 Post-plugin lint baseline (Stage 2 QA round 7 correction)

Verified against the current `eslint-plugin-jsx-a11y` installation: every configured `jsx-a11y/*` rule name in `eslint.config.js` exists in the installed plugin rule registry. No invalid configured rule names remain. The pre-plugin TASK-01 baseline was **0 errors, 0 warnings**. With the plugins enabled, the corrected lint run reports:

```
POST-PLUGIN LINT BASELINE: 7 errors, 0 warnings (command: pnpm lint)
  - jsx-a11y/control-has-associated-label: 5 errors
  - jsx-a11y/click-events-have-key-events: 1 error
  - jsx-a11y/no-noninteractive-element-interactions: 1 error
BUILD: pass (command: pnpm build; 505 modules transformed)
```

The seven accessibility errors were already present in the application before the plugins were enabled and remain unfixed. The prior post-plugin run also reported **one newly introduced** `react-refresh/only-export-components` warning from the `NotFound` declaration in `src/main.tsx:9`; it was **not** a pre-existing plugin finding. Moving `NotFound` into its own component module resolved that warning (7 errors, 0 warnings on the corrected run). Browser checks (throwing boundary, unmatched routes and screenshots) were not verified in this correction.

W1.3 must record, in this file, verbatim:

```
LINT BASELINE:  <error count> errors, <warning count> warnings   (command: npm run lint)
BUILD BASELINE: pass | fail                                    (command: npm run build)
TYPECHECK:      pass | fail                                    (command: npx tsc -b)
```

If `npm run lint` fails on **pre-existing** errors, that count becomes the ceiling for every later stage: a stage may not raise it. Fixing pre-existing lint errors is permitted but must be counted separately so the Tech Lead can distinguish pre-existing from newly-introduced failures. Note in advance that W2.16 (new lint plugins) is expected to surface additional *pre-existing* violations — those must be baselined separately at Stage 2, not counted as Stage 2 regressions.

### Known pre-existing issues that will likely surface

- `react-hooks/exhaustive-deps` warning at `HomePage.tsx:70` (function defined in the component body used in a `[]` effect)
- `react-refresh/only-export-components` warning at `MenuBar.tsx:15` (type exported alongside a component)
- Possibly a JSX prop-contract error where `HomePageScroll.tsx:123-131` passes `cardType`/`image`/`title`/`description` to a `Component` union that includes `ContactForm` (`() => JSX.Element`) — resolved implicitly by TS's JSX prop leniency. This is a typing hole, not a proven compile error; W2.3's typed registry is the real fix.

---

## Prerender-Readiness Rationale (W4.14)

**Decision: client-only best-effort SEO now; architecture for prerender later. No prerender tool is added.**

### What limits SEO today

All content — bio, 8 projects, 6 recognitions, contact — is mounted client-side via `createRoot()` at `main.tsx:8`. Non-JS crawlers and every social-preview bot receive `<div id="root"></div>`. That is the single largest SEO gap.

### What client-only best-effort delivers (~70% of the value, zero risk)

- Correct `og:`/`twitter:` tags → **social previews render** (the highest-visibility win; works today because crawlers read the static head)
- JSON-LD → **rich results / knowledge-panel eligibility**; JSON-LD is read from the static HTML and does not require body content
- `robots.txt` + `sitemap.xml` → URL discovery works
- Single `<h1>`, valid landmarks, `<article>`, `<time>`, real `alt`, correct `lang` → semantic quality signals
- `<noscript>` → non-JS crawlers get *something* meaningful
- Security headers, manifest, theme-color

### What it does not deliver

Body copy (bio, project descriptions) still requires JS execution to be indexed.

### Why prerender is deferred, not rejected

The app depends on four APIs that execute at mount and are the classic prerender failure points on WebKit:

| API | Site | Prerender hazard |
|---|---|---|
| `IntersectionObserver` | `HomePageScroll.tsx:44` | Section reveal never fires without a real viewport; a prerenderer must inject a synthetic one |
| `window.scrollTo` (smooth) | `HomePage.tsx:45`, `HomePageScroll.tsx:40`, `MenuBar.tsx:40` | Smooth scrolling is async and unresolvable in a static snapshot |
| `<dialog>.showModal()` | `ContactForm.tsx:10` | Top-layer + `::backdrop` have no meaning in static HTML |
| `document.createElement` + `offsetTop` | `HomePage.tsx:37-48` | Depends on a live layout box to compute the scroll target |

Adding a prerenderer on top of an already-unverified build would mix toolchain migration with a 68-item refactor and would put the branding invariant at risk for the remaining ~30% of SEO value.

### What W4.14 buys (small, low-risk, reversible)

1. **Per-route SEO metadata as typed data** instead of one hardcoded `index.html` block.
2. **Content modules are serialisable** — no component instances or DOM references in `data/`.
3. **No `window`/`document` access at module scope** — all DOM work lives inside effects (audit `src/main.tsx`, `src/data/*`, `MenuBar.tsx`, `HomePage.tsx`, `HomePageScroll.tsx`).

With those three in place, adding prerendering later is a config change (a build plugin + a snapshot step), not a rewrite.

### Rollback

W4.14 is purely additive data + an audit; removing it reverts to today's hardcoded head block with no other trace.

---

## Alternative Approaches

### Approach B — Minimal Bug Sweep Only

#### Description

Fix the 20 correctness defects and add meta tags, JSON-LD, `robots.txt`, `sitemap.xml`. Stop. Leave CSS, tokens and structure untouched.

#### Advantages

- Smallest effort (~24 items instead of 68), fastest to ship, lowest regression risk.
- Fixes the defects that cause visible breakage: mobile `100vh` overflow (`index.css:26-27`), the observer rebuild loop (`HomePageScroll.tsx:83`), the keyboard-focus blackout (`MenuSectionButton.css:28`).

#### Disadvantages

- Leaves 28 `@media` blocks, 5 conflicting breakpoint conditions, a 3-token colour system alongside ~24 literals, 18 duplicated `:hover` blocks, 10 duplicated layout literals.
- **Tablet (768–1023px) stays undesigned** — there is still no `768px` breakpoint anywhere.
- A new `<h1>`, `<article>`, `<time>` and unique `alt` land on top of an unchanged, ad-hoc patched CSS base.
- The `vh`→`dvh` regression will recur; the git history shows it already has.

#### Estimated Effort

**Small.**

### Approach C — Full Restructure + Design Tokens + Prerendering

#### Description

Rewrite around tokens and a typed section registry, shared layout shell, `framer-motion` → `LazyMotion`, plus static prerendering of both routes.

#### Advantages

- Highest ceiling: real indexable HTML for crawlers, smallest bundle, cleanest architecture.
- Solves the SEO root cause (CSR) rather than mitigating it.

#### Disadvantages

- Largest blast radius; changes the build/deploy contract.
- Prerender fragility on WebKit across the four APIs tabulated above.
- **Highest risk of visually changing branding**, which violates the approved invariant.

#### Estimated Effort

**Large.**

### Custom Approach

#### Description

User-defined scope, staging or ordering.

#### Notes

Any custom approach must still satisfy the five "Must Not Change" invariants in `index.md`, must keep `npm run lint` and `npm run build` green at every stage boundary, and must pass the 13-width verification matrix wherever CSS or layout is touched. Common useful variations:

- **Reorder stages** — e.g. run Stage 4 (SEO) first, since W4.1/W4.4/W4.5 are purely additive and deliver the highest-visibility SEO win at near-zero risk. Recommended if SEO visibility is the urgent driver.
- **Split Stage 3** into 3a (tokens + literals only, zero breakpoint change) and 3b (breakpoint migration + `768px` introduction), to halve the visual-regression surface.
- **Defer Stage 2 items W2.15–W2.17** (tooling/lint plugins) past Stage 4, since they add dev dependencies and can surface pre-existing violations.
- **Add a test harness.** No framework is configured. Adding Vitest + Testing Library + Playwright would make the 13-width matrix and the Stage 1 defect fixes automatically verifiable. Recommended as a Stage 1.5, but it *is* a new dependency and was not part of the approved scope.

---

## Decision Summary

### Selected Approach

Approach A — Staged Targeted Refactor, 4 stages, 68 work items.

### Reasoning

The audit found 20 correctness defects, ~30 organisation problems, 20 hacks and 5 conflicting breakpoint conditions in a ~2,050-line codebase. That size makes a full restructure unnecessary while making a bug-sweep-only insufficient — the duplication and the missing tablet breakpoint are themselves defects. Staging by risk (behaviour-only → structure-only → CSS-only → additive) makes each stage independently verifiable against the approved invariants and independently revertible, which matches the observed git history of small, risky, hard-to-review CSS patches.

### Expected Outcome

- Zero correctness defects from the audit list; zero order-dependent layout rules.
- One breakpoint system (5 tokens) with the tablet-portrait gap closed and a 13-width verification matrix enforced per task.
- Full client-only SEO surface delivered and validated, with prerendering reduced to a future config change.
- Identical rendered output and identical behaviour.

### Open Questions

1. **`#646cff` at `Button.css:15`** — is the blue hover border a deliberate brand accent or a leftover Vite template value? W3.5 is gated on this answer: keep it as a token if deliberate, replace with the branding hover colour if not.
2. **`mainMotionValue` initial `200` vs `MOTION_VALUE = 100`** (`HomePage.tsx:23,28`) — which value is intended? W2.10 must not silently change first-render animation.
3. **`.menu-section__items li::after` dividers** (`MenuSectionButton.css:56-67`) — is the current stacked-at-bottom rendering the bug, or the intent? W1.15 must confirm before changing.
4. **Lint baseline counts** — unknown until W1.3. Every later stage's exit criteria reference them.
5. **Real pixel dimensions of the 14 images** — required by W4.7; must be read from the files, not guessed.
