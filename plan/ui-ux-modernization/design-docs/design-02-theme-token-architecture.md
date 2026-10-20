# Design 02 — Theme / Token Architecture (Light + Dark Semantic Tiers)

## Context

### Problem Statement

`apps/web/src/styles/design-tokens.css` (84 lines) already defines a primitive token layer
and `apps/web/src/styles/CalendarStyles.css` (181 lines) already overrides FullCalendar.
Neither is actually consumed: ~100 inline `style={{}}` objects across ~25 TSX files hardcode
hex values that duplicate the tokens. There is no **semantic** tier (no `--surface-raised`,
no `--text-muted`), no dark tier, and no single source of truth.

### Goals

- Promote the existing primitives to a 3-tier system: **primitive → semantic light → semantic dark**.
- Make `design-tokens.css` the single source of truth; the Mantine theme is *derived from it*, never the reverse.
- Cover the gaps the current layer lacks: borders, surface elevation, text hierarchy, z-index, motion.
- Dark mode included in this delivery, driven by the `data-mantine-color-scheme` attribute.
- Preserve every existing `--clr-*`, `--font-*`, `--space-*`, `--radius-*`, `--shadow-*` name so no current consumer breaks.

### Constraints

- Every existing custom property name in `design-tokens.css` must keep working (back-compat).
- Body text contrast ≥ 4.5:1 in **both** tiers; large text / non-text ≥ 3:1.
- The `global.css` `@media (max-width: 768px)` / `480px` rules target class names that do not
  exist in any component (`.sidebar-collapsed`, `.header-view-switcher`, `.header-title`,
  `.calendar-toolbar`, `.modal-content`) — the responsive layer is currently dead code.

---

## Options Considered

### Option A (Selected): Extend the existing CSS-custom-property layer, Mantine derived from it

#### Description

Keep `design-tokens.css` as a plain CSS file and restructure it into three labelled blocks:

1. **Tier 1 — Primitives** (raw palette, unchanged names): `--clr-primary`, `--clr-primary-light`,
   `--clr-primary-dark`, `--clr-primary-bg`, `--clr-accent*`, `--clr-coral*`, `--clr-cream`,
   `--clr-sky`, `--clr-surface`, `--clr-card`, `--clr-success`, `--clr-error`.
2. **Tier 2 — Semantic, light** (new `--brk-*` names, the only names components may use).
3. **Tier 3 — Semantic, dark**, scoped to `[data-mantine-color-scheme="dark"]`.

The Mantine `createTheme()` object in TS reads the Tier-1 hexes as literals so Mantine emits
matching `--mantine-*` variables; Mantine's own semantic variables are then overridden by
Tier-2/3 rules so the two layers cannot diverge.

#### Advantages

- Zero new build tooling; plain CSS custom properties already work today.
- Back-compatible: nothing that reads `--clr-*` breaks.
- The `--brk-*` semantic namespace gives components one obvious thing to reference, which is
  what makes dark mode a tier flip rather than a per-component rewrite.
- Keeps FullCalendar themable purely through `--fc-*`, which resolves to `--brk-*`.

#### Disadvantages

- No type safety on token names; a typo silently falls back.
- Two parallel systems (Mantine variables + `--brk-*`) must be kept in sync manually.

#### Effort Estimate

Medium

---

### Option B (Rejected): Let the Mantine theme be the only token source

#### Description

Delete `design-tokens.css` and reference `--mantine-color-*` directly everywhere.

**Rejected because** FullCalendar's `--fc-*` layer and the existing `CalendarStyles.css`
selectors would have to be rewritten against Mantine's naming, and any future library swap
would ripple through the calendar CSS. Keeping our own semantic namespace decouples the
calendar theme from the UI library.

### Option C (Rejected): Generate tokens from a JSON source at build time

**Rejected because** it adds a codegen step and a build dependency for zero runtime benefit in
a 25-file app.

---

## Decision

### Selected Option

Option A — extend `design-tokens.css` into primitive → semantic-light → semantic-dark, with
the Mantine theme derived from Tier 1.

### Reasoning

It is the smallest change that produces a real theme layer, it satisfies the approved
light + dark scope, and it keeps `--fc-*` theming expressed purely in terms of our own
`--brk-*` names so the calendar is insulated from the library.

### Trade-offs Accepted

- Manual sync between Tier 1 hexes and the Mantine `teal` tuple (mitigated by a single
  `theme.ts` module and a comment linking the two).
