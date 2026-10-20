# Design 05 — Component Inventory and Public API

## Context

### Problem Statement

`packages/ui` exports four primitives (`Button`, `Modal`, `FAB`, `BottomSheet`) with
Mantine-independent inline styles. `apps/web` uses them inconsistently — most components
hand-roll their own markup instead — which is why ~100 inline `style={{}}` objects exist.

With Mantine adopted, `packages/ui` must become a **brand wrapper layer**: app code keeps
importing from `@brkroutnxdle/ui`, and that layer applies the Brkroutnxdle teal brand and
brk sizing without every call site restating them.

### Goals

- One import surface (`@brkroutnxdle/ui`) so brand defaults are applied once.
- Cover every primitive the shell, calendar and feature panels need.
- Keep existing exports source-compatible where practical.
- Preserve the exact rendered element types and copy the E2E suite depends on.

### Constraints

- `packages/ui` builds with plain `tsc` (`packages/ui/package.json` → `"build": "tsc"`); it has
  **no CSS pipeline**. Mantine styles are imported once by `apps/web`, so `packages/ui` must
  never import a `.css` file.
- `packages/ui` must declare `@mantine/core` as a `peerDependency` (plus a `devDependency` for
  local `tsc`), so pnpm does not create a second Mantine instance.
- No component may render an E2E-asserted string, and no component may split
  `Tasks (N)` across elements.

---

## Options Considered

### Option A (Selected): Thin brand wrappers over Mantine

#### Description

Each export is a small component that renders the Mantine primitive with Brkroutnxdle
defaults (radius, brand color, size scale) and a narrowed prop surface. Complex composites
that do not exist in Mantine (`CalendarToolbar`, `StatusBar`, `EventContent`) live in
`apps/web`, not in `packages/ui`, because they are app-specific.

#### Advantages

- Brand defaults applied once; call sites stay terse.
- App code has a single import surface, so a future library swap touches `packages/ui`.
- Keeps `packages/ui` buildable with `tsc` — no new build tooling in the package.
- Accessibility comes from Mantine rather than being re-implemented.

#### Disadvantages

- Wrappers must be maintained when Mantine APIs change.
- Adds an indirection layer; debugging requires knowing which layer owns a style.

#### Effort Estimate

Medium

---

### Option B (Rejected): Delete `packages/ui`, import Mantine directly in `apps/web`

**Rejected because** it is the fastest path, but it deletes the shared package, pushes brand
styling into ~25 call sites, and contradicts the approved scope which explicitly includes
"packages/ui primitives".

### Option C (Rejected): Keep `packages/ui` as-is and add Mantine alongside

**Rejected because** two styling systems would coexist with no owner, reproducing exactly the
inconsistency this redesign removes.

---

## Decision

### Selected Option

Option A — thin brand wrappers, with app-specific composites kept in `apps/web`.

### Reasoning

It satisfies the approved scope, preserves the `tsc`-only build of `packages/ui`, and keeps
the brand in one editable place. The rule "generic primitive → `packages/ui`; app-specific
composite → `apps/web`" is what prevents the package from turning into a second `apps/web`.

### Trade-offs Accepted

- One extra layer between call site and Mantine.
- Wrapper props are intentionally narrower than Mantine's; a needed prop must be added to the
  wrapper rather than reaching past it.

---

## Impact

### Documentation Affected

- None beyond this document.

### Architecture Affected

- `packages/ui/package.json` — add `@mantine/core` peer + dev dependency
- `packages/ui/src/index.ts` — new export surface
- **new** `packages/ui/src/components/*` — the wrappers below
- `apps/web/src/components/**` — switched to `@brkroutnxdle/ui` imports

---

## `packages/ui` Public API

### Retained (existing exports)

| Export | Status | Change |
|---|---|---|
| `Button` | kept | Now wraps Mantine `Button`; `ButtonVariant`/`ButtonSize` re-exported unchanged so existing imports still compile |
| `Modal` | kept | Now wraps Mantine `Modal`; `isOpen`/`onClose` props unchanged |
| `FAB` | kept | Now wraps Mantine `ActionIcon` with `position: fixed` |
| `BottomSheet` | kept | Now wraps Mantine `Drawer` positioned bottom on small screens |

### New — brand primitives

| Export | Wraps | Purpose / notes |
|---|---|---|
| `IconButton` | `ActionIcon` | `label` prop **required** and forwarded to `aria-label` — prevents the current class of unnamed-icon a11y bug in `Header` |
| `SegmentedControl` | `SegmentedControl` | `label` prop required → `aria-label`; used for Day/Week/Month |
| `Badge` | `Badge` | Status pill (unsaved changes, business hours) |
| `Alert` | `Alert` | Success / error banners in `StatusBar`; `title` and children are caller-supplied text |
| `StatusMessage` | `Alert` + `role="status"` | Fixed accessibility contract for the banner region |
| `Card` | `Paper` | Section container for panels |
| `TextField` | `TextInput` | Label + description + error, wired for `getByLabel` |
| `NumberField` | `NumberInput` | Settings numeric inputs |
| `SelectField` | `Select` | Settings dropdowns |
| `ToggleField` | `Switch` | Settings booleans |
| `CheckboxGroup` | `Checkbox` + `Group` | Settings "Work Days" fieldset — **must render labels as one text node** |
| `Tooltip` | `Tooltip` | Icon affordances |
| `Spinner` | `Loader` | Button loading states |
| `Kbd` | `Kbd` | Keyboard shortcut hints (follow-up) |
| `ThemeToggle` | `ActionIcon` + `useMantineColorScheme` | Light/dark switch; `aria-label="Toggle color scheme"` |

