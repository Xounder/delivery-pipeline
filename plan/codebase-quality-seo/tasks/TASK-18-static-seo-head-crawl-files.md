# Task — Static SEO Head Block, Favicon MIME, Crawl Files & `<noscript>`

## Task Information

### ID

TASK-18

### Title

Add the full head metadata block, fix the favicon MIME mismatch, ship `robots.txt` + `sitemap.xml`, add a `<noscript>` fallback

### Owner

senior-frontend

### Status

Source complete — external/deployed verification pending

### Stage

Stage 4 — SEO & Performance

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W4.1** | High | Add the full head block: `meta description`, `link rel=canonical`, `meta robots`, complete Open Graph, complete Twitter card, `meta theme-color`, manifest link, `apple-touch-icon` | `index.html:3-8` |
| **W4.2** | High | Fix the favicon MIME mismatch — `type="image/svg+xml"` declared for a `.webp` asset | `index.html:5` |
| **W4.4** | High | Create `public/robots.txt` (allow-all + sitemap reference) and `public/sitemap.xml` listing `/` and `/scroll` with `lastmod` | new `public/robots.txt`, new `public/sitemap.xml` |
| **W4.5** | High | Add `<noscript>` fallback content | `index.html:9-12` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **`index.html` is the shell the app mounts into — `<div id="root">` must remain the mount point.**
2. **Visual branding** — rendered output must be pixel-identical. **`<noscript>` never renders when JS is enabled, so it is invisible — verify that.** Head metadata has no visual effect.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged. **The `<noscript>` text and the `meta description` are NEW text.** `feasibility.md` constrains this: *"Values must be drawn from existing copy — no new marketing text is authored by this plan."* Reuse existing strings from the site; do not write new prose.
4. **Routing** — `/` and `/scroll` only, `vercel.json`'s SPA rewrite untouched. **`sitemap.xml` lists exactly `/` and `/scroll` — no other route.**
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content h2`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **`<div id="root">` is an invariant-5 contract — do not change its id, tag or position.**

---

## Description

Four purely additive SEO items that deliver the highest-visibility SEO win at near-zero risk. `feasibility.md` rates all four Small/Medium and notes this group is *"purely additive (head metadata + new public files)"*.

**W4.1 — the head block.** Today `index.html:3-8` has almost nothing. Add `meta description`, `link rel=canonical`, `meta robots`, complete Open Graph (`og:title/description/image/type/url/site_name/locale`), complete Twitter card (`twitter:card/title/description/image`), `meta theme-color`, a manifest link and an `apple-touch-icon`.

**W4.2 — favicon MIME mismatch.** `index.html:5` declares `type="image/svg+xml"` for a `.webp` asset, so browsers may ignore the icon entirely.

**W4.4 — crawl files.** Vercel serves `public/` directly, so `robots.txt` and `sitemap.xml` work **independently of any future prerender** (OOS-6). This is URL discovery — the one SEO capability that does not need indexable body content.

**W4.5 — `<noscript>`.** Non-JS crawlers currently receive `<div id="root"></div>` and nothing else. `feasibility.md` constrains this to *"Short summary + link to the live profile. Keep minimal."*

---

## Acceptance Criteria

- [ ] `index.html` contains `meta name="description"`, `link rel="canonical"`, `meta name="robots"`.
- [ ] Complete Open Graph: `og:title`, `og:description`, `og:image`, `og:type`, `og:url`, `og:site_name`, `og:locale`.
- [ ] Complete Twitter card: `twitter:card`, `twitter:title`, `twitter:description`, `twitter:image`.
- [ ] `meta name="theme-color"` present.
- [ ] Manifest link present. **If no manifest file exists, either create one in `public/` or record that the link is deferred — do not ship a link to a 404.**
- [ ] `apple-touch-icon` present and pointing at a real asset.
- [ ] The favicon's `type` attribute matches its actual file format (W4.2).
- [ ] Every Open Graph/Twitter value is **drawn from existing site copy** — no newly authored marketing prose.
- [ ] `public/robots.txt` exists: allow-all plus a `Sitemap:` line pointing at the production sitemap URL.
- [ ] `public/sitemap.xml` exists: exactly two `<url>` entries (`/`, `/scroll`), each with `<lastmod>`.
- [ ] **`meta robots` and `robots.txt` agree.** R-risk **O5** calls a mismatch *"a silent SEO failure"*.
- [ ] The canonical origin, `og:url` and the sitemap origin all use **one** consistent production origin (`risks.md` **O6**). Record which origin was used.
- [ ] `index.html` contains a `<noscript>` with a short summary + a link, drawn from existing copy.
- [ ] **`<noscript>` does not render when JavaScript is enabled** — screenshot diff clean.
- [ ] `<div id="root">` unchanged: same id, tag, and position in the document.
- [ ] `vercel.json` untouched by this task (W4.11 is TASK-21).
- [ ] `npm run build` passes; `robots.txt` and `sitemap.xml` appear in `dist/`.
- [ ] `npm run lint` does not regress above the TASK-01 baseline.

---

## Dependencies

### Required Tasks

- TASK-01
- Stage 1, Stage 2 and Stage 3 exit criteria

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **Stage 3 exit** — `index.md` Stage 4 entry criteria: *"All Stage 3 exit criteria met (visual baseline is stable and trusted)."*

**Parallelism:** this task is in Stage 4 batch S4-A alongside TASK-21. Their file sets are **completely disjoint** (`index.html` + `public/` vs `vercel.json` + `index.css` + `HomePageScroll.tsx`). Per `impact-analysis.md` Tier 0, *"No overlap with Tier 1+. Tier 0 is fully parallel-safe."*

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `index.html` | 4 (W4.1, W4.2, W4.3, W4.5) | `:3-8`, `:5`, `:9-12` | **TASK-19** (W4.3 — the JSON-LD `<script>` block) | **Sequential.** TASK-19 runs in S4-B and appends the JSON-LD script. **TASK-19's W4.14 metadata module should supply the canonical URL — do not hardcode the origin twice.** |
| `public/robots.txt` *(new)* | 1 | created here | — | No overlap. |
| `public/sitemap.xml` *(new)* | 1 | created here | — | No overlap. |
| `public/` manifest / apple-touch-icon *(new)* | — | created here | — | No overlap. **Do not add other files to `public/`** — `impact-analysis.md` § *Expected Changes* lists exactly which files change. |
| `<div id="root">` | — | read-only invariant | TASK-22 (W4.12 rewrites `main.tsx`, not `index.html`) | No overlap. |

**`index.html` merge ledger:** TASK-18 (W4.1, W4.2, W4.5) → TASK-19 (W4.3 JSON-LD script). Strictly sequential.

---

## Technical Context

### Relevant Components

- The HTML shell — head metadata and the mount point

### Relevant Modules

- `index.html:3-8` — head
- `index.html:5` — favicon
- `index.html:9-12` — body / mount point
- `public/robots.txt` *(new)*
- `public/sitemap.xml` *(new)*

### Relevant APIs

- Open Graph protocol (`og:*`)
- Twitter card markup (`twitter:*`)
- `robots.txt` directives and `Sitemap:`
- Sitemap protocol (`<urlset>`, `<url>`, `<lastmod>`)
- Vite static asset serving from `public/`
- Vercel static file serving from `public/`

### Relevant Types

- None (markup + static files)

---

## Implementation Guidance

### Expected Changes

1. **W4.1** — populate the head with the full block. Draw every value from existing site copy: the site title ("Renan Fellipe"), the bio from `AboutMe`, project descriptions from `data.ts`. Use the production origin from `risks.md` **O6**.
2. **W4.2** — correct the favicon `type` to match the actual asset format (`.webp` → `image/webp`, or convert/repoint the asset).
3. **W4.4** — create `public/robots.txt` with `User-agent: *`, `Allow: /`, and `Sitemap: https://<origin>/sitemap.xml`. Create `public/sitemap.xml` with two `<url>` entries and `<lastmod>`.
4. **W4.5** — add a `<noscript>` inside `<body>`, before or after `#root`, with a short summary and a link, drawn from existing copy.