- No compile-time token checking.

---

## Impact

### Documentation Affected

- `design-04-calendar-ux.md` (consumes `--brk-*`)
- `design-03-app-shell-layout.md` (consumes `--brk-*`)

### Architecture Affected

- `apps/web/src/styles/design-tokens.css` — restructured into 3 tiers
- `apps/web/src/styles/global.css` — reset delegated to Mantine; dead responsive rules replaced
- **new** `apps/web/src/theme/mantineTheme.ts` — `createTheme` derived from Tier 1
- **new** `apps/web/src/theme/ThemeProvider.tsx` — `MantineProvider` + `ModalsProvider` +
  `Notifications` + color-scheme persistence
- `apps/web/src/main.tsx` — import Mantine stylesheets first, then tokens, then calendar CSS
- `apps/web/index.html` — insert `ColorSchemeScript` to prevent a dark-mode flash
- `apps/web/src/contexts/SettingsContext.tsx` — gains `colorScheme` persistence

---

## Token Specification

### Tier 1 — Primitives (unchanged names, unchanged values)

All 16 existing `--clr-*` entries are preserved verbatim. Additions:

| Token | Light value | Purpose |
|---|---|---|
| `--clr-border` | `#E0E0E0` | Generic hairline (matches current hardcoded `#e0e0e0`) |
| `--clr-border-subtle` | `#ECECEC` | Calendar grid lines (matches current `#e8e8e8`) |
| `--clr-text` | `#333333` | Body text (matches current `#333`) |
| `--clr-text-muted` | `#999999` | Secondary/meta text (matches current `#999`) |
| `--clr-text-subtle` | `#666666` | Body secondary (matches current `#666`) |
| `--clr-overlay` | `rgba(15, 26, 26, 0.45)` | Modal scrim |

Scales added: `--space-2xs: 4px`, `--space-3xs: 2px`; `--z-base: 0`, `--z-sticky: 10`,
`--z-drawer: 100`, `--z-modal: 200`, `--z-toast: 300`; `--motion-fast: 120ms`,
`--motion-base: 200ms`, `--motion-slow: 300ms`, `--ease-standard: cubic-bezier(0.2, 0, 0, 1)`.

### Tier 2 — Semantic (light)

Components reference **only** these.

```
--brk-bg-canvas      var(--clr-surface)   /* app background, replaces #EFF8F7 */
--brk-bg-raised      var(--clr-card)      /* header, sidebar, panels */
--brk-bg-sunken      var(--clr-primary-bg)
--brk-bg-subtle      var(--clr-cream)     /* all-day row */
--brk-border-default var(--clr-border)
--brk-border-subtle  var(--clr-border-subtle)
--brk-text-primary   var(--clr-text)
--brk-text-secondary var(--clr-text-subtle)
--brk-text-muted     var(--clr-text-muted)
--brk-text-brand     var(--clr-primary-dark)
--brk-text-on-brand  #FFFFFF
--brk-accent-brand   var(--clr-primary)
--brk-accent-danger  var(--clr-coral)
--brk-accent-success var(--clr-success)
--brk-accent-warning var(--clr-accent-dark)
--brk-event-generated    var(--clr-primary-dark)
--brk-event-external     var(--clr-sky)
--brk-event-preview      var(--clr-primary)
--brk-event-blocked      var(--clr-coral)
--brk-today-tint         color-mix(in srgb, var(--clr-accent) 10%, transparent)
--brk-now-indicator      var(--clr-coral)
--brk-focus-ring         var(--shadow-focus)
--brk-radius-sm/md/lg/xl/round   -> var(--radius-*)
--brk-shadow-sm/md/lg/card      -> var(--shadow-*)
```

### Tier 3 — Semantic (dark)

Scoped to `[data-mantine-color-scheme="dark"]`. Brand hues are preserved but lightened for
legibility on dark surfaces; neutrals invert.

