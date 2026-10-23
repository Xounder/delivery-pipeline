# Task — Keyboard Focus-Visible Outlines & Menu Divider Positioning Context

## Task Information

### ID

TASK-04

### Title

Restore working keyboard focus indicators and establish the divider pseudo-element's positioning context

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.14** | High | Replace `all:unset` UA-ring stripping and the removed `outline: 4px auto -webkit-focus-ring-color` with explicit `:focus-visible` outlines | `src/components/MenuSectionButton/MenuSectionButton.css:27-44`, `src/components/Button/Button.css:18-21` |
| **W1.15** | High | Establish an intended positioning context for `.menu-section__items li::after` | `src/components/MenuSectionButton/MenuSectionButton.css:56-67` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Two scoped exceptions, both pre-approved:**
- **W1.14** is the one Stage 1 item that intentionally changes the *focus* rendering. It is scoped to `:focus-visible` only so pointer users see zero change.
- **W1.15** must reproduce the *intended* divider positions. If the current stacked-at-bottom rendering turns out to be the design (see Edge Cases), document and leave rather than guess (`risks.md` R14).

---

## Description

Two defects in the two button stylesheets, both accessibility-critical.

**W1.14 — zero working focus indicators.** Two independent mechanisms destroy them:

- `MenuSectionButton.css:28` uses `all: unset`, which resets `outline` to its initial value (`none` for a non-focus-ring UA) with **no replacement**. Every menu-section button is unfocusable-in-practice for keyboard and screen-reader users.
- `Button.css:20` uses `outline: 4px auto -webkit-focus-ring-color`. Both `auto` as an outline colour and `-webkit-focus-ring-color` were removed from the platform; the whole declaration is invalid and dropped.

The fix is explicit `:focus-visible` outlines using existing brand colours. Never `outline: none`.

**W1.15 — dividers stack at the bottom.** `.menu-section__items li::after` is `position:absolute`, but no `<li>` is positioned. An absolutely-positioned element whose nearest positioned ancestor is not the `<li>` resolves against `.menu-section__items` — so **every** divider is positioned relative to the list container and they all collapse to the same place (its bottom edge) instead of appearing between items.

The fix must establish the intended positioning context **without changing the rendered result** — which, per the Edge Cases below, is not yet established.

---

## Acceptance Criteria

- [ ] `MenuSectionButton.css:28` no longer uses `all: unset`, or `all: unset` is replaced by a reset that explicitly restores a visible `:focus-visible` outline.
- [ ] `Button.css:20` no longer contains `-webkit-focus-ring-color` or `outline: 4px auto`.
- [ ] Both files declare an explicit `:focus-visible` rule with a concrete `outline` (colour, width, style, offset) drawn from existing brand colours.
- [ ] Keyboard-only walkthrough confirms a visible focus indicator on **every** interactive element on both `/` and `/scroll`: menu-section buttons, the section arrows, the menu-bar view-change link, the contact-form inputs and submit button.
- [ ] Pointer interaction shows **no** visible change — no focus ring on mouse click, because the rule is `:focus-visible` scoped.
- [ ] `all: unset` (if retained in a reduced form) does not strip font, line-height, or box-sizing in a way that changes rendered size. Screenshot diff must confirm this.
- [ ] The divider pseudo-element resolves against an intended positioning context. Either the `<li>` is positioned (`position: relative`) or the pseudo-element's offsets are re-expressed so dividers appear between items.
- [ ] The rendered divider layout at all 13 widths matches the **approved** reference — either the current stacked rendering (if that is confirmed to be the intent) or the intended per-item rendering (if confirmed to be the bug). The decision is recorded with a screenshot of both states.
- [ ] Screenshot diff clean at all 13 widths for every change that is not the focus ring: 360×640 · 390×844 · 430×932 · 667×375 · 844×390 · 768×1024 · 820×1180 · 834×1194 · 1024×1366 · 1180×820 · 1280×800 · 1440×900 · 1920×1080.
- [ ] Hover-state screenshot diff clean at all 13 widths (adding `position: relative` to `<li>` must not alter hover geometry).
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01

### Dependency Notes

TASK-01 is required so `npm run build` / `npm run lint` can run.

No dependency on any other Stage 1 task — this task's files are disjoint from TASK-02, TASK-03, TASK-05, TASK-06.

**Open-Question gate (task-level, not stage-level):** `risks.md` **R16** states that unresolved brand decisions block *the specific task, not the stage*. `feasibility.md` **Open Question 3** — *"is the current stacked-at-bottom rendering the bug, or the intent?"* — blocks **W1.15 only**.

**W1.14 is NOT gated** and must be implemented regardless. If the W1.15 question cannot be answered, deliver W1.14, and record W1.15 as blocked with the evidence gathered. Do not let an unanswered question stall the focus-indicator fix, which is the higher-severity accessibility defect.

