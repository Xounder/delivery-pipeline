# Risk Register — UI/UX Modernization

Complete merged register. Design-source risks come from `.opencode/plan/ui-ux-modernization/design-docs/design-07-risks-and-e2e-register.md`;
validation risks were discovered by checking the approved design against the actual repository on
2026-10-03.

Legend for **Source**: `D` = design doc, `V` = codebase validation, `D+V` = both.

---

## Critical and High Risks

| ID | Risk | L | I | Mitigation | Constraint | Phase | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `R-01` | Mantine `SegmentedControl` emits no `<button>`; Mantine `Select`/`NumberInput` emit no native `<select>` → `TASK-02` locators (`button:has-text("Day"/"Week")`, `selectOption("#task-duration")`, `label:has-text("Duration (minutes)") select`, `fill("#task-name")`) fail | High | Critical | Build the view switcher from `Button`/`UnstyledButton` with segmented styling; keep native `<select>`/`<input>` in `TaskModal` + `CreateActionModal` | `B1`, `B2` | 4, 5, 6 | V |
| `R-02` | Two mounted sidebars → duplicate `<aside>` and duplicate `Tasks (N)` panel; a `keepMounted` dialog leaves hidden "preview" text so `text=Preview` is not unique | High | Critical | Single-sidebar invariant at every viewport; `ConflictDialog` unmounts when closed; prefer a plain `<aside>` wrapper | `M7`, `M8` | 4, 6-B | V |
| `R-03` | `eventContent` replaces FullCalendar's internal event DOM, killing every `.fc-event-title` / `.fc-event-time` rule; renderer may miss `dayGridMonth` or an event kind | High | High | Emit project-owned classes and update selectors in the same change; cover dayGrid + timeGrid × task/block/generated/preview/external; keep the title inside `.fc-event`; `stopPropagation` on the delete button | `M5` | 5 | V |
| `R-04` | AppShell defaults to `min-height: 100dvh`, removing the definite parent height the calendar's `height="100%"` depends on → calendar grows unbounded | High | High | Explicit bounded height on the AppShell root (`100dvh` + `100vh` fallback); `min-height: 0` + `overflow: auto` on the scroller | `M3` | 4 | V |
| `R-05` | `packages/ui` is `tsc`-only; `apps/web` consumes `dist`. A targeted web build silently uses stale declarations | High | High | Always `build packages/ui` before `build apps/web`; never edit `dist` by hand | `P-1` | 3–7 | V |
| `R-06` | FullCalendar implements `businessHours` as inverse background events; `timeGridWeek` starts Sunday and the E2E slot is Sunday `14:00` → it renders `.fc-non-business` (the tinted class) and may intercept clicks | Medium | High | **Defer the `businessHours` prop entirely**; deliver workday/weekend tint via CSS on existing day-column classes. If ever enabled, must cover all 7 days | `B3` | 5 | V |
| `R-07` | Design maps `sm` → `36em`; Mantine's `sm` is `48em`. Breakpoints shift 12em, so the navbar/Drawer collapse fires at the wrong width and can break the single-`aside` invariant | Medium | High | Use Mantine names or override `breakpoints` in `createTheme`; keep PostCSS media queries numerically identical | `M4` | 2, 4 | V |
| `R-21` | Mantine modals portal to `document.body` with their own z-index scale → visibility assertions or the `aside` locator may break; custom `--z-*` tokens collide with Mantine's | Medium | High | Keep modal content inside the AppShell subtree; reconcile `--z-*` with Mantine's scale (modal ~200, drawer ~100, notifications ~300) | `R-20` mitigation | 4, 6 | V |

---

## Medium Risks

