# Task — Security Headers, Dead Font Declaration Removal & `content-visibility` Threshold Documentation

## Task Information

### ID

TASK-21

### Title

Add the four safe security headers to `vercel.json`, remove the dead Inter font-face declaration, and document the `content-visibility` threshold decision

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 4 — SEO & Performance

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W4.11** | Medium | Add the four **safe** security headers to `vercel.json`: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`. **The four-way `headers` merge exists specifically to preserve the existing content-type header** | `vercel.json:1-5` |
| **W4.15** | Medium | Remove the dead `@font-face` declaration for Inter in `index.css` — unused across all 21 font declarations | `src/index.css:2` |
| **W4.17** | Low | Document the `content-visibility` threshold decision (600px) as a **documentation note only**. **R7: current thresholds are well-tested and low-priority; no threshold change** | `src/pages/HomePageScroll/HomePageScroll.tsx:76` (comment) |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **W4.11 adds HTTP response headers only — no application logic. W4.17 changes a comment, not a threshold.**
2. **Visual branding** — rendered output must be pixel-identical. **W4.15 removes a font declaration — prove it is genuinely unused first; if any text falls back to Inter, this is a visual regression.**
3. **Copy** — unchanged.
4. **Routing** — unchanged — **and W4.11 must not break `vercel.json`'s SPA rewrite.** The rewrite and the `headers` array coexist in the same file.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **W4.17's comment lands in `HomePageScroll.tsx` — no selector or attribute may change.**

**Hard exclusion:** `Content-Security-Policy` is **out of scope** (R19). Framer-motion uses inline styles and Vite injects inline scripts; a CSP would likely break the app. Do not add one.

---

## Description

Three small items: two safe edits and one documentation note.

**W4.11 — security headers.** `vercel.json:1-5` currently has a single header (`X-Content-Type-Options`). Add `X-Frame-Options`, `Referrer-Policy` and `Permissions-Policy`. `impact-analysis.md` notes *"The four-way `headers` merge exists specifically to preserve the existing content-type header"* — meaning the merge must keep the existing entry intact while adding the three new ones.

**W4.15 — dead font.** `index.css:2` declares `@font-face` for Inter. `index.css:34-54` declares 21 `@font-face` blocks; **Inter is not among the fonts actually used** across the site.

**W4.17 — documentation only.** `HomePageScroll.tsx:76` uses `content-visibility: auto` with a `contain-intrinsic-size` threshold. R7: *"Current thresholds are well-tested and low-priority"* — **document the decision, do not change the value.**

---

## Acceptance Criteria

- [ ] `vercel.json` declares all four headers: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`.
- [ ] **The pre-existing `X-Content-Type-Options` entry is preserved unchanged** — not replaced or duplicated.
- [ ] **`vercel.json`'s SPA rewrite is untouched.** `/scroll` still resolves after deployment.
- [ ] `vercel.json` remains valid JSON (a trailing comma or comment breaks the deploy).
- [ ] **No `Content-Security-Policy` is added** (R19).
- [ ] The Inter `@font-face` declaration at `index.css:2` is removed.
- [ ] No `@font-face` block other than Inter is touched — the other 20 stay.
- [ ] **The font actually used for body text is unchanged.** Confirm by computed-style inspection, not by reading the file.
- [ ] The remaining `@font-face` `src` URLs are untouched.
- [ ] `HomePageScroll.tsx:76` carries a comment documenting the 600px `contain-intrinsic-size` threshold and R7's decision not to change it.
- [ ] **`contain-intrinsic-size` value is byte-identical** — no threshold change.
- [ ] `threshold: 0.5` on the IntersectionObserver is unchanged (this task documents it; it does not modify it).
- [ ] Screenshot diff clean at all 13 widths, both routes.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-13
- TASK-14
- TASK-16
- TASK-19

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-14** — the **hard gate**. R6.
- **TASK-16** — `src/index.css:2` is inside the same file TASK-16 owns for W3.5's colour tokens. `:2` and `:6` are adjacent; **sequential to keep the diff clean.**
- **TASK-19** — `src/index.css:49` (W4.9's `:lang()`) is in the same file. **Sequential.**
- **TASK-13** — W2.17 added lint tooling including `vite.config.ts` (a `headers` merge exists precisely to preserve the existing content-type header). Ensure the tooling does not reformat or flag `vercel.json`.

**Parallelism:** S4-A alongside TASK-18. `vercel.json` is touched only here; `index.css` was last owned by TASK-16 (Stage 3, already complete); `HomePageScroll.tsx` is next owned by TASK-22 (S4-C). **Per `impact-analysis.md` Tier 0, this task is parallel-safe with TASK-18 — their file sets are disjoint.**

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `vercel.json` | 1 | `:1-5` (all four headers) | — | **No overlap.** But it is a **config file with a hard routing contract** (invariant 4). Treat as a whole-file review: validate JSON and confirm the rewrite survived. |
| `src/index.css` | 4 | `:2` (W4.15 dead font) | TASK-02, **TASK-16** (W3.5 `:6`), **TASK-19** (`:49` W4.9) | **Sequential.** Ledger: TASK-02 → **TASK-16** → **TASK-19** → **TASK-21**. `:2` and `:6` are adjacent — a careless merge can delete the colour tokens or restore the dead font. |
| `src/pages/HomePageScroll/HomePageScroll.tsx` | **9** | `:76` (W4.17 comment) | TASK-03, TASK-07, TASK-08, TASK-10, **TASK-17** (`:92,117,143`), **TASK-22** (W4.13 `:12`) | **Sequential.** Ledger: … → **TASK-17** → **TASK-21** → **TASK-22**. **`:76` vs TASK-22's `:12` — isolated line ranges, but same file, so S4-A must complete before S4-C starts.** |
| `src/pages/HomePage/HomePageScroll.tsx` `:76` threshold value | — | read-only (must not change) | — | No overlap. |

---

## Technical Context

### Relevant Components

- `HomePageScroll` — `content-visibility` / `contain-intrinsic-size`, IntersectionObserver `threshold: 0.5`

### Relevant Modules

- `vercel.json:1-5`
- `src/index.css:2`, `src/index.css:34-54`
- `src/pages/HomePageScroll/HomePageScroll.tsx:76`

### Relevant APIs

- Vercel `headers` configuration (array of `source` + `headers[]`)
- HTTP response headers: `X-Content-Type-Options`, `X-Frame-Options`, `Referrer-Policy`, `Permissions-Policy`
- CSS `@font-face`
- CSS `content-visibility` / `contain-intrinsic-size`
- IntersectionObserver `threshold`

### Relevant Types

- The `vercel.json` `headers` schema (each entry: `source`, `headers: [{ key, value }]`)
- `Headers` value strings for `Permissions-Policy` (a comma-separated allow-list)

---

## Implementation Guidance

### Expected Changes

1. **W4.11** — in `vercel.json`, add three header entries alongside the existing `X-Content-Type-Options`: `X-Frame-Options: DENY` (or `SAMEORIGIN`), `Referrer-Policy: strict-origin-when-cross-origin`, `Permissions-Policy` with a minimal allow-list. **Preserve the existing entry byte-for-byte. Validate the JSON.**
2. **W4.15** — remove the Inter `@font-face` block at `index.css:2`. Leave the other 20 declarations untouched.
3. **W4.17** — add a comment at `HomePageScroll.tsx:76` recording: the current `contain-intrinsic-size` threshold, R7's decision that it is well-tested and low-priority, and that no change was made.

### Constraints

- **Never add `Content-Security-Policy`.** R19 explicitly excludes it. Framer-motion inline styles and Vite's inline scripts would likely be blocked.
- **Do not touch the SPA rewrite in `vercel.json`.** Invariant 4 — this is a breaking-deploy risk if missed.
- **Do not remove or reorder the existing `X-Content-Type-Options` entry.** `impact-analysis.md` calls out its preservation.
- **Do not add headers beyond the four.** `Strict-Transport-Security` in particular is a **hosting/domain decision** with long-lived caching consequences — flag it as a follow-up, do not add it.
- **Do not remove any `@font-face` other than Inter.** W4.15 is scoped to the single dead declaration.
- **Do not change `font-family` declarations.** W4.15 removes an unused `@font-face`; the actual font stack is unchanged.
- **Do not change the `contain-intrinsic-size` value or `threshold: 0.5`.** W4.17 is documentation only.
- **Do not convert W4.17 into an optimisation task.** R7 is explicit that the thresholds are well-tested and low-priority.
- **Do not hand-edit generated files.** `vercel.json` is source.

### Validation Rules

- **Validate `vercel.json` parses as JSON** after editing. A syntax error fails the deployment.
- **Deploy (or `npm run preview`, which does not apply Vercel headers) and inspect response headers** on `/` and `/scroll`. **Local preview will not show Vercel's headers — this requires a deployment.**
- **Confirm `/scroll` still resolves** on the deployment — this is the SPA-rewrite regression check for W4.11.
- **Confirm the computed `font-family` for body text is unchanged** before and after removing the Inter `@font-face`. If Inter was in a fallback stack and the file 404s today, removal is still safe — but verify no glyph substitution occurs.
- Screenshot diff at all 13 widths → expect zero diff.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **`vercel.json` is a whole-file contract.** A single trailing comma breaks the deploy. Validate JSON explicitly rather than eyeballing it.
- **The `headers` array replaces, not appends, when the same `source` matches.** `impact-analysis.md`'s note about "the four-way `headers` merge" implies separate entries per `source`. **If two entries share the same `source`, Vercel's merge behaviour matters** — verify the resulting response contains all four headers, not just one.
- **`X-Frame-Options: DENY` blocks embedding entirely.** `SAMEORIGIN` is more permissive. If the portfolio is ever previewed in an iframe (e.g. a Vercel preview inside a dashboard), `DENY` blocks it. Choose deliberately and record the choice.
- **`Referrer-Policy: no-referrer` would strip referrer data** that may matter for the portfolio's analytics or link previews. `strict-origin-when-cross-origin` is the modern default; do not tighten it without a reason.
- **`Permissions-Policy` with a wrong directive name is silently ignored.** Names are exact (`geolocation`, `camera`, `microphone`, `interest-cohort`). A typo produces no error and no effect.
- **`Permissions-Policy` is irreversible for cached responses** once deployed with a restrictive value — browsers cache policy decisions. Start permissive.
- **Removing Inter's `@font-face` when a `local()` or CDN `src` is declared** — if `src` points at a remote font and any text still resolves to Inter, the rendering changes. **Verify computed styles, not just the file.**
- **A dead `@font-face` may not be dead.** It might be referenced by a font stack in a file not in the plan's scope. Grep every `font-family` across all CSS before removing.
- **Font loading is asynchronous** — a font swap after removing a declaration can shift text metrics. Screenshot diff catches it.
- **`content-visibility: auto` interacts with the IntersectionObserver.** W4.17 does not change it, but a future optimiser will. The comment must record enough to make that future decision safe: the current value, why, and that it was deliberately left alone.
- **The comment at `:76` must not change behaviour.** Verify the diff shows a comment-only change to that line.
- **`threshold: 0.5` on the observer** is documented here but owned by no implementation change. Record that the value is intentionally unchanged under R7.
- **Local `npm run preview` does not apply Vercel headers.** Do not mark W4.11 verified from a local run; the verification is post-deployment and belongs to the Stage 4 exit criteria.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Parse `vercel.json` as JSON → valid; no trailing commas or comments.
- [ ] Confirm the SPA rewrite block is byte-identical to before.
- [ ] Confirm all four headers are present in the file, with `X-Content-Type-Options` unchanged.
- [ ] Confirm no `Content-Security-Policy` anywhere in the file.
- [ ] Deploy → `curl -I https://<domain>/` shows all four headers.
- [ ] Deploy → `curl -I https://<domain>/scroll` returns 200 (SPA rewrite intact).
- [ ] Grep all CSS for `font-family` → confirm no rule depends on Inter.
- [ ] DevTools computed `font-family` for body text before vs after → unchanged.
- [ ] Confirm exactly 20 `@font-face` blocks remain and Inter is not among them.
- [ ] Confirm `HomePageScroll.tsx:76` diff is comment-only; `contain-intrinsic-size` and `threshold: 0.5` unchanged.
- [ ] Screenshot diff at all 13 widths → expect zero diff.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. W4.11's verification **requires a deployment**; record that dependency in the task's completion notes and complete it at Stage 4 exit.

---

## Definition of Done

- [ ] Four safe security headers added; existing entry preserved; no CSP
- [ ] SPA rewrite intact; `/scroll` resolves on the deployment
- [ ] `vercel.json` is valid JSON
- [ ] Dead Inter `@font-face` removed; other 20 declarations untouched
- [ ] Body text's computed font unchanged
- [ ] `content-visibility` threshold decision documented; no threshold change
- [ ] `threshold: 0.5` unchanged
- [ ] `X-Frame-Options` / `Referrer-Policy` / `Permissions-Policy` values chosen deliberately and recorded
- [ ] `Strict-Transport-Security` flagged as a follow-up, not added
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes; `npm run lint` green
- [ ] Validation completed (header verification pending deployment)

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W4.11, W4.15, W4.17; Stage 4 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 4 rows for all three items
- `.opencode/plan/codebase-quality-seo/risks.md` — R7, R19, § *Deployment & Configuration Risk*
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 0 overlap map; § *Expected Changes* (`vercel.json`, `index.css`)
- `vercel.json:1-5`
- `src/index.css:2, 34-54`
- `src/pages/HomePageScroll/HomePageScroll.tsx:76`