---

## File Overlap Warning

`src/components/MenuSectionButton/MenuSectionButton.css` is a **max-1-concurrent-editor** file (9 work items across the plan: W1.14, W1.15, W1.19, W3.2, W3.3, W3.4, W3.5, W3.7, W3.12).

| Other task | W-IDs | Lines touched | Merge strategy |
|---|---|---|---|
| **TASK-05** | W1.19 | `MenuSectionButton.tsx:33` (**.tsx, not .css**) | **Parallel-safe** — different file extension. `impact-analysis.md` lists W1.19 against the `.tsx`. |
| **TASK-15** | W3.2, W3.3, W3.4 | media queries + literals across 12 CSS files | **Sequential** — Stage 3. |
| **TASK-16** | W3.5, W3.7, W3.12 | `:39,47,53,65` colours, `:13-22,41` overflow, `:66` legacy props | **Sequential** — Stage 3. |

`src/components/Button/Button.css` (3 items: W1.14, W3.5, W3.7):

| Other task | W-IDs | Lines touched | Merge strategy |
|---|---|---|---|
| **TASK-16** | W3.5, W3.7 | `:15` colour token, `:2` padding | **Sequential** — Stage 3. |

No other Stage 1 task touches either stylesheet. Within Stage 1 this task is parallel-safe with TASK-02, TASK-03, TASK-05, TASK-06.

---

## Technical Context

### Relevant Components

- `MenuSectionButton` (`src/components/MenuSectionButton/`) — the menu list whose buttons lose focus rings and whose dividers stack
- `Button` (`src/components/Button/`) — the shared button primitive with the invalid outline

### Relevant Modules

- `src/components/MenuSectionButton/MenuSectionButton.css:27-44` — `all: unset`, focus styles
- `src/components/MenuSectionButton/MenuSectionButton.css:56-67` — `li::after` divider
- `src/components/Button/Button.css:18-21` — invalid `outline`

### Relevant APIs

- CSS `:focus-visible` pseudo-class
- CSS `outline` / `outline-offset`
- CSS `position: relative` / `position: absolute` containing-block resolution
- CSS `all: unset` shorthand

### Relevant Types

- None (CSS-only)

---

## Implementation Guidance

### Expected Changes

**W1.14**
1. `MenuSectionButton.css:28` — replace `all: unset` with either a targeted reset or `all: unset` plus an explicit `:focus-visible { outline: … }`. Prefer a targeted reset so font/line-height are not silently changed; if `all: unset` is kept, add back the properties the site actually depends on (`font: inherit`, `color: inherit`, etc.).
2. `MenuSectionButton.css` — add `:focus-visible` with a concrete outline using an existing brand colour. Use `outline`, never `outline: none`.
3. `Button.css:20` — replace `outline: 4px auto -webkit-focus-ring-color` with a valid `outline` using an existing brand colour.
4. Keep the ring scoped to `:focus-visible`. Do not add a `:focus` rule.

**W1.15**
5. Determine the intent first (see Edge Cases). Then either:
   - add `position: relative` to `.menu-section__items li` so each `::after` resolves against its own `<li>`, **or**
   - leave the rendering as-is and document that the stacked layout is the design.
6. Do not change `::after`'s colour, height, width, or opacity.

### Constraints

