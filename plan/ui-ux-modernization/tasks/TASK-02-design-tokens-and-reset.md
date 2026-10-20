# TASK-02 — Design Tokens, Global Reset and Breakpoint Alignment

## Task Information

### ID

TASK-02

### Title

Restructure `design-tokens.css` into three tiers, move the global reset, remove the duplicate token load, align breakpoints and fix the dark event palette

### Phase

Phase 2a — Design tokens, theme, providers, dark mode

### Owner

senior-frontend

### Status

Pending

---

## Description

Turn `design-tokens.css` (84 lines) into the single source of truth with three labelled tiers —
Tier 1 primitives (existing `--clr-*`, `--font-*`, `--space-*`, `--radius-*`, `--shadow-*`
preserved verbatim), Tier 2 semantic light (`--brk-*`), Tier 3 semantic dark scoped to
`[data-mantine-color-scheme="dark"]`.

Three correctness fixes travel with this task: the global reset currently lives in the wrong file,
the token sheet is imported twice, and the design's proposed dark event colour fails WCAG AA.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/styles/design-tokens.css` | modify — 3 tiers; **remove** the `*` / `body` reset |
| 2 | `apps/web/src/styles/global.css` | modify — **receive** the reset; drop the line-1 duplicate `@import`; real breakpoint media queries |

### File count

