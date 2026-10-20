# Risk Assessment — UI/UX Modernization

Merged register: the risks recorded in the approved design docs (`design-07`) plus the risks
discovered while validating the design against the actual repository.

Full table with likelihood, impact, mitigation and source: [`risk-register.md`](./risk-register.md).
Constraint IDs (`B1`–`B4`, `M1`–`M18`): [`constraints.md`](./constraints.md).

---

## Risk Summary

| ID | Risk | Likelihood | Impact | Mitigation |
| --- | --- | --- | --- | --- |
| `R-01` | E2E selector breakage from Mantine component swaps (`B1`, `B2`) | **High** | **Critical** | Preserve real `<button>`s and native form controls; E2E suite is the gate |
| `R-02` | Duplicate `<aside>` / `TaskPanel` / hidden `Preview` text (`M7`, `M8`) | **High** | **Critical** | Single-instance mount invariant; `keepMounted={false}`; prefer `<aside>` wrapper |
| `R-03` | `eventContent` rewrite breaks event DOM or styling (`M5`) | **High** | **High** | Preserve `.fc-event` + inner title; project-owned classes; cover dayGrid + all kinds |
| `R-04` | AppShell `min-height` default breaks calendar height (`M3`) | **High** | **High** | Explicit bounded height + `min-height: 0` on the scroll container |
| `R-05` | Stale `packages/ui/dist` produces misleading typecheck errors | **High** | **High** | Always `build packages/ui` before `build apps/web` (`P-1`) |
| `R-06` | FullCalendar `businessHours` shades/intercepts the asserted Sunday `14:00` slot (`B3`) | Medium | **High** | **Defer the prop** — deliver workday tint via CSS |
| `R-07` | Breakpoint naming mismatch breaks the responsive single-`aside` contract (`M4`) | Medium | **High** | Align Mantine theme breakpoints with PostCSS media queries |
| `R-08` | Dark-mode contrast failures (`B4`) | **High** | Medium | Contrast-check the whole event + brand palette before Phase 5 exits |
| `R-09` | FullCalendar runtime CSS beats app stylesheet (`M1`, `M6`) | **High** | Medium | Scope to the real wrapper class; theme via custom properties |
| `R-10` | PostCSS config failure / wrong import order / duplicate token load (`M2`, `M13`) | Medium | Medium | `.cjs` config; Mantine styles first; single token import |
| `R-11` | Stale reused dev server yields a false Playwright result (`M14`) | **High** | Medium | Ensure the dev port is free before each run |
| `R-12` | Selector drift invisible to the build (`M12`) | **High** | Medium | `test apps/web` is the only guard; never treat build as spec coverage |
| `R-13` | `DashboardPage.tsx` churn across three phases (`M9`) | **High** | Medium | Single owner + early mandatory split into prop-stable children |
| `R-14` | Design-06 file list/paths wrong (`M10`, `M18`) | **High** | Low | Use [`file-impact.md`](./file-impact.md) as the authoritative inventory |
| `R-15` | `SettingsModal` 400+ option dropdown cost (`M11`) | Medium | Low | Keep native select or add search filtering |
| `R-16` | Header/CalendarToolbar duplication or orphaned controls (`M15`) | Medium | Medium | Ownership table in `impact-analysis.md`; single owner per control |
| `R-17` | Sticky-header conflict with FullCalendar's built-in JS (`M16`) | Medium | Low | Observe built-in behaviour first; scope any custom CSS |
| `R-18` | Calendar CSS / other files breach the 400-line rule (`M17`, `M9`) | **High** | Low | Plan the stylesheet split in Phase 5; audit in Phase 7 |
| `R-19` | Bundle growth with 4 Mantine packages, no budget gate | Medium | Low | Record the delta in Phase 7; no enforcement script exists |
| `R-20` | Mantine z-index scale vs. custom `--z-*` tokens | Medium | Medium | Align tokens with Mantine's (modal ~200, drawer ~100, notifications ~300) |
| `R-21` | Modal portal changes break visibility or `<aside>` locators | Medium | **High** | Modal content stays inside `AppShell`; verify Playwright visibility assertions |
| `R-22` | Unverified polymorphic typing (`AppShell.Navbar component="aside"`) | Low | Medium | Prefer the plain `<aside>` wrapper, which is guaranteed to compile |
| `R-23` | New visible copy collides with `text=Preview` / `Tasks (N)` uniqueness | Medium | Medium | Zero new copy containing `Preview` or `Tasks (`; see `R-02` |
| `R-24` | Dark mode has no existing preference/toggle to build on | Medium | Medium | Choose the colour-scheme source deliberately in Phase 2 |
| `R-25` | Phase 6 lanes merge into a tree no single lane validated | Medium | **High** | Full gate once per lane **plus** a consolidated post-merge run |

---

## Technical Risks

- **`R-01` / `R-02` — the E2E suite is the sharpest constraint in this delivery.** Nine-plus
  locators assert element type (`button`, `select`, `aside`), DOM nesting
  (`label ... select`, title inside `.fc-event`), and text uniqueness (`text=Preview`,
  `Tasks (N)`). Mantine's idiomatic components break element type and nesting by default, so the
  migration must be selective, not blanket. `apps/web/tsconfig.json` excludes `e2e`, so nothing
  but `test apps/web` catches a mistake.
- **`R-03` — the `eventContent` rewrite is the single riskiest edit.** It replaces
  `eventDidMount` DOM mutation in a 373-line component and invalidates every
  `.fc-event-title` / `.fc-event-time` rule in the stylesheet. It must also work for `dayGridMonth`,
  which the design barely discusses, and for five event kinds.