- **Never `outline: none`.** It converts a broken indicator into a deliberately absent one — strictly worse (`risks.md` § *Accessibility Regression Risk*).
- **Scope to `:focus-visible`** so pointer users see no change (`risks.md` R8).
- **Outline colour must come from existing brand colours** (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`). Do not introduce a new colour, and do not introduce a token — tokens are Stage 3 (TASK-15/16).
- **Do not add `--focus` or any custom property** in this task. Stage 3 owns token introduction.
- **Do not change `:hover` rules.** The outline must be additive to hover, not replace it.
- **Do not change any `font`, `line-height`, `padding`, or `box-sizing`** as a side effect of the `all: unset` change. If the reset is narrowed, the rendered size must be byte-identical.
- **Do not add `position: relative` to `<li>` before answering Open Question 3.** `risks.md` R14: *"If ambiguous, document and leave rather than guess."*
- Do not touch `MenuSectionButton.tsx` in this task (that is TASK-05, W1.19).
- Do not change `z-index` stacking.

### Validation Rules

- The keyboard-only focus walkthrough is a **blocking** Stage 1 exit criterion (`impact-analysis.md` § *Verification Plan*).
- Assert a *visible* outline — not merely that `:focus-visible` matches. A zero-width outline or an outline matching the background colour fails.
- Screenshot diff at all 13 widths must be clean for everything except the focus ring. Hover-state diff is also required (adding `position: relative` to `<li>` must not change hover geometry).
- Compare the divider rendering at widths 1, 6 and 11 (360×640, 768×1024, 1280×800) at minimum — the list wraps differently at narrow widths, so a per-item divider fix would look correct on desktop and wrong on mobile.
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **`all: unset` is load-bearing for the reset.** It is there to strip the UA button styling. Narrowing it may reintroduce default button padding/border/background. Screenshot-diff every width; do not assume the reset was incidental.
- **`all: unset` also resets `box-sizing` and `line-height`.** Restoring them with different values changes rendered height. Compare computed styles before and after for `font-size`, `line-height`, `box-sizing`, `padding`, `border`.
- **Browsers without `:focus-visible` support.** Every browser in the verification matrix supports it. If a legacy fallback is desired, `:focus` for non-supporting browsers plus `:focus:not(:focus-visible) { outline: none }` is the standard pattern — but that reintroduces a `:focus` rule, which R8 warns about. Prefer `:focus-visible` alone; note the choice.
- **Forced-colors / Windows High Contrast mode.** `outline: <colour>` may be ignored; system colours are substituted. Prefer `outline` (which High Contrast preserves) over `box-shadow`-based rings, which it discards.
- **The outline may be clipped by `overflow: hidden`.** `MenuSectionButton` sits inside a positioned container. If `outline-offset` pushes the ring outside a clipping ancestor, the indicator becomes partially invisible. Test with the menu open **and** closed.
- **Dark brand background.** The brand purples are very dark (`#1D0C3A`). A dark-purple outline on a dark-purple background is technically "visible" to a tool and invisible to a human. Choose a contrasting brand value and verify by eye, not by contrast-checker output alone.
- **The divider question is genuinely open (Open Question 3).** Before changing anything, determine: (a) does the design intent, as visible elsewhere in the app, place a divider between items or under the list? (b) does `HomePage`'s equivalent list render dividers correctly? (c) is `.menu-section__items li::after` visually indistinguishable from a list underline today? If any of these are ambiguous, **document and leave**. Recording a wrong "fix" violates invariant 2 — it changes the rendering on the strength of a guess.
- **Fixing W1.15 changes the divider count and positions.** Dividers currently stack at the bottom; per-item dividers would appear N times between items. If the stacked rendering is confirmed as the intent, do not "fix" it — the count difference is a visible change.
- **`position: relative` on `<li>` creates a new containing block for any descendant that is `position: absolute`.** If a nested icon or badge is absolutely positioned inside a menu item, adding `position: relative` to the `<li>` will move it. Screenshot-diff the open menu at all 13 widths, not just the closed state.
- **`position: relative` also changes paint order** for z-indexed descendants. Verify the active/selected menu item still renders above its siblings.
- **Touch devices have no keyboard.** The `:focus-visible` outline may never appear on mobile. That is expected — verify it does not appear on tap either (it should not, with `:focus-visible`).
- **Focus ring must not appear on page load.** After a reload or route change, no element may hold a stale visible focus ring.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Keyboard-only walkthrough on `/` and `/scroll`: Tab through every interactive element and confirm a visible indicator at each stop. Blocking.
- [ ] Mouse-click walkthrough: confirm no focus ring appears on any element.
- [ ] DevTools: inspect computed `outline` on a focused menu-section button and a focused `Button` — confirm a concrete colour, width > 0, and a valid style (not `auto`).
- [ ] Grep the repo for `-webkit-focus-ring-color` → expect zero hits.
- [ ] Grep the repo for `all: unset` → expect zero hits in `MenuSectionButton.css`, or a documented `:focus-visible` override.
- [ ] Screenshot diff at all 13 widths (focus state excluded).
- [ ] Hover-state screenshot diff at all 13 widths.
- [ ] Open-menu screenshot diff at widths 1 (360×640), 6 (768×1024) and 11 (1280×800) — the list wraps differently at each.
- [ ] Record the W1.15 decision and both divider states as screenshots.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; the keyboard walkthrough takes ~3–5 min and the 13-width diff ~7 min.

---

## Definition of Done

- [ ] `all: unset` no longer strips focus indicators with no replacement
- [ ] `-webkit-focus-ring-color` removed
- [ ] Explicit `:focus-visible` outlines on both stylesheets, using existing brand colours
- [ ] Keyboard-only walkthrough passes on both routes
- [ ] Pointer interaction unchanged
- [ ] Divider positioning context established or the decision to leave it documented
- [ ] Screenshot diff clean at all 13 widths (focus and hover states included)
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.14, W1.15; invariant 2 exception note
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 rows for W1.14 / W1.15; Open Question 3
- `.opencode/plan/codebase-quality-seo/risks.md` — R8, R13, R14, R16, T8, § *Accessibility Regression Risk*
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 3 overlap map (`MenuSectionButton.css`, 9 items)
- `src/components/MenuSectionButton/MenuSectionButton.css:27-44, 56-67`
- `src/components/Button/Button.css:18-21`