### Constraints

- **No new marketing copy.** Reuse existing strings. `feasibility.md` is explicit.
- **Do not change `<div id="root">`.** Invariant 5.
- **Do not touch `vercel.json`.** W4.11 is TASK-21.
- **Do not add a prerender tool.** OOS-6.
- **Do not hardcode the production origin in more than one place.** O6: *"Derive from one place so there is a single constant to change on a domain migration."* TASK-19's W4.14 metadata module is that place — **record that this task's values are duplicated there and will be consolidated.**
- **Do not list any route other than `/` and `/scroll` in the sitemap.** Invariant 4.
- **Do not add `Content-Security-Policy`.** R19 explicitly excludes CSP from this plan.
- **Do not ship a manifest link to a nonexistent file.** If no manifest exists, create one or record the deferral.
- **Keep `<noscript>` minimal.** No styling, no scripts, no heavy markup.
- **Do not set `meta robots` to anything other than `index, follow`.** Combined with allow-all `robots.txt` (O5).

### Validation Rules

- **Validate with external tools at Stage 4 exit** (`impact-analysis.md` § *Verification Plan*): Facebook Sharing Debugger, LinkedIn Post Inspector, Twitter Card Validator. These are blocking for the stage.
- **Fetch `robots.txt` and `sitemap.xml` from the deployed domain** — not from the local build. Vercel serves `public/` directly.
- Confirm both files appear in `dist/` after `npm run build`.
- Screenshot diff at all 13 widths to confirm `<noscript>` does not render and nothing else moved.
- Verify the favicon renders (browser tab shows the icon).
- `npm run build` passes; `npm run lint` green.

