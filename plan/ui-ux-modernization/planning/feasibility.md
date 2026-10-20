# Feasibility Analysis — UI/UX Modernization

Design: approved Option C. Validation performed against the actual repository on 2026-10-03.
Verdict: **FEASIBLE WITH MANDATORY CORRECTIONS**.

Four design assumptions are provably incompatible with the current code or with the immutable
E2E suite. They are recorded below with stable IDs and must be treated as hard constraints by the
Tech Lead when writing task files.

---

## Baseline Validation

| Check | Command | Result |
| --- | --- | --- |
| Build / typecheck | `run-package-command build apps/web` | **PASS** — `tsc && vite build`, 158 modules |
| JS bundle | — | 524.63 kB raw / 156.62 kB gzip |
| CSS bundle | — | 6.40 kB raw / 1.75 kB gzip |
| Lint | — | **Not available** — `apps/web/package.json` has no `lint` script |

The design's stated baseline matches the measured baseline exactly, so the design's performance
figures can be used as the comparison point.

---

## Blocking Corrections (B1–B4)

These four are **not optional**. Implementing the design literally breaks `TASK-02.spec.ts` or
regresses layout/legibility. Each appears as a risk in [`risks.md`](./risks.md).

### B1 — Do NOT use Mantine `SegmentedControl` for the Day/Week/Month view switcher

**Constraint.** The calendar view switcher must be built from elements that render a real
`<button>`. Mantine `Button` / `UnstyledButton` (visually grouped as a segmented control via
custom CSS on a shared container) is the approved replacement. `SegmentedControl` must not appear
anywhere in the implementation.

**Why (verified).** `SegmentedControl@8.3.18` renders `<input type="radio">` elements plus
`<label>` wrappers. It emits **no `<button>` element at all**.

**Breaks.** `apps/web/e2e/TASK-02.spec.ts` asserts:

- `button:has-text("Day")`
- `button:has-text("Week")`
- and clicks them to switch views.

**Additional requirements.**

- The three view buttons must remain **visible text buttons** labelled exactly `Day`, `Week`,
  `Month` (today they are native `<button>`s in `components/Header.tsx`).
- `aria-pressed` / `aria-current` state may be added for accessibility; the accessible name and
  element type must stay a `<button>` with the exact visible label.
- Grouping must be keyboard-navigable in the same tab-stop order as today.

---

### B2 — Native `<select>` and `<input type="number">` are load-bearing in two E2E-asserted forms

**Constraint.** `apps/web/src/features/tasks/TaskModal.tsx` and
`apps/web/src/features/shared/CreateActionModal.tsx` must keep **native** `<select>` elements
(and native `<input>` for text/number fields). Mantine `Select`, `MultiSelect`, `Combobox`,
`NumberInput` and `TextInput` must **not** be used in these two files.

**Why (verified).**

- `TaskModal.tsx` nests a native `<select>` **inside** its `<label>` (the "Duration (minutes)"
  control), so Playwright resolves `label:has-text("Duration (minutes)") select` only when the
  select is a descendant of that label.
- `CreateActionModal.tsx` renders `<select id="task-duration">` with a separately associated
  `<label htmlFor="task-duration">`.

**Breaks.**

- `page.selectOption("#task-duration", "30")` — requires a native `<select>`; a Mantine combobox
  exposes an `<input role="combobox">` instead.
- `label:has-text("Duration (minutes)") select` — requires the nested-label DOM shape.
- `#task-name` must remain a native input (`page.fill("#task-name", ...)`).
- `button[type="submit"]:has-text("Create Task")` and `:has-text("Create")` must stay real
  submit buttons.

**Scope of the exemption.** Mantine inputs **are** permitted in surfaces that no E2E spec
asserts — e.g. `components/SettingsModal.tsx`. The design's blanket `NumberField`/`SelectField`
→ Mantine mapping is therefore valid for `SettingsModal` only, and invalid for `TaskModal` /
`CreateActionModal`. See `M2` in `impact-analysis.md` for the resulting selector-literal rule.

---

### B3 — Defer the FullCalendar `businessHours` prop; deliver workday tint via CSS

**Constraint.** Do **not** pass `businessHours` to the FullCalendar instance in
`apps/web/src/components/CalendarView.tsx`. Workday/weekend tinting must be delivered by CSS
scoped to the existing FullCalendar day-column classes. If a `businessHours` prop is ever
introduced later, it may only cover **all seven days** — never the Monday–Friday subset the
current settings imply.

**Why (verified).**

1. FullCalendar `6.1.20` implements `businessHours` as **inverse background events**: it renders
   background events for every *non*-business time range and marks them `.fc-non-business`.
