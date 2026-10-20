# TASK-03 — Mantine Theme, Provider Tree and Stylesheet Import Order

## Task Information

### ID

TASK-03

### Title

Create `createTheme` + `ThemeProvider`, wire `MantineProvider`/`ModalsProvider`/`Notifications` into `main.tsx`, add `ColorSchemeScript`, fix the stylesheet import order

### Phase

Phase 2b — Design tokens, theme, providers, dark mode

### Owner

senior-frontend

### Status

Pending

---

## Description

There is currently **no** Mantine provider anywhere in the app. This task creates the theme derived
from Tier-1 primitives, composes the provider tree, makes the dark tier reachable via
`data-mantine-color-scheme`, and fixes the deterministic stylesheet load order so app-level rules
win over Mantine's.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `apps/web/src/theme/mantineTheme.ts` | **create** |
| 2 | `apps/web/src/theme/ThemeProvider.tsx` | **create** |
| 3 | `apps/web/src/theme/breakpoints.ts` | **create** — shared breakpoint constants |
| 4 | `apps/web/src/main.tsx` | modify — provider tree + stylesheet imports |
| 5 | `apps/web/index.html` | modify — insert `ColorSchemeScript` |

### File count

5 (3 created, 2 modified)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/src/styles/**` — TASK-02 owns `design-tokens.css` and `global.css`; TASK-09 owns `CalendarStyles.css`
- `apps/web/src/components/**`, `apps/web/src/pages/**`, `apps/web/src/features/**`
- `apps/web/src/contexts/SettingsContext.tsx` — colour scheme is `localStorage`-backed, **not** server-persisted
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)

---

## Constraints Satisfied

- `B4` — `primaryShade` and the theme colours must agree with the contrast-corrected tokens
- `M4` — Mantine breakpoints MUST be numerically identical to the `global.css` media queries
- `M13` — `@mantine/core/styles.css` MUST be imported before the app stylesheets
- `P-6`, `P-9` — no config change; every new file under 400 lines
- `R-24` — the dark tier must become reachable; the colour-scheme source is chosen deliberately here
- `R-20` — no second z-index scale

---

## Implementation Steps

1. **Create `apps/web/src/theme/breakpoints.ts`.** Export a single frozen constant object used by
   both the theme and any shell config:
   `{ xs: "36em", sm: "48em", md: "62em", lg: "75em", xl: "88em" }` plus a `parseRemEm`/comment
   reminder. These are the **only** legal breakpoint numbers in the delivery (`M4`). Add a comment
   stating that `global.css` media queries must repeat these values and that the design's
   `sm = 36em` figure was rejected because Mantine's `sm` is `48em`.
2. **Create `apps/web/src/theme/mantineTheme.ts`** exporting a `createTheme({...})` result:
   - `primaryColor: "teal"` with the 10-step `teal` tuple so `teal[4] = #3CC4BD`,
     `teal[5] = #2BA8A2`, `teal[6] = #1E8C86`, `teal[0] = #E8F6F5` (existing brand hexes land on
     canonical slots with no value change);
   - `coral` (`#EF6C4A` at index 5, `#D45233` at 6) and `sky` (`#5DADE2` at 5);
   - `primaryShade: { light: 6, dark: 4 }` — keeps filled buttons legible in both tiers and
     satisfies `B4`;
   - `fontFamily: "var(--font-stack)"`, `defaultRadius: "md"`, heading sizes h1/h2/h3;
   - `breakpoints: { xs, sm, md, lg, xl }` imported from `breakpoints.ts`.
   Add a comment linking the tuple back to `design-tokens.css` (the tokens are the source of truth;
   the theme is derived from them).
3. **Create `apps/web/src/theme/ThemeProvider.tsx`.** A component that composes, outermost first:
   `<MantineProvider theme={mantineTheme} defaultColorScheme="auto" forceColorScheme={undefined}>`
   → `<ModalsProvider>` → `<Notifications position="top-right" />` → `children`.
   - Colour-scheme source: **Mantine's `localStorageColorSchemeManager`** (default) with
     `defaultColorScheme="auto"`. Do **not** add a server-persisted `colorScheme` — that would be a
     backend contract change and is out of scope (`R-24`).
   - `MantineProvider` MUST wrap everything that can render a Mantine component. `ErrorBoundary`
     stays **outermost** so a provider crash still falls back to `ErrorFallback`.
   - Accept `{ children: React.ReactNode }` and return a fragment of the four wrappers.
4. **Modify `apps/web/src/main.tsx`:**
   - Wrap the existing tree in `<ThemeProvider>`, placed **directly inside** `<React.StrictMode>` and
     **outside** `<ErrorBoundary>`? — No: place it **inside** `<ErrorBoundary>` and **outside**
     `<QueryClientProvider>`, so providers are inside the boundary but wrap every consumer. Keep
     `ErrorBoundary` outermost inside `StrictMode`.
   - Preserve the existing nesting exactly: `QueryClientProvider` → `AuthProvider` →
     `SettingsProvider` → `TaskProvider` → `BlockProvider` → `CalendarProvider` → `PreviewProvider`
     → `App`. Only add the new wrapper; remove nothing.
   - **Replace the stylesheet import list** with this exact, deterministic order:
     ```
     import "@mantine/core/styles.css";
     import "@mantine/notifications/styles.css";
     import "./styles/design-tokens.css";   /* Tier 1/2/3 + Mantine variable pinning */
     import "./styles/global.css";
     import "./styles/CalendarStyles.css";  /* --fc-* layer, must stay last */
     ```
      TASK-09 will append the calendar partials **after** `CalendarStyles.css`.
      Installed `@mantine/modals` does not export `styles.css`; importing it breaks the build.
5. **Modify `apps/web/index.html`** — insert Mantine's `<ColorSchemeScript />` in `<head>`
   **before** the module script, so the dark tier is applied before first paint (no white flash).
   Import it from `@mantine/core` inside the script that already loads `main.tsx`, or add a separate
   `type="module"` script; either is acceptable as long as it executes before the app renders.
6. **Verify the tier switch is observable:** `<html>` must receive `data-mantine-color-scheme`
   (`light` or `dark`). That attribute is the scope selector Tier 3 and TASK-09 rely on.

---

## Acceptance Criteria

- [ ] `apps/web/src/theme/breakpoints.ts` is the single source of `xs 36em / sm 48em / md 62em / lg 75em`
- [ ] `apps/web/src/theme/mantineTheme.ts` exports a `createTheme` result with `teal`, `coral`, `sky`, `primaryShade: { light: 6, dark: 4 }` and the shared breakpoints
- [ ] `apps/web/src/theme/ThemeProvider.tsx` composes `MantineProvider` → `ModalsProvider` → `Notifications` → children
- [ ] `ErrorBoundary` remains the outermost component inside `StrictMode`
- [ ] `main.tsx` imports Mantine stylesheets **before** all app stylesheets, `CalendarStyles.css` last
- [ ] The seven existing context providers are unchanged and still in the same order
- [ ] `index.html` contains `<ColorSchemeScript />` before the app script
- [ ] `<html>` receives `data-mantine-color-scheme`; both tiers render
- [ ] `build apps/web` → **PASS**, `test apps/web` → **9/9 PASS**
- [ ] No E2E, config or manifest file was modified

---

## Dependencies

### Required Tasks

- TASK-02

### Dependency Notes

TASK-02 must land first: the theme derives from Tier-1 primitives and its breakpoints must match
the media queries written there. TASK-09 appends calendar partial imports to `main.tsx` afterwards.

---

## Validation Gate

1. `run-package-command build apps/web` → **PASS**
2. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
3. No lint step (`P-4`)

---

## Edge Cases

- **Mantine renders unstyled** — `@mantine/core/styles.css` is imported after an app stylesheet, or
  PostCSS never ran (TASK-01). Re-check the import order before anything else.
- **Dark tier never activates** — no stored preference and `defaultColorScheme` misconfigured. The
  attribute must appear on `<html>`; verify with `localStorage` cleared.
- **White flash before hydration** — `ColorSchemeScript` missing or placed after the app script.
- **Breakpoint mismatch** — a shell agent later hardcodes `48em` where the theme says `sm`. Both
  sides MUST import from `breakpoints.ts` or repeat the value with the cross-reference comment.
- **Mantine variable pinning loses** — Tier 3 in `design-tokens.css` must come after the
  `:root` pinning block; Mantine's `CssVariablesGenerator` uses a higher-specificity selector.
- **`Notifications` needs a provider** — rendering `notifications.show(...)` without
  `<Notifications>` throws. It is composed in step 3.
- **Duplicate Mantine instance** — two copies in `node_modules/.pnpm` produce broken context and
  styling. Report it; do not "fix" it by reordering imports.
- **StrictMode double-invocation** — provider composition must be side-effect free; no
  `localStorage` writes during render.

---

## Testing

### Unit Tests

- [ ] None — provider/theme wiring has no isolated unit target in this repo

### Integration Tests

- [ ] `run-package-command test apps/web` is the gate; `build` cannot detect stylesheet-order regressions

### Manual Validation

- [ ] Both tiers render: toggle the OS colour scheme with `defaultColorScheme="auto"`
- [ ] No white flash on a dark-mode reload
- [ ] `<html data-mantine-color-scheme>` is present in the served DOM

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Do not add a Playwright project, config change, or fixture for colour scheme (`P-6`)

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/feasibility.md` — Phase 2](../planning/feasibility.md#phase-2--design-tokens-theme-mantine-providers-dark-mode)
- [`planning/impact-analysis.md` — M4, M13](../planning/impact-analysis.md#material-corrections-m1m18)
- [`design-docs/design-02-theme-token-architecture.md`](../design-docs/design-02-theme-token-architecture.md)
- `apps/web/src/main.tsx`, `apps/web/index.html`
