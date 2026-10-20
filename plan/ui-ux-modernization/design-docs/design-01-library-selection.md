# Design 01 — Component Library Selection (Option C Sub-decision)

## Context

### Problem Statement

Option C was approved: adopt a component library to obtain accessible, polished primitives instead of hand-rolling CSS. The choice must be between **MUI v6 (+Emotion)** and **Mantine**, evaluated against five criteria:

1. Bundle / build cost in this pnpm monorepo
2. Theming fidelity to the existing teal brand palette
3. How cleanly it themes FullCalendar v6 via `--fc-*` custom properties (FullCalendar DOM must not be restyled with library components)
4. SSR / Emotion friction with Vite 5
5. Risk to the existing Playwright E2E suite

### Goals

- Pick exactly one library, with a version, justified against all five criteria.
- Confirm the choice is compatible with the project's pinned React version.
- Keep the FullCalendar DOM and the existing E2E assertions intact.

### Constraints

- **React 18.2** (`apps/web/package.json`). No React major upgrade is in scope.
- **Vite 5.4.21** installed (`apps/web/node_modules/vite/package.json`); `vite.config.ts` is 15 lines.
- FullCalendar v6 stays; `.fc-timegrid-slots`, `.fc-timegrid-body`, `.fc-event`, `.fc-timegrid-slot` DOM must survive.
- `apps/web/e2e/TASK-01.spec.ts` and `TASK-02.spec.ts` are acceptance criteria and must not be modified.
- One `<aside>` element must remain in the DOM (`TASK-02.spec.ts:82`).
- All E2E-asserted copy must be preserved byte-for-byte.

---

## Options Considered

### Option A (Selected): Mantine v8 — `@mantine/core@8.3.18`

#### Description

Adopt Mantine **8.3.18** (latest 8.x line) plus `@mantine/hooks`, `@mantine/modals`,
`@mantine/notifications`, and `@tabler/icons-react@3.48.0`. Mantine v7+ ships **static CSS
files with CSS variables** — no runtime CSS-in-JS engine. Theming goes through
`createTheme()` + `MantineProvider`, which emits `--mantine-*` CSS custom properties that
the existing `design-tokens.css` layer maps onto.

#### Advantages

- **React 18 compatible.** Mantine 8.x declares `react: ^18.x || ^19.x` peer. Mantine 9.x
  (released 2026-03-31) raised the peer to `^19.2.0` — incompatible with this project's
  React 18.2 without an out-of-scope framework upgrade.
- **Static CSS, no styling runtime.** Mantine v7 replaced Emotion with CSS modules +
  CSS variables. Nothing is injected at runtime, so there is no Emotion/SSR concern
  under Vite 5 at all, and no `jsxImportSource` tsconfig change is needed for style props.
- **Best FullCalendar interop.** Because Mantine's CSS is imported statically at build
  time, cascade order against FullCalendar's runtime-injected stylesheet is
  deterministic. A `--fc-*` custom-property block scoped to `.brk-calendar` therefore wins
  reliably, and no `!important` fight is required.
- **Lighter bundle.** Measured ~67 kB gzip added for a four-component page versus
  ~112 kB for MUI on the same benchmark (July 2026); MUI grows to 250–400 kB in real apps
  because the Emotion runtime and theming layer ship with the first import.
- **Dark mode is first-class.** `MantineProvider defaultColorScheme` +
  `ColorSchemeScript` set `data-mantine-color-scheme="light|dark"`, which maps directly onto
  the `[data-theme]` semantic tier required by the approved scope.
- **Dense widgets included** — `Modal`/`Drawer`, `Menu`, `Tooltip`, `SegmentedControl`,
  `NumberInput`, `Select`, `Switch`, `Badge`, `Alert`, `Skeleton`, plus `@mantine/modals`
  for an imperative `modals.open()` API that suits the seven existing modals/dialogs.

#### Disadvantages

- Requires a PostCSS step (`postcss-preset-mantine`, `postcss-simple-vars`) — a new
  build-tool touch point in a project that currently has none.
- `createTheme` wants 10-shade color tuples; the existing hex palette must be expanded into
  tuples (handled in `design-02`).
- Mantine v8 is one major behind current (v9). A future Mantine 9 adoption is blocked on
  React 19.2+ and is recorded as deferred work.
- Mantine ships less "enterprise-battle-tested" history than MUI.

#### Effort Estimate

Medium

---

### Option B (Rejected): MUI v6 + Emotion

#### Description

Adopt `@mui/material` v6 with `@emotion/react` / `@emotion/styled`, wrap the app in
`ThemeProvider` with a custom teal theme, and replace Header/Sidebar/Modal/Button/dialogs.

#### Advantages

- Best-in-class accessibility and keyboard navigation; broadest component catalogue.
- `createTheme` palette maps almost 1:1 onto the existing token names
  (`primary.main/light/dark`, `error`, `background.paper`, `divider`, `text.primary/secondary`).
- `@mui/material` v6 supports `colorSchemes: { light, dark }` with CSS variables, so the
  dark tier is achievable.

#### Disadvantages

- **MUI v6 is two majors behind.** MUI v9 shipped 2026-04-08. Choosing v6 means adopting a
  non-current major that already has a published migration path — technical debt created on
  day one.