2 (both modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/main.tsx` — TASK-03 owns the stylesheet import list
- `apps/web/index.html` — TASK-03 owns `ColorSchemeScript`
- `apps/web/src/theme/**` — TASK-03 creates it
- `apps/web/src/styles/CalendarStyles.css` — TASK-09 owns it
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)
- `apps/web/src/**/*.tsx` — no component changes in this task

---

## Constraints Satisfied

- `B4` — the dark event palette MUST NOT ship `#3CC4BD` with white text (≈2.1:1)
- `M2` — reset moves to `global.css`; exactly one token load survives
- `M4` — PostCSS/media-query breakpoints MUST be numerically identical to the Mantine theme
- `M13` — token sheet loads after `@mantine/core/styles.css`
- `P-6`, `P-9` — no config change; `design-tokens.css` and `global.css` stay under 400 lines
- `R-08`, `R-10`, `R-20` — contrast, cascade order, z-index scale

---

## Implementation Steps

1. **`design-tokens.css` — Tier 1 (primitives).** Copy all 16 existing `--clr-*` values, all
   `--font-*`, `--space-*`, `--radius-*`, `--shadow-*` custom properties **verbatim**. Add the
   approved additions: `--clr-border`, `--clr-border-subtle`, `--clr-text`, `--clr-text-muted`,
   `--clr-text-subtle`, `--clr-overlay`; `--space-2xs`, `--space-3xs`; `--z-base: 0`,
   `--z-sticky: 10`, `--z-drawer: 100`, `--z-modal: 200`, `--z-toast: 300` (aligned with Mantine's
   own scale, not a second scale — `R-20`); `--motion-fast/base/slow`, `--ease-standard`.
2. **Remove the global reset from `design-tokens.css`.** Delete the `* { box-sizing: border-box }`
   and `body { margin: 0; ... }` blocks. They move to `global.css` in step 6 (`M2`).
   Keep `@keyframes spin`, `fadeIn`, `slideUp` and the `:focus-visible` rule here.
3. **`design-tokens.css` — Tier 2 (semantic, light).** Add the `--brk-*` names from the approved
   token table: `--brk-bg-canvas`, `--brk-bg-raised`, `--brk-bg-sunken`, `--brk-bg-subtle`,
   `--brk-border-default`, `--brk-border-subtle`, `--brk-text-primary`, `--brk-text-secondary`,
   `--brk-text-muted`, `--brk-text-brand`, `--brk-text-on-brand`, `--brk-accent-brand`,
   `--brk-accent-danger`, `--brk-accent-success`, `--brk-accent-warning`,
   `--brk-event-generated`, `--brk-event-external`, `--brk-event-preview`, `--brk-event-blocked`,
   `--brk-today-tint`, `--brk-now-indicator`, `--brk-focus-ring`, `--brk-radius-*`,
   `--brk-shadow-*`. All Tier-2 values resolve from Tier-1 via `var(...)`.
4. **Add the workday tint token** `--brk-workday-tint` (light and dark) — TASK-09 needs it for the
   CSS-only workday shading mandated by `B3`.
5. **`design-tokens.css` — Tier 3 (semantic, dark)**, in a
   `[data-mantine-color-scheme="dark"]` block. Use the approved dark neutrals
   (`--brk-bg-canvas #0E1618`, `--brk-bg-raised #16232B`, `--brk-bg-sunken #111C1C`,
   `--brk-text-primary #E6F0EF`, `--brk-text-secondary #A8BCBB`, `--brk-text-muted #7A9190`).
   **`B4` correction — mandatory:** the design proposes `--brk-event-generated: #3CC4BD` and
   `--brk-event-preview: #3CC4BD` with `--brk-text-on-brand: #FFFFFF`. That pair is ≈2.1:1 and
   fails AA. Adopt one of the two accepted resolutions instead:
   - keep the light teal fill and switch event text to the dark-tier `--brk-text-primary`; **or**
   - use a darker teal from the approved teal tuple (`#2BA8A2` / `#1E8C86`) with white text.
   Contrast-check **every** event token (`--brk-event-generated`, `--brk-event-external`,
   `--brk-event-preview`, `--brk-event-blocked`) **and** the brand fill used by the primary buttons
   against its text in **both** tiers. Every pair MUST be ≥ 4.5:1 for body text and ≥ 3:1 for
   large text / non-text. Record the chosen pair in a comment next to the token.
6. **`design-tokens.css` — Mantine variable pinning.** In `:root`, pin
   `--mantine-color-body`, `--mantine-color-text`, `--mantine-color-default`,
   `--mantine-color-default-border`, `--mantine-color-dimmed` to the corresponding `--brk-*`
   values so Mantine and the semantic tier cannot diverge. This block MUST come **after** the
   Tier-2/Tier-3 declarations (`M13`).
7. **`global.css` — receive the reset.** Move `* { box-sizing: border-box }` and the `body` rule
   here, re-pointed at `--brk-*` (`background: var(--brk-bg-canvas)`, `color: var(--brk-text-primary)`,
   `font-family: var(--font-stack)`). Place it **above** any `@media` block so it always applies.
8. **`global.css` — remove the duplicate token load.** Delete line 1
   (`@import "./design-tokens.css";`). `main.tsx` already imports the sheet (`M2`). Exactly one
   token load must remain.
9. **`global.css` — replace the dead responsive rules.** The current `@media (max-width: 768px)` /
   `480px` blocks target `.sidebar-collapsed`, `.header-view-switcher`, `.header-title`,
   `.calendar-toolbar`, `.modal-content` — **none of which exist in any component**. Delete them and
   replace with media queries whose breakpoint values are **numerically identical** to the Mantine
   theme breakpoints TASK-03 will declare: `max-width: 48em` (Mantine `sm`), `max-width: 62em`
   (`md`), `max-width: 75em` (`lg`). Do not use `36em` under the name `sm` — Mantine's `sm` is
   `48em` and a mismatch shifts every breakpoint by 12em (`M4`). **Keep `.sr-only`** unchanged; it
   is used by screen-reader-only text.
10. **Add a comment** to both files recording the breakpoint contract
    (`xs 36em / sm 48em / md 62em / lg 75em`) and a cross-reference to `src/theme/mantineTheme.ts`.

---

## Acceptance Criteria

- [ ] Every pre-existing `--clr-*`, `--font-*`, `--space-*`, `--radius-*`, `--shadow-*` name still resolves with an unchanged value
- [ ] `design-tokens.css` contains three clearly labelled tiers plus the Mantine variable pinning block
- [ ] `design-tokens.css` contains **no** `* { box-sizing }` or `body { … }` rule
- [ ] `global.css` contains the reset and **no** `@import "./design-tokens.css"`
- [ ] Exactly one place in the codebase imports `design-tokens.css` (`main.tsx`)
- [ ] `global.css` contains no selector for `.sidebar-collapsed`, `.header-view-switcher`, `.header-title`, `.calendar-toolbar`, `.modal-content`
- [ ] `.sr-only` still present in `global.css`
- [ ] Media-query breakpoint values are `48em` / `62em` / `75em`, matching the theme constants TASK-03 must use
- [ ] `--brk-event-generated` is **not** `#3CC4BD` paired with white text; the chosen dark-tier pair is ≥ 4.5:1 and is documented in a comment
- [ ] Both stylesheets are under 400 lines
- [ ] No component, config, manifest or E2E file was modified

---

## Dependencies

### Required Tasks

- TASK-01

### Dependency Notes

Mantine packages must be installed before the token sheet can pin `--mantine-color-*`.
TASK-03 must consume the breakpoint values defined here.

---

## Validation Gate

1. `run-package-command build apps/web` → **PASS**
2. `run-package-command test apps/web` → **9/9 PASS** (confirm no stale dev server — `M14`)
3. No lint step (`P-4`)

---

## Edge Cases

- **CSS bundle shrinks but the app loses its background** — the reset was deleted instead of moved.
  Both files must contain their half.
- **Token sheet applied twice** — a `.cjs`/CSS `@import` duplicate was left behind. Exactly one
  import path may load `design-tokens.css`.
- **`--brk-*` resolving to an empty string** — a Tier-2 name references a Tier-1 name that does not
  exist. `var()` silently falls back; grep the token file for every referenced name.
- **Dark tier unreachable** — no `[data-mantine-color-scheme="dark"]` attribute is set anywhere yet
  (`R-24`). TASK-03 introduces it; do not add a JS toggle in this task.
- **`color-mix()` support** — used by `--brk-today-tint`. It is already used by
  `CalendarStyles.css`, so browser support is unchanged; if the target browser list changes, fall
  back to a literal `rgba()` rather than leaving an invalid value.
- **Breakpoint drift** — an agent writing a media query later must read the numbers from
  `global.css`, not from a design doc. `M4` makes drift a phase-gate failure.
- **`!important` reintroduced** — not permitted in the token files; the reset must stay overridable.
- **400-line ceiling** — if Tier 2 + Tier 3 exceed 400 lines, split the dark tier into
  `design-tokens.dark.css` imported **after** `design-tokens.css`, and report the deviation.

---

## Testing

### Unit Tests

- [ ] None — stylesheet-only change, no TypeScript

### Integration Tests

- [ ] The app renders identically in the light tier (`build` + E2E gate covers it)

### Manual Validation

- [ ] Dev server boots with no CSS parse warning
- [ ] Computed `body` background equals `--brk-bg-canvas`
- [ ] Forcing `[data-mantine-color-scheme="dark"]` on `<html>` changes the dark neutrals

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] `run-package-command test apps/web` must remain 9/9 green

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/impact-analysis.md` — M2, M4, M13](../planning/impact-analysis.md#material-corrections-m1m18)
- [`planning/feasibility.md` — B4](../planning/feasibility.md#b4--fix-dark-mode-event-contrast-3cc4bd-with-white-text-is-not-acceptable)
- [`design-docs/design-02-theme-token-architecture.md`](../design-docs/design-02-theme-token-architecture.md)
- `apps/web/src/styles/design-tokens.css`, `apps/web/src/styles/global.css`