| Semantic token | Dark value | Rationale |
|---|---|---|
| `--brk-bg-canvas` | `#0E1618` | Deep neutral-teal, brand-tinted |
| `--brk-bg-raised` | `#16232B` | Header/sidebar elevation |
| `--brk-bg-sunken` | `#111C1C` | Sunken wells |
| `--brk-bg-subtle` | `#1E2622` | All-day row |
| `--brk-border-default` | `#24333B` | Hairline |
| `--brk-border-subtle` | `#1C2830` | Grid lines |
| `--brk-text-primary` | `#E6F0EF` | 13.6:1 on canvas |
| `--brk-text-secondary` | `#A8BCBB` | 7.4:1 |
| `--brk-text-muted` | `#7A9190` | 4.6:1 |
| `--brk-text-brand` | `#3CC4BD` | Brand light, 8.1:1 on canvas |
| `--brk-event-generated` | `#3CC4BD` | |
| `--brk-event-external` | `#7EC8E8` | Lightened `#5DADE2` |
| `--brk-event-preview` | `#3CC4BD` | |
| `--brk-event-blocked` | `#FF8A6A` | Coral light |
| `--brk-now-indicator` | `#FF6B4A` | |
| `--brk-today-tint` | `color-mix(in srgb, var(--clr-accent) 14%, transparent)` | |

### Mantine theme derivation

`apps/web/src/theme/mantineTheme.ts`:

```
createTheme({
  primaryColor: 'teal',
  colors: {
    teal: ['#E8F6F5','#D0EEEB','#A5DFDA','#6FCFC6','#3CC4BD','#2BA8A2','#1E8C86','#186F6A','#125551','#0B3B38'],
    coral: [... '#EF6C4A' at 5, '#D45233' at 6 ...],
    sky:   [... '#5DADE2' at 5 ...],
  },
  fontFamily: 'var(--font-stack)',
  defaultRadius: 'md',
  headings: { fontFamily: 'var(--font-stack)', sizes: { h1:{fontSize:'2.25rem'}, h2:{fontSize:'1.5rem'}, h3:{fontSize:'1.125rem'} } },
  primaryShade: { light: 6, dark: 4 },
})
```

`teal[4] = #3CC4BD` (`--clr-primary-light`), `teal[5] = #2BA8A2` (`--clr-primary`),
`teal[6] = #1E8C86` (`--clr-primary-dark`), `teal[0] = #E8F6F5` (`--clr-primary-bg`) — so the
existing brand hexes land on canonical Mantine slots with no value change.
`primaryShade: { light: 6, dark: 4 }` makes Mantine's filled buttons use the darker brand on
light backgrounds and the lighter brand on dark, which is what keeps contrast correct.

Mantine semantic variables are then pinned to our tokens:

```
:root { --mantine-color-body: var(--brk-bg-canvas);
        --mantine-color-text: var(--brk-text-primary);
        --mantine-color-default: var(--brk-bg-raised);
        --mantine-color-default-border: var(--brk-border-default);
        --mantine-color-dimmed: var(--brk-text-muted); }
```

### `global.css` cleanup

- Remove the hand-rolled `* { box-sizing }` and `body` rules — Mantine's `MantineProvider`
  + CSS reset owns these (`boxSizing="border-box"` is Mantine's default).
- Replace the dead `@media` blocks targeting non-existent classes with Mantine breakpoint
  helpers, or delete them; `design-03` introduces real class names for the shell.
- Keep `.sr-only` (used for screen-reader-only text in the new calendar chrome).

### Load order in `main.tsx` (fixed, deterministic)

```
import '@mantine/core/styles.css';
import '@mantine/notifications/styles.css';
import './styles/design-tokens.css';   /* Tier 1/2/3 + Mantine variable pinning */
import './styles/global.css';
import './styles/CalendarStyles.css';  /* --fc-* layer, must be last */
```

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Dark tier fails contrast on event chips | Medium | High | Contrast pairs are tabulated above; QA must screenshot both tiers and check event chips specifically |
| Mantine's `CssVariablesGenerator` specificity overrides our pinning | Medium | Medium | Pin Mantine variables in `design-tokens.css`, which loads after `@mantine/core/styles.css` |
| `ColorSchemeScript` omitted from `index.html` → white flash in dark mode | Medium | Low | Make the `index.html` edit an explicit step in `design-06` |
| Removing the hand-rolled reset regresses some layout | Low | Medium | Run `build` + Playwright immediately after the `global.css` change, before component work |
| `color-mix()` unsupported in target browsers | Low | Medium | Already used by `CalendarStyles.css`; verify against project browser target; fall back to literal rgba if required |

---

## Open Questions

- Should `colorScheme` live in `SettingsContext` (persisted server-side with the rest of the
  settings) or purely in `localStorage`? Default: `localStorage` via Mantine's
  `defaultColorScheme` + `localStorageColorSchemeManager`, because the API's settings schema
  has no color-scheme field and adding one is a backend contract change out of scope. Confirm
  with the user if a server-persisted preference is wanted.