- **`R-04` — a layout regression that no unit of the design covers.** AppShell's `min-height`
  default removes the definite parent height that the calendar's `height="100%"` chain depends on.
  The symptom (calendar taller than the viewport, page scrolling instead of the calendar) is easy
  to misdiagnose as a calendar bug.
- **`R-09` / `R-10` — cascade and pipeline ordering.** FullCalendar injects styles at runtime,
  after the app stylesheet. The current stylesheet already compensates with 17 `!important`
  blocks; adding a second theming system (Mantine + custom properties) without an explicit import
  order will produce invisible-wins/losses.
- **`R-19` — bundle growth is unmeasured and unenforced.** Four Mantine packages plus Tabler icons
  will grow the 524.63 kB / 156.62 kB baseline. No budget script exists, so this can only be
  reported, not enforced.
- **`R-24` — dark mode has no foundation in the codebase.** There is no stored preference, no
  toggle, and no `MantineProvider`. The colour-scheme source must be chosen and wired in Phase 2
  or the dark tier will be unreachable.

---

## Delivery Risks

- **`R-13` / `M9` — `DashboardPage.tsx` is the coordination bottleneck.** 567 lines, over the
  project limit, and required by phases 4, 5 and 6. Without a single owner and an early split,
  the three phases will collide and the file will grow further.
- **`R-14` — the design's Phase 6 file list is incomplete and two paths are wrong.** Planning
  tasks directly from `design-06` would omit `TaskCard.tsx`, `BlockCard.tsx` and
  `CompletionAction.tsx`, and would look for `TokenRetryModal.tsx` / `RefreshButton.tsx` in
  `components/` instead of `features/completion/`.
- **`R-25` — merged-lane blind spot.** Three concurrent lanes each validated in isolation can
  still produce a broken combination (e.g. 6-C's `App.tsx` reverting Phase 2's provider additions,
  or `packages/ui` wrapper API drift between 6-B and 6-C). A consolidated post-merge run is
  mandatory.
- **`R-22` — unverified polymorphic typing.** `AppShell.Navbar component="aside"` is *probably*
  valid because Mantine components are `factory`-based, but it was not verified. The plain
  `<aside>` wrapper is a guaranteed-safe alternative with identical locator behaviour.
- **Sequencing pressure.** Phases 1–5 must not be parallelised to save time. `main.tsx`, the
  package manifests, the lockfile and the token stylesheets are all multi-phase write targets
  (see `OVERLAP-2` … `OVERLAP-8`).

---

## Operational Risks

- **`R-11` — stale dev server.** `reuseExistingServer: true` means Playwright can attach to a dev
  server started during an earlier phase and run assertions against an old bundle. A green run may
  be meaningless; a red run may already be fixed. With `fullyParallel: false` and `retries: 1`, one
  contaminated run can wrongly block a phase sign-off.
- **`R-05` — stale `packages/ui/dist`.** `packages/ui` is `tsc`-only and `apps/web` resolves it via
  `dist`. A targeted `apps/web` build does not rebuild it, so wrapper changes can appear to have
  no effect or to produce unrelated type errors.
- **`R-21` — portal and stacking changes.** Mantine modals render in portals at the document body
  with their own z-index range. Moving dialog content out of the AppShell subtree can affect
  visibility assertions and the `aside` locator. Custom `--z-*` tokens (`R-20`) must be reconciled
  with Mantine's internal scale rather than layered on top of it.
- **`R-18` — 400-line rule breaches.** The calendar stylesheet starts at 181 lines with 17
  `!important` blocks and will exceed 400 once re-skinned; `DashboardPage.tsx` is already over.
- **No lint gate exists.** Validation is build + Playwright only. Do not add a lint script to
  "complete" the gate, and do not report a lint result that was never produced.

---

## Risk Treatment Priorities

1. **`R-01` / `R-02`** — must be designed in, not tested for. Build `B1`/`B2`/`M7`/`M8` into the
   first implementation task of each affected lane.
2. **`R-03` / `R-04`** — schedule as dedicated, isolated tasks with their own full validation gate.
3. **`R-05` / `R-11`** — procedural. Encode `P-1` and `M14` as explicit steps in every task's
   validation section.
4. **`R-13`** — resolve in Phase 4 with the mandatory split; do not defer.
5. **`R-25`** — a mandatory post-merge gate before Phase 7.

---

## Decision Summary

### Selected Approach

Proceed with the approved Option C design under the 32 documented constraints, using the
7-phase sequence with a 3-lane Phase 6.

### Reasoning

The design carries 25 tracked risks after merging. Only four are critical/high-likelihood
enough to threaten the delivery outright (`R-01`, `R-02`, `R-03`, `R-04`), and all four have
deterministic mitigations that are now written as implementation constraints rather than left as
advice. The remaining risks are either procedural (mitigated by `P-1` and `M14`) or cosmetic
(mitigated by Phase 7). No risk requires reopening the design.

### Expected Outcome

Risk-adjusted delivery confidence is high provided the four blocking constraints are honoured in
the first pass of each affected phase. `R-13` and `R-05` are the two most likely sources of
schedule friction, both addressed by explicit ownership and ordering rules.

### Open Questions

None. Three items were explicitly deferred by user decision and are recorded as constraints rather
than open questions: `B3` (`businessHours` deferral, risk `R-06`), and `M3` / `M15`
(AppShell height and Header/CalendarToolbar ownership, risks `R-04` and `R-16`).