---

## Edge Cases

- **`<noscript>` renders in DOM-inspection tools.** A crawler that does not execute JS sees it; a screenshot tool that does execute JS does not. If a diff tool reports a `<noscript>` change, that is expected and not a regression — verify the *rendered* state instead.
- **`meta robots` and `robots.txt` disagreeing is silent.** O5. Both allow indexing. Verify both.
- **The production origin may be wrong.** O6 lists confirming the canonical domain as an open question (`README.md:44` records `https://portfolio-renan-fellipes-projects.vercel.app`). **Confirm before writing it into three places.** A wrong canonical is worse than no canonical — it tells crawlers the real site lives elsewhere.
- **`og:image` needs an absolute URL.** A relative path is ignored by most social scrapers. Same for `og:url`.
- **`og:image` must be a real, publicly fetchable image.** Pointing at a source asset (e.g. `me.webp`) works only if that asset is served at a stable public URL. Verify it resolves from the deployed domain.
- **`twitter:image` and `og:image` should match.** Mismatched images produce inconsistent previews across platforms.
- **Favicon `type` mismatch is the actual bug (W4.2).** Some browsers ignore the icon entirely when `type` disagrees with the content. Fix the attribute — do not necessarily re-encode the asset.
- **`apple-touch-icon` has no `type` attribute** and must be a PNG (iOS ignores WebP for touch icons). If no suitable asset exists, record the deferral rather than pointing iOS at a `.webp`.
- **`lastmod` in `sitemap.xml` must be a valid W3C date.** Use `YYYY-MM-DD` or full ISO 8601. A malformed date causes the sitemap to be ignored.
- **Vite copies `public/` verbatim into `dist/`.** Verify no transformation occurs and the files are served at the root (`/robots.txt`, `/sitemap.xml`), not nested.
- **`robots.txt` must be plain text.** A BOM or HTML content type can cause crawlers to skip it.
- **A manifest link requires a real webmanifest file** with at least `name` and `icons`. Creating one is a new asset — confirm it does not change the deployed file list beyond what the plan anticipates.
- **Adding head tags can shift layout in edge cases** (e.g. a `<meta name="viewport">` change). **Do not modify the viewport meta** — verify it is unchanged.
- **`<noscript>` placement matters for crawlers.** Inside `<body>` is correct. Verify it does not precede `#root` in a way that confuses the mount.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Inspect `dist/index.html` for the complete head block.
- [ ] Confirm `dist/robots.txt` and `dist/sitemap.xml` exist.
- [ ] Deploy (or `npm run preview`) and fetch `/robots.txt` and `/sitemap.xml` → 200, correct content type, valid XML.
- [ ] Facebook Sharing Debugger → OG preview renders with image, title and description.
- [ ] LinkedIn Post Inspector and Twitter Card Validator → same.
- [ ] Browser tab shows the favicon (W4.2).
- [ ] Verify `<div id="root">` unchanged.
- [ ] Screenshot diff at all 13 widths → expect zero diff.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. The external validators require a **deployed** URL, so this task's full verification happens at Stage 4 exit, not at task completion. Note that dependency in the task's completion record.