- **Emotion runtime.** Requires `@emotion/react` + `@emotion/styled` in the bundle
  (~21 kB min for `@emotion/react` alone, before any component), plus a styling runtime
  injected at runtime.
- **Non-deterministic cascade against FullCalendar.** Emotion injects rules into `<head>` at
  runtime; FullCalendar also injects its own CSS at runtime. Overriding FullCalendar
  reliably requires `StyledEngineProvider injectFirst` and careful ordering — strictly more
  fragile than static CSS for the `--fc-*` layer that the calendar UX depends on.
- **Heaviest option by a wide margin** (~112 kB gzip baseline, 250–400 kB typical).
- CSS-prop usage requires `jsxImportSource: "@emotion/react"` in `apps/web/tsconfig.json`.
- Larger chance of a "three visual languages" outcome: MUI theme vs. existing
  `design-tokens.css` vs. the `!important` calendar sheet.

#### Effort Estimate

Large

---

### Custom Option

A user-defined variant (e.g. a smaller library such as Radix primitives + hand-rolled
styling, or adopting Mantine alongside a React 19 upgrade) remains possible. It was not
selected because the approved Option C explicitly calls for a component library, and the
React 19 upgrade would materially widen the delivery beyond the approved scope.

---

## Decision

### Selected Option

**Mantine v8.3.18** — `@mantine/core@8.3.18`, `@mantine/hooks@8.3.18`,
`@mantine/modals@8.3.18`, `@mantine/notifications@8.3.18`, `@tabler/icons-react@^3.48.0`.

### Reasoning

Decisive on three of the five criteria:

1. **React compatibility (hard constraint).** Mantine 9 requires React 19.2+; the project is
   on 18.2. Mantine 8 is the newest Mantine line that supports React 18, so v8.3.18 is the
   only viable Mantine choice without a framework upgrade. MUI v6's equivalent status is
   worse — it is superseded by two majors.
2. **FullCalendar interop.** The user's core complaint is the calendar. Mantine's static CSS
   gives deterministic cascade ordering, which is exactly what makes a clean `--fc-*`
   theming layer possible without `!important`. Emotion's runtime injection is the single
   biggest technical risk to the calendar work.
3. **Cost.** Mantine is materially lighter and adds no styling runtime. Vite 5 friction is
   limited to a conventional PostCSS config.

Where MUI is stronger — palette ergonomics — the gap is small and fully absorbed by
`design-02`, which makes `design-tokens.css` the single source of truth and derives the
Mantine `teal` tuple from the existing hexes rather than the reverse.

### Trade-offs Accepted

- PostCSS is added to `apps/web` where none exists today.
- We sit on Mantine v8, not v9. Upgrade is deferred until/unless React moves to 19.2+.
- Mantine must be added to `packages/ui` as a dependency (or peer) since `packages/ui`
  becomes the brand wrapper layer.

---

## Impact

### Documentation Affected

- `.opencode/plan/ui-ux-modernization/design-docs/design-02-theme-token-architecture.md`
- `.opencode/plan/ui-ux-modernization/design-docs/design-04-calendar-ux.md`
- `.opencode/plan/ui-ux-modernization/design-docs/design-06-migration-plan.md`

### Architecture Affected

- `apps/web` — new runtime + dev dependencies, `postcss.config.cjs`, provider tree in `main.tsx`
- `packages/ui` — gains `@mantine/core` peer dependency; becomes the brand wrapper layer
- Build — PostCSS pipeline introduced into the Vite build

### Dependencies Added

| Package | Version | Scope | Reason |
|---|---|---|---|
| `@mantine/core` | `8.3.18` | runtime | Component library |
| `@mantine/hooks` | `8.3.18` | runtime | Peer of core |
| `@mantine/modals` | `8.3.18` | runtime | Imperative modal manager for the 7 existing dialogs |
| `@mantine/notifications` | `8.3.18` | runtime | Toast layer for save/error feedback |
| `@tabler/icons-react` | `^3.48.0` | runtime | Icon set (peer `react >= 16`, React 18 safe) |
| `postcss` | `^8.4` | dev | Mantine PostCSS preset host |
| `postcss-preset-mantine` | `^1.18` | dev | Mantine mixins/breakpoint helpers |
| `postcss-simple-vars` | `^7.0.1` | dev | Breakpoint variables for the preset |

No dependency is removed. FullCalendar packages are untouched.

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| Mantine 8 static CSS and FullCalendar runtime CSS conflict in cascade order | Low | Medium | Load order fixed in `main.tsx`; `--fc-*` scoped to `.brk-calendar`; verify with Playwright screenshots during QA |
| PostCSS preset misconfiguration breaks the Vite build | Medium | Medium | Add `postcss.config.cjs` first, run `build` immediately, before any component migration |
| Mantine 9 forced later by an unrelated upgrade | Low | Medium | Pin exact `8.3.18`; record React 19.2 prerequisite in this doc and in `concerns` |
| `packages/ui` consuming Mantine creates a circular/peer resolution problem under pnpm | Medium | Medium | Declare `@mantine/core` as a `peerDependency` **and** `devDependency` in `packages/ui`; keep the app as the CSS owner |

---

## Open Questions

- None blocking. Confirm at install time whether a newer `8.3.x` patch has shipped; the
  design targets the 8.x API surface, so patch drift is acceptable.