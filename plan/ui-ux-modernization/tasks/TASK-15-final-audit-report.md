# TASK-15 final audit — QA freshness gate verified

## Scope and reproducibility

Audited the present working tree (which already contains the prior phases' uncommitted changes). `apps/web/qa/task12-browser-validation.ts` is permanent, authorized browser QA tooling: it recursively prints line counts, launches its own isolated Vite server on an ephemeral port, mocks authentication/API, opens real headless Chromium and closes the browser/server in `finally`. The consolidated Playwright run used `run-package-command test apps/web`. Its existing `playwright.config.ts` uses `webServer: { command: "pnpm dev", port: 5173, reuseExistingServer: true }`, so Playwright output alone does not certify freshness. Final QA performed a localhost HTTP probe immediately before an additional test run: `curl.exe -sS --max-time 3 http://localhost:5173/` failed with curl exit 7, "Failed to connect to localhost:5173". The immediately following Playwright invocation passed 9/9 against that port; with no responding server immediately before it, the configured Playwright webServer had to start one for this run. This is HTTP-availability evidence, not OS process inspection or an assertion that no non-HTTP process existed. In contrast, the QA runner's port-0 server is created by its invocation and closed in `finally`. No E2E specs, configs, manifests or lockfiles were changed in this task.

## Source line-count audit (P-9)

The permanent QA runner enumerated **every file** recursively under `apps/web/src` and `packages/ui/src` and printed each filename and line count. Result: **77/77 <= 400**, zero offenders. Maximum: `CalendarView.tsx` 374, `DashboardPage.tsx` 346, `useScheduleEditing.ts` 293, `TaskModal.tsx` 274; `packages/ui/src` maximum `Button.tsx` 57. The starting 567-line `DashboardPage.tsx` is now 346 (−221). Re-run `node apps/web/qa/task12-browser-validation.ts` for the complete per-file enumeration. The runner counts a trailing newline as one additional line, a conservative measurement.

## Contrast audit (B4)

Computed CSS custom properties were rendered in real Chromium (canvas pixel sampling and WCAG 2.1 linear-sRGB relative luminance), **separately for light and dark**, against the calendar's `--brk-bg-raised` and white on-brand text. Each result below is a measured fill/text and fill/background pair; thresholds are 4.5:1 and 3:1, respectively. These are opaque base-token measurements, not a claim that every overlaid event/past-event or all application text combinations have been visually checked.

| Tier | Pair (fill vs white text / calendar surface) | Text | Boundary |
| --- | --- | ---: | ---: |
| light | generated `#187E78` | 4.89 | 4.89 |
| light | external `#2E78AA` | 4.79 | 4.79 |
| light | preview `#187E78` | 4.89 | 4.89 |
| light | blocked `#BC4F30` | 4.88 | 4.88 |
| light | brand `#16706B` | 5.89 | 5.89 |
| dark | generated `#187E78` | 4.89 | 3.28 |
| dark | external `#2E78AA` | 4.79 | 3.35 |
| dark | preview `#187E78` | 4.89 | 3.28 |
| dark | blocked `#BC4F30` | 4.88 | 3.28 |
| dark | brand `#187E78` | 4.89 | 3.28 |

Corrections: prior event tints were indistinguishable from white; prior dark fills and the dark filled-button shade did **not** meet the white-text claims in comments. Event tokens now use solid fills, event text uses on-brand white, and Mantine's primary filled shade is teal[7] in both tiers. The primitive `#3CC4BD` and dark `--brk-text-brand: #3CC4BD` remain for non-white-on-brand usage; neither is used as a white-text fill in the inspected event/brand styling. Generated/completed/past treatments and arbitrary UI foreground/background combinations were not exhaustively measured. No manual human visual verification was performed.

## Structural constraint checklist

| Constraint | Evidence / status |
| --- | --- |
| No `eventDidMount` / `eventWillUnmount` | PASS — `CalendarView.tsx` has neither. |
| No `businessHours` on FullCalendar | PASS — explicit prop listing in `CalendarView.tsx` has neither. |
| Button view switcher, not SegmentedControl | PASS — `CalendarToolbar.tsx` maps views to Mantine `Button`, visible labels and `aria-pressed`; real Chromium changed Day → Month → Week successfully. |
| Native task and create-action controls | PASS — inspected native `<select>` and `<input>` in `TaskModal.tsx` and `CreateActionModal.tsx`. |
| Closed ConflictDialog unmounted | PASS by source inspection — `if (!isOpen) return null`; closed-dialog browser scenario not separately exercised. |
| Exactly one aside and Tasks (N) | PASS in real Chromium at widths **1440, 768, 576, 375**, height 900; one of each at every tested width. After awaiting FullCalendar's ResizeObserver, grid bounding sizes: 1142×618, 470×567, 502×567, 357×567. Main widths 1440, 768, 576, 375; calendar hosts 1160, 488, 576, 375; document scroll widths 1440, 768, 576, 375. Earlier constant 1142 was a stale measurement taken immediately after viewport changes, **not** persistent mobile overflow. Width and counts are automated geometry, not human visual sign-off. |
| Calendar selector scoping | PASS for calendar CSS files (`CalendarStyles.css`, `calendar-tokens.css`, `calendar-events.css`, `calendar-dark.css`): all selector lists start with `.brk-calendar-google`, optionally prefixed by dark-tier attribute; no unscoped `.fc` rule. |
| No calendar `!important` | PASS — zero across four files. |
| Mantine-first imports | PASS — `main.tsx`: core, notifications, tokens, global, base calendar, three partials. `@mantine/modals/styles.css` is not exported by installed package and must NOT be imported. |
| Single token import | PASS — only `main.tsx` imports `design-tokens.css`. |

Historical-doc correction: TASK-05, design-03 and design-07 no longer require the unsupported Mantine 8.3.18 `AccordionPanel.keepMounted`; Tasks must be expanded before dragging. TASK-03, TASK-09 and TASK-14 no longer require the unexported modals stylesheet. `MainLayout` now uses `.brk-app-shell` with `height: 100vh; height: 100dvh` in that order in `global.css`, restoring the deferred fallback while keeping `overflow: hidden` and the scroller's `minHeight: 0`/`overflow: auto`.

## Bundle delta (reported, not enforced)

| Metric | Baseline | Current | Delta |
| --- | ---: | ---: | ---: |
| JS raw | 524.63 kB | 767.36 kB | +242.73 kB (+46.3%) |
| JS gzip | 156.62 kB | 235.58 kB | +78.96 kB (+50.4%) |
| CSS raw | 6.40 kB | 213.63 kB | +207.23 kB (+3238.0%) |
| CSS gzip | 1.75 kB | 32.00 kB | +30.25 kB (+1728.6%) |
| Modules transformed | 158 | 7159 | +7001 |

Mantine core and notifications CSS are imported and compiled. The large CSS increase is consistent with their styles and merits independent review if a product budget is later established; no budget script was added.

## Final full gate (ordered)

1. `run-package-command build packages/ui`: **PASS** (`tsc`).
2. `run-package-command build apps/web`: **PASS** (`tsc && vite build`, 7159 modules; metrics above).
3. `run-package-command test apps/web`: implementation run **PASS 9/9**, 24.8s; its server freshness was not proved. Final QA repeated the ordered gate (`build packages/ui` **PASS**, `build apps/web` **PASS**, then `test apps/web` **PASS 9/9**, 30.4s), then explicitly established port-5173 HTTP unavailability with curl exit 7 and immediately reran `test apps/web`: **PASS 9/9**, 25.8s. `TASK-01.spec.ts` **5/5**, `TASK-02.spec.ts` **4/4**; real Chromium E2E rather than build-based inference. The immediate pre-run HTTP refusal plus success on the configured port provides M14 fresh-server evidence for the second QA run. Independently the owned ephemeral server ran the permanent QA suite to completion successfully.

No lint command run: TASK-15 explicitly specifies no lint gate (`P-4`).

## Outstanding acceptance / sign-off blockers

- Permanent QA **PASS** on its own freshly created ephemeral Vite server with real Chromium: login navigation to mocked OAuth, dev-token storage/reopen, settings persistence, error-boundary retry, Day/Month/Week, future-slot inline task creation and delete confirmation, sidebar task create/edit/delete, real mouse task drag to a future Day column (new event visible at 15:00), mocked API save success with POST containing the edited title, task ID and future 15:00 start (two POST operations), preview/discard, and mobile-width future slot dialog open/cancel. Four viewport checks at 1440/768/576/375 establish one aside/task panel, settled grid size and no document horizontal overflow. Contrast pairs above all pass in both tiers. The QA runner has no final success banner, but it exits zero and emits all ten tier-pair ratios after the final mobile check. Earlier drag failures were due to the app's native HTML5 task drag not being registered with FullCalendar's external drag system; `TaskPanel` now registers FullCalendar `Draggable`, `TaskCard` exposes task metadata, and `CalendarView` reads the dragged element's metadata without changing the app's scheduling rules. The initially incorrect fixed-count assertion was replaced with a pre/post count and a check of day, time, title and saved task ID.
- Automated viewport geometry, event DOM, and computed token contrast are not a human visual review or a touch-device test. Other application text colours, tinted/past overlays, and physical-device rendering have not been exhaustively audited; do not claim they were. Neither Google OAuth nor a live external API was accessed.
- **M14 resolved by final QA rerun:** Although `reuseExistingServer: true` makes Playwright output alone inconclusive, the immediate unsuccessful localhost HTTP probe before the second 9/9 run demonstrates there was no reusable responding server at port 5173. No process inspection, termination, or configuration change was needed. This matches the documented TASK-13 HTTP-probe precedent.
- `get-git-diff` returned exit 128 (`main..HEAD` unknown ref); `git diff --stat` shows extensive earlier-phase uncommitted changes, so only the files listed in the response as TASK-15 changes are claimed as this task's edits.

The gate's fresh-server condition is supported by the immediately preceding HTTP refusal and the passing rerun; it is not inferred from build or Playwright's exit status alone. Human visual verification remains unperformed; mobile page overflow is disproved by settled browser geometry, not by human inspection.