---

## Definition of Done

- [ ] Full head block present (description, canonical, robots, OG, Twitter, theme-color, manifest, apple-touch-icon)
- [ ] Favicon MIME mismatch fixed
- [ ] `public/robots.txt` created with allow-all + sitemap reference
- [ ] `public/sitemap.xml` created with exactly `/` and `/scroll` + `lastmod`
- [ ] `meta robots` and `robots.txt` agree
- [ ] One consistent production origin recorded
- [ ] `<noscript>` added from existing copy; does not render with JS enabled
- [ ] `<div id="root">` unchanged
- [ ] `vercel.json` untouched
- [ ] No new marketing copy authored
- [ ] Build passes; `npm run lint` green
- [ ] Validation completed (external validators pending deployment)

## Implementation Evidence (2026-10-22)

- Confirmed production origin (user): `https://portfolio-renan-fellipes-projects.vercel.app`. `index.html` canonical, `og:url`, absolute OG/Twitter image URL, `public/robots.txt` sitemap reference, and both `public/sitemap.xml` locs use this origin. These static literals must be consolidated with TASK-19's planned `src/data/seo.ts` metadata source when the shared generation approach is implemented; do not introduce another independently maintained origin in TASK-19.
- `index.html`: full description/robots/OG/Twitter/theme/manifest/touch-icon block; favicon points to real `public/favicon.webp` with `image/webp` MIME. Description reproduces the existing `src/data/data.ts` AboutMe paragraph; title/site name reuse `Renan Fellipe`; no new marketing copy. `<noscript>` after unchanged `#root` reuses the existing AboutMe image alt text, heading, and LinkedIn link from `src/data/iconsData.ts`.
- Static assets: `public/og-image.webp` copied from `src/assets/images/me.webp`; `public/favicon.webp` copied from `src/assets/logo.webp`; `public/apple-touch-icon.png` converted from the same logo to an actual 180×180 PNG. `public/site.webmanifest` references the real PNG icon. `public/robots.txt` allows all; sitemap contains exactly `/` and `/scroll` with `lastmod` 2026-10-22 (date of this static shell/crawl-file change; update only when those URLs actually change).
- `npm run build` succeeded (461 modules; existing mixed dynamic/static import warnings). `dist/` inspection showed `index.html`, `robots.txt`, `sitemap.xml`, `site.webmanifest`, `favicon.webp`, `og-image.webp`, and `apple-touch-icon.png` at the root. `npm run lint` returned 6 existing errors / 0 warnings, all in untouched `ContactForm.tsx` and `MenuBar.tsx`; no regression from this task.
- NOT VERIFIED (per user's no-browser/no-server/no-deploy restriction): browser/favicon rendering, JS-enabled noscript invisibility and 13-width screenshot diff, deployed asset fetch/HTTP content types, Facebook/LinkedIn/Twitter previews, deployed robots/sitemap availability. Stage 4 exit must verify these after deployment; no browser or server was run here.

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W4.1, W4.2, W4.4, W4.5; Stage 4 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 4 rows for all four items; § *Prerender-Readiness Rationale*
- `.opencode/plan/codebase-quality-seo/risks.md` — O5, O6, O8, R19, OOS-6
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 0 overlap map; § *Expected Changes* (new files list)
- `index.html:3-8, 5, 9-12`
- `vercel.json:1-5` (untouched here)
- `README.md:44` (production origin)