| ID | Risk | L | I | Mitigation | Constraint | Phase | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `R-08` | `#3CC4BD` dark event fill with white text ≈ 2.1:1 contrast — fails WCAG AA | High | Medium | Dark text on the light teal, or a darker teal fill with white text; contrast-check the whole event + brand palette in both tiers | `B4` | 2, 5 | V |
| `R-09` | FullCalendar injects CSS at runtime after the app stylesheet; design selectors use `.brk-calendar` while the DOM has `.brk-calendar-google`; stylesheet already has 17 `!important` blocks | High | Medium | Scope to the real wrapper class; theme via CSS custom properties (they inherit and win regardless of injection order); stop adding `!important` | `M1`, `M6` | 5 | V |
| `R-10` | `apps/web` is `"type": "module"` → `postcss.config.js` fails to load; wrong import order inverts overrides; `design-tokens.css` is loaded twice (`main.tsx` + `global.css` line 1); reset lives in the wrong file | Medium | Medium | `.cjs` PostCSS config; import Mantine styles first; remove the duplicate token import; move the reset to `global.css` | `P-3`, `M2`, `M13` | 1, 2 | V |
| `R-11` | `reuseExistingServer: true` → Playwright can attach to a stale dev server and run assertions against an old bundle; `fullyParallel: false`, `retries: 1` amplify the blast radius | High | Medium | Ensure the dev port is free before every run; treat a run on a reused server as invalid | `M14` | all | V |
| `R-12` | `apps/web/tsconfig.json` excludes `e2e`, so `build apps/web` cannot detect selector drift caused by `B1`/`B2`/`M5`/`M7` | High | Medium | `test apps/web` is the only guard; never present a green build as spec coverage | `M12` | all | V |
| `R-13` | `DashboardPage.tsx` is 567 lines (over the 400-line rule) and is required by phases 4, 5 and 6 → write collisions and further growth | High | Medium | Mandatory split into prop-stable children in Phase 4; single named owner for the whole delivery; never two concurrent writers | `M9`, `P-8`, `P-9` | 4, 5, 6 | V |
| `R-16` | Five status regions must fold into the StatusBar (design text alternates four/five); Header vs. CalendarToolbar ownership unresolved → duplicated or orphaned date-nav / view switcher | Medium | Medium | Five is authoritative; CalendarToolbar owns date-step + view switcher, Header owns brand/title/`Today`; Header must not also render them | `M15` | 4 | V |
| `R-22` | `AppShell.Navbar component="aside"` polymorphic typing is unverified; if wrong the build fails late | Low | Medium | Prefer the plain `<aside>` wrapper — guaranteed to compile and satisfies the locator identically | `M7` | 4 | V |
| `R-23` | New visible copy containing `Preview` or `Tasks (` breaks the uniqueness assertions | Medium | Medium | Zero new copy with those strings; `PreviewToolbar` remains the sole owner of the visible `Preview` label | `M8` | 6 | V |
| `R-24` | No theme preference, toggle, or `MantineProvider` exists → the dark tier may be unreachable | Medium | Medium | Choose and wire the colour-scheme source explicitly in Phase 2 | `B4` | 2 | V |
| `R-25` | Three Phase 6 lanes validated individually may still merge into a broken tree (provider reverts, `packages/ui` API drift, cascade regressions) | Medium | High | Full validation gate per lane **plus** a mandatory consolidated post-merge run before Phase 7 | `P-1` | 6, 7 | V |

---

## Low Risks

| ID | Risk | L | I | Mitigation | Constraint | Phase | Source |
| --- | --- | --- | --- | --- | --- | --- | --- |
| `R-14` | design-06 omits `TaskCard.tsx`, `BlockCard.tsx`, `CompletionAction.tsx`; misplaces `TokenRetryModal.tsx` / `RefreshButton.tsx` (they are in `features/completion/`, not `components/`) | High | Low | Treat `file-impact.md` as the authoritative inventory | `M10`, `M18` | 6 | V |
| `R-15` | `SettingsModal` builds 400+ timezone options via `Intl.supportedValuesOf`; Mantine `Select` is not virtualized → slow dropdown | Medium | Low | Keep the native `<select>` or add search filtering | `M11` | 6-C | V |
| `R-17` | FullCalendar's timegrid already applies sticky header positioning in JS; custom sticky CSS causes jitter or detached headers | Medium | Low | Observe built-in behaviour first; scope any custom CSS away from `.fc-sticky` | `M16` | 5 | V |
| `R-18` | Calendar CSS (181 lines, 17 `!important`) and `DashboardPage.tsx` (567 lines) breach the 400-line rule once re-skinned | High | Low | Plan the stylesheet split in Phase 5; audit all files in Phase 7 | `M17`, `M9`, `P-9` | 5, 7 | V |
| `R-19` | Four Mantine packages + Tabler icons grow the 524.63 kB / 156.62 kB baseline; no bundle-budget script exists | Medium | Low | Record the delta in Phase 7; enforcement is impossible without a new script, which must not be added | — | 7 | V |
| `R-20` | Custom `--z-*` tokens duplicate Mantine's internal z-index scale | Medium | Medium | Align token values with Mantine's cube; do not layer a second scale | `M13` | 2, 4 | V |

---

## Design-Source Risks Retained from design-07

These were already identified by the Solution Designer and remain valid. They are covered by the
constraints above rather than restated as new risks.

| Design risk | Addressed by |
| --- | --- |
| E2E selector/copy breakage from the migration | `B1`, `B2`, `M5`, `M7`, `M8`, `R-01`, `R-02`, `R-23` |
| FullCalendar DOM structure must be preserved | `M5`, `R-03` |
| Dark-mode contrast regression | `B4`, `R-08`, `R-24` |
| Calendar height/scroll regression under AppShell | `M3`, `R-04` |
| Token duplication between old and new layers | `M2`, `M13`, `R-10` |
| Bundle growth | `R-19` |
| Missing lint gate | `P-4` |
| Phase 6 file-list gaps | `M10`, `M18`, `R-14` |
| Parallel-lane write collisions | `M9`, `P-7`, `P-8`, `R-13`, `R-25` |

---

## Register Summary

| Severity | Count | IDs |
| --- | --- | --- |
| Critical impact | 2 | `R-01`, `R-02` |
| High impact | 6 | `R-03`, `R-04`, `R-05`, `R-06`, `R-07`, `R-21`, `R-25` |
| Medium impact | 9 | `R-08` – `R-13`, `R-16`, `R-20`, `R-22` – `R-24` |
| Low impact | 6 | `R-14`, `R-15`, `R-17`, `R-18`, `R-19` |
| **Total tracked** | **25** | `R-01` … `R-25` |

Every risk maps to at least one constraint in [`constraints.md`](./constraints.md), and every
constraint maps to at least one risk.