2. `timeGridWeek` starts on **Sunday** by default. `apps/web/e2e/TASK-02.spec.ts` locates the
   time slot with `.first()`, i.e. the **Sunday** `14:00` slot, and asserts its label and that a
   click opens the "Create in this slot" modal.
3. Current settings are `workStartHour: 8`, `workEndHour: 18`, `workDays: [1,2,3,4,5]`
   (Sunday = 0). Sunday is therefore a non-business day.

**Breaks / degrades.** The Sunday `14:00` slot the E2E suite depends on would be shaded with
`.fc-non-business` — exactly the class the proposed tint stylesheet paints — and the assertion
`locator(".fc-timegrid-slot").filter({ hasText: "14:00" }).first()` would resolve to a background
element instead of a real slot.

**Additional risk retained.** Those background events are separate DOM elements layered in the
same grid; whether they intercept pointer events for slot clicks is **not yet verified**. This is
tracked as risk `R-06` and is a second, independent reason to defer the prop.

---

### B4 — Fix dark-mode event contrast; `#3CC4BD` with white text is not acceptable

**Constraint.** The dark-mode colour for generated/preview events must be changed so that the
event title and time text meet **at least 4.5:1** contrast against their background. Do not ship
`#3CC4BD` as an event fill with white (`--brk-text-on-brand`) text.