### Re-exported theme utilities

`mantineTheme`, `brkTokens` (the token name map, so app code never hardcodes a hex), and
`BrkroutnxdleProvider` (composes `MantineProvider` + `ModalsProvider` + `Notifications`).

> **Placement correction:** `BrkroutnxdleProvider` lives in `packages/ui`, but the *stylesheet
> imports* (`@mantine/core/styles.css`, `@mantine/notifications/styles.css`) stay in
> `apps/web/src/main.tsx`, because `packages/ui` has no CSS pipeline. Documented here to
> prevent an implementer from putting an `import "@mantine/core/styles.css"` inside the package.

### `apps/web`-only composites (deliberately not in `packages/ui`)

| Component | File | Why app-local |
|---|---|---|
| `CalendarToolbar` | `src/components/calendar/CalendarToolbar.tsx` | Depends on `ViewType`, `Settings`, calendar callbacks |
| `EventContent` | `src/components/calendar/EventContent.tsx` | Depends on `CalendarEvent` |
| `StatusBar` | `src/components/StatusBar.tsx` | Depends on query/save state machines |
| `TaskPanel`, `BlockPanel`, `TaskModal`, `BlockModal`, `PreviewToolbar`, `SaveButton`, `CalculateWeekButton`, `RefreshButton`, `AllocationFailurePanel`, `DeleteEventDialog`, `ConflictDialog`, `TokenRetryModal`, `CreateActionModal`, `DevModePanel`, `DevModeBadge` | `src/features/**`, `src/components/**` | Domain-specific; wrap `packages/ui` primitives, do not move |

---

## Feature Component Mapping

| Existing component | Current markup | Target | Copy constraint |
|---|---|---|---|
| `TaskPanel` | `<div>` + `<span>` + `<button>` + list | `Card` + `Text` + `Button` + `ScrollArea` | **`Tasks ({tasks.length})` must be one contiguous text node** (TASK-02:56,80,115,167) |
| `TaskModal` | `Modal` + `TextField`s | `packages/ui` `Modal` + `TextField` + `NumberField` | `"New Task"` / `"Edit Task"` heading **verbatim** (TASK-02:64,93,173,177) — the conditional `isEditing ? … : …` must remain a single node |
| `BlockPanel` / `BlockModal` | hand-rolled | `Card` / `Modal` + fields | No E2E assertions; free to restyle |
| `PreviewToolbar` | hand-rolled | `Group` + `Button` + `Badge` | `"Preview: {n} events"` **verbatim** (TASK-01:177) — single node, and must not be duplicated |
| `SaveButton` | hand-rolled + inline span | `Button` with `Spinner` + progress | No text assertion; keep label |
| `CalculateWeekButton` | hand-rolled | `Button` | No assertion |
| `RefreshButton` | hand-rolled | `Button` + `Spinner` | No assertion |
| `AllocationFailurePanel` | hand-rolled | `Card` + `Alert` | No assertion |
| `DeleteEventDialog` | hand-rolled | `packages/ui` `Modal` | `"Delete Event"` copy preserved |
| `ConflictDialog` | hand-rolled | `packages/ui` `Modal` | `"Discard Preview — Start fresh"` **verbatim** (currently `ConflictDialog.tsx:80`); must stay mutually exclusive with `PreviewToolbar` |
| `TokenRetryModal` | hand-rolled | `packages/ui` `Modal` | No assertion |
| `CreateActionModal` | hand-rolled | `packages/ui` `Modal` | `"Create in this slot"` **verbatim** (TASK-01:160,171; TASK-02:146,157) |
| `SettingsModal` | hand-rolled | `packages/ui` `Modal` + form fields | Field labels (`"Available Start Hour"`, `"Work Days"`, `"Timezone"`, …) preserved |
| `DevModePanel` / `DevModeBadge` | hand-rolled | `Modal` / `ActionIcon` | DEV-only, unrestricted |
| `ErrorFallback` | inline styles | `Result` + `Button` | `"Oops!"` preserved |
| `LoginPage` | inline styles | `Card` + `TextField` + `Button` | No E2E assertion (tests use dev-mode bypass) |
| `App.tsx` loading state | bare `<div>Loading...</div>` | `Center` + `Spinner` | No assertion; `"Loading calendar events..."` lives in `StatusBar`, different string |

---

## Risks

| Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|
| `packages/ui` gains a Mantine peer that pnpm resolves twice | Medium | Medium | Declare as `peerDependency` **and** `devDependency`; verify a single `node_modules/.pnpm/@mantine+core*` entry after install |
| A stylesheet import lands inside `packages/ui` | Medium | High | Explicitly documented above; `tsc`-only build means a `.css` import would not even compile — build failure catches it |
| `Tasks (N)` split across elements → strict-mode failure | Medium | High | Single text node; listed in the copy-constraint column and the E2E register |
| Narrowing Mantine props to wrapper props blocks a needed feature | High | Low | Add the prop to the wrapper; do not reach past it |
| Mantine `Modal` portal moves focus, affecting the drag-based create flow | Low | Medium | QA verifies TASK-02:157 (`"Create in this slot"` hidden) still passes with focus restoration |

---

## Open Questions

- Should `Button`/`Modal` keep accepting their current inline `style` prop? Default: yes, via
  `style` pass-through, to avoid touching every call site in one step; the prop is documented
  as deprecated.