**Why (verified).** `#3CC4BD` (design's `--brk-event-generated`, dark tier) against white text
yields approximately **2.1:1** — far below WCAG AA.

**Accepted resolutions (either one).**

- Keep the light teal fill and switch event text to a dark neutral (e.g. the dark-tier
  `--brk-text-primary`), or
- Use a darker teal fill from the approved teal tuple (the design's darker shades) with white text.

**Scope.** This is not limited to one token. The whole event palette in both light and dark tiers
must be contrast-checked: task, block, generated, preview and external events, plus the primary
brand fill used by `Save to Calendar` / `Create Task`. Any fill below 4.5:1 against its text must
be corrected before Phase 5 exits. Tracked as risk `R-08`.

---

## Per-Phase Feasibility Verdicts

### Phase 1 — Dependencies, lockfile, PostCSS pipeline

**Verdict: FEASIBLE.** Entry: none. This phase unblocks everything else.

Entry conditions: none.
Mandatory constraints applied: `.cjs` PostCSS config requirement (below), `pnpm install`.

Tasks implied:

1. Add to `apps/web/package.json`: `@mantine/core@8.3.18`, `@mantine/hooks@8.3.18`,
   `@mantine/notifications@8.3.18`, `@mantine/modals@8.3.18`, `@tabler/icons-react`.
   (`@mantine/hooks` must match the core version exactly — it is a peer dependency.)
2. Add dev dependencies for the PostCSS pipeline (`postcss`, `postcss-preset-mantine`,
   `postcss-simple-vars`).
3. **Run `pnpm install` to regenerate `pnpm-lock.yaml`.** Without this the new imports will not
   resolve and the build fails. The lockfile diff is a required deliverable of this phase.
4. Create `apps/web/postcss.config.cjs`.

**`.cjs` PostCSS config requirement.** `apps/web/package.json` declares `"type": "module"`, so a
`postcss.config.js` would be parsed as ESM and the CommonJS-style PostCSS config object would
fail to load. The config file **must** be named `postcss.config.cjs`.

Exit gate: `pnpm install` completes without peer-dependency errors, and
`run-package-command build apps/web` PASSES (the PostCSS pipeline must not break the existing
build before any component is migrated).

---

### Phase 2 — Design tokens, theme, Mantine providers, dark mode

**Verdict: FEASIBLE.** Entry: Phase 1 complete.

Mandatory constraints applied: `M2` (reset location + duplicate import), `M4` (breakpoint names),
`B4` (contrast), `M13` (PostCSS + stylesheet import order).

Tasks implied:

1. Add `MantineProvider` (with `ModalsProvider` and `Notifications` if those packages are used) to
   the provider tree in `apps/web/src/main.tsx`. There is currently **no** Mantine provider.
   Provider order matters: `MantineProvider` must wrap everything that renders Mantine components,
   and `ErrorBoundary` may stay outermost.
2. Move the global `* { box-sizing }` / `body` reset **out of `design-tokens.css`** into
   `global.css`, which is where the design places it — it is currently in the wrong file.
3. Remove the duplicate token load: `global.css` line 1 is `@import "./design-tokens.css";` **and**
   `main.tsx` imports `./styles/design-tokens.css` directly. Exactly one of the two must survive.
4. Establish the colour-scheme source on `MantineProvider` (auto / light / dark) so
   `data-mantine-color-scheme` is actually set; there is no theme preference or toggle in the
   codebase today, so this must be introduced deliberately.
5. Use Mantine's breakpoint names or override `breakpoints` in `createTheme` — see `M4`.
   Keep the PostCSS media queries in `global.css` numerically identical to whatever Mantine theme
   breakpoints end up being.
6. Import `@mantine/core/styles.css` in `main.tsx` **before** the app stylesheets, so app tokens
   and overrides win the cascade.

Exit gate: `run-package-command build apps/web` PASSES. Dark and light tiers both render; the
duplicate token load is gone.

---

### Phase 3 — `packages/ui` Mantine wrapper components

**Verdict: FEASIBLE.** Entry: Phase 2 complete.

Mandatory constraints applied: wrapper API preservation, build-order requirement.

Tasks implied:

1. Rebuild the four existing wrappers in `packages/ui/src/components/` (`Button`, `Modal`, `FAB`,
   `BottomSheet`) on Mantine, keeping their **current public props**. `packages/ui/src/index.ts`
   exports `Button` with `ButtonVariant`/`ButtonSize` and `Modal` taking `isOpen`/`onClose`;
   Mantine uses `opened`/`onClose` and `variant`/`size`. The wrappers must map, not leak, these
   names — otherwise the two existing consumers break.
2. Add `@mantine/core` as a **peer** dependency (and a dev dependency for typecheck) in
   `packages/ui/package.json`, version-pinned to `8.3.18`. Do not add it as a hard runtime
   dependency.
3. Only two files in the app consume this package today: `components/ErrorFallback.tsx`
   (`Button`) and `features/completion/TokenRetryModal.tsx` (`Modal`). `FAB` and `BottomSheet`
   have **no consumers** — migrating them is low-value and cannot be validated by any test;
   treat as optional cleanup.
4. `packages/ui` is `tsc`-only and has **no CSS pipeline**. Do not add Mantine stylesheet imports
   to `packages/ui` — styles belong in `apps/web`.

**Build-order gate (critical).** `apps/web` resolves `@brkroutnxdle/ui` through
`packages/ui/dist`. A targeted `apps/web` build does **not** build the package. Therefore:

- `run-package-command build packages/ui` **must** run before `run-package-command build apps/web`
  in every subsequent phase. Skipping it makes `apps/web` typecheck against **stale declarations**
  and report confusing unrelated errors. Tracked as risk `R-05`.

Exit gate: `build packages/ui` PASSES, then `build apps/web` PASSES.

---

### Phase 4 — App shell (AppShell, Header, Sidebar, StatusBar)

**Verdict: FEASIBLE.** Entry: Phase 3 complete.

Mandatory constraints applied: `M3` (bounded height), `M7` (exactly one `<aside>`),
`M15` (five status regions + Header/toolbar ownership), `B1` (view switcher is buttons),
`M9` (mandatory `DashboardPage.tsx` split).

Tasks implied:

1. Migrate `components/MainLayout.tsx` (57 lines) to `AppShell` / `AppShell.Header` /
   `AppShell.Navbar` / `AppShell.Main` with the corresponding config props.
2. **Bound the AppShell root height explicitly** — see `M3`. Without this the calendar's
   `height="100%"` chain breaks.
3. `components/Header.tsx` (113 lines): keep `Today`, `Day`, `Week`, `Month` as real buttons
   (`B1`). The previous/next date-navigation buttons must keep accessible names.
4. `components/Sidebar.tsx` (109 lines): exactly one `<aside>` in the document at any viewport
   (`M7`).
5. `pages/DashboardPage.tsx` (567 lines): fold **all five** conditional status regions into the
   new StatusBar (`M15`) and perform the **mandatory split** (`M9`).
6. Apply the Header / CalendarToolbar ownership split defined in `M15`.

Exit gate: full gate — `build packages/ui` → `build apps/web` → `test apps/web` — PASSES.
The E2E assertions covering this phase are the single `<aside>`, single `Tasks (N)` panel, and the
Day/Week button switches.

---

### Phase 5 — Calendar (tokens, `eventContent`, grid, weekend tint)

**Verdict: FEASIBLE.** Entry: Phase 4 complete.

Mandatory constraints applied: `B3`, `B4`, `M1`, `M5`, `M6`, `M16`.

Tasks implied:

1. Rewrite `apps/web/src/styles/CalendarStyles.css` (181 lines, 17 `!important` blocks) against the
   **real** wrapper class `.brk-calendar-google` — not `.brk-calendar` (`M1`).
2. Replace the `eventDidMount` DOM-mutation block in `components/CalendarView.tsx` (373 lines)
   with an `eventContent` React renderer (`M5`). This is the highest-risk single edit in the
   delivery.
3. Do **not** add `businessHours` (`B3`).
4. Apply the corrected dark event palette (`B4`).
5. Resolve the sticky-header conflict with FullCalendar's built-in behaviour (`M16`).
6. Expect the calendar stylesheet to grow past 400 lines — split it into
   `CalendarStyles.css` (layout/grid) plus focused partials (tokens, events, dark tier) and
   import them from `main.tsx`.

Exit gate: full gate PASSES. Asserted E2E DOM must survive: `.fc-timegrid-slots`,
`.fc-timegrid-body`, `.fc-event` (with the title **inside** `.fc-event`), and the
`.fc-timegrid-slot` whose text contains `14:00`.

---

### Phase 6 — Feature component migration

**Verdict: FEASIBLE — largest phase.** Entry: Phase 5 complete. **3 file-disjoint parallel lanes.**

Mandatory constraints applied: `B2` (native controls in `TaskModal` + `CreateActionModal`),
`M8` (`ConflictDialog` unmounts when closed), `M10` (design-06 file list was incomplete),
`M11` (`SettingsModal` select volume), `M18` (two design-06 paths are wrong).

Lane split, ownership rules and forbidden-file lists: see
[`dispatch-plan.md`](./dispatch-plan.md#phase-6--definitive-lane-split).

Lane sizes: 6-A ≈ 758 lines, 6-B ≈ 865 lines, 6-C ≈ 633 lines.

Exit gate: full gate PASSES **three times** — once per lane — plus a final consolidated run after
all three merge.

---

### Phase 7 — Dead-CSS cleanup, bundle check, final validation

**Verdict: FEASIBLE.** Entry: Phase 6 complete (all lanes merged).

Mandatory constraints applied: `M12` (Playwright excludes `e2e`), stale-server rule below.

Tasks implied:

1. Remove the dead responsive classes in `apps/web/src/styles/global.css` (42 lines) that no
   longer have consumers after the AppShell migration.
2. Compare the production bundle against the measured baseline (524.63 kB / 156.62 kB gzip) and
   record the delta. There is **no bundle-size budget script**, so this is a reported metric, not
   an enforced gate.
3. Confirm no source file exceeds 400 lines.

Exit gate: full gate PASSES, on a **fresh** dev server.

---

## Validation Gate — Mandatory Rules

1. **No lint gate exists or may be invented.** `apps/web/package.json` has no `lint` script. Do not
   add one and do not report a lint result. Validation is build + Playwright only.
2. **Order is always:** `build packages/ui` → `build apps/web` → `test apps/web`.
3. **Stale dev server invalidates a Playwright run.** `playwright.config.ts` sets
   `reuseExistingServer: true`. Before each `test apps/web`, confirm the dev port is not already
   served by a process from an earlier phase; otherwise the run may exercise stale bundles and be
   reported green when it is not (or fail for reasons already fixed). Tracked as risk `R-11`.
4. **`build apps/web` does not typecheck the E2E specs** (`apps/web/tsconfig.json` excludes `e2e`).
   Any change that alters the selectors the specs rely on can only be caught by `test apps/web`.
   There is no substitute check.
5. **Never modify `apps/web/e2e/TASK-01.spec.ts` or `TASK-02.spec.ts`.** If an assertion fails,
   the implementation is wrong and must be adjusted until it passes. Pre-existing tests are
   verified acceptance criteria, not obstacles.
6. **Never modify project configuration** (tsconfig, eslint, vitest, `package.json` scripts) to
   make an implementation or test pass.

---

## Decision Summary

### Selected Approach

Approved Option C, executed as one delivery in 7 phases, with `B1`–`B4` applied as hard
constraints and `M1`–`M18` recorded in [`impact-analysis.md`](./impact-analysis.md).

### Reasoning

The library selection, three-tier token architecture and AppShell shell direction were all
validated as implementable against the actual repository. The four failures found are all
*implementation-detail* conflicts — an element-type assumption, two DOM-shape assumptions, and a
colour-value assumption — not architectural disagreements. Recording them as constraints (the user
approved this option over reopening design) preserves the approved design while preventing the
three test-breaking failures.

### Expected Outcome

Phase 7 completes with: `build packages/ui` PASS, `build apps/web` PASS, `test apps/web` PASS on a
fresh dev server, no file over 400 lines, and the E2E suite unmodified and fully green.

### Open Questions

None. Deferred by explicit user decision and recorded as constraints: `B3` (`businessHours`
deferral) and `M3` / `M15` (Header vs. CalendarToolbar ownership).
