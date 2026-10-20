# TASK-15 — Phase 7: Final Line-Count, Contrast, Bundle and Full-Gate Audit

## Task Information

### ID

TASK-15

### Title

Run the delivery's closing audit: 400-line compliance, contrast verification, bundle delta against baseline, and the final full validation gate

### Phase

Phase 7b — Final validation

### Owner

senior-frontend

### Status

Completed — QA approved on 2026-10-18; evidence in TASK-15-final-audit-report.md.

---

## Description

The closing gate. TASK-14 removed dead CSS; this task proves the delivery satisfies every structural
and visual constraint, records the bundle delta against the measured baseline, and runs the full
validation one final time on a fresh dev server.

Validation-first: this task fixes only defects that its own audit surfaces, and any source change is
routed to the owning task's file set.

---

## Files This Task May Create / Modify

| # | File | Action |
| --- | --- | --- |
| 1 | `.opencode/plan/ui-ux-modernization/tasks/TASK-15-final-audit-report.md` | **create** — the audit record |

### File count

1 created (a report), 0 source files modified

### Allowed Conditional Edits

Only audit-discovered defects in files whose owning task is closed:

- line-count violations → split within the owning file's folder
- contrast failures → correct the token value in `apps/web/src/styles/design-tokens.css` (**OVERLAP-5**, Phase 7 owner)

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-6`)
- `pnpm-lock.yaml` — regenerated only via `pnpm install` (**OVERLAP-4**)
- `packages/ui/dist/**` — generated only (**OVERLAP-10**)
- `apps/api/**`, `packages/shared/**`, `packages/backend/**`

---

## Constraints Satisfied

- `P-9` — no source file above 400 lines anywhere in `apps/web` or `packages/ui`
- `B4` — event and brand palette contrast verified in **both** tiers
- `M12` — build is not evidence for the E2E contract
- `M14` — fresh dev server for the final Playwright run
- `R-05` — `packages/ui` built before `apps/web`
- `R-25` — consolidated gate

---

## Implementation Steps

1. **Line-count audit.** Enumerate every source file in `apps/web/src` and `packages/ui/src` and list
   any file above **400 lines**. The known starting offender was
   `apps/web/src/pages/DashboardPage.tsx` at 567. For each remaining offender, split it **inside its
   own folder** — never across another task's file set. Report the before/after counts.
2. **Contrast audit (`B4`).** For the event and brand palette, verify in **both** tiers:
   - event fill vs event text ≥ **4.5:1**;
   - event fill vs calendar background ≥ **3:1** for non-text boundaries;
   - the brand/accent colour is **not** `#3CC4BD` paired with white text (≈2.1:1). If a token still
     carries that pairing, fix the **token** in `design-tokens.css` and re-verify.
   - Record each pair and its measured ratio in the report. A claim without a number is not evidence.
3. **Structural constraint audit.** Confirm, by inspection, that:
   - `eventDidMount` / `eventWillUnmount` are gone from `CalendarView.tsx`;
   - no `businessHours` prop exists on the `<FullCalendar>` instance;
   - the view switcher uses real `<button>` elements, not `SegmentedControl` (`B1`);
   - `TaskModal.tsx` and `CreateActionModal.tsx` keep their native `<select>` / `<input>` (`B2`);
   - `ConflictDialog` unmounts when closed (`M8`);
   - exactly one `<aside>` and one `Tasks (N)` panel at desktop and mobile widths (`M7`);
   - every calendar selector is scoped to `.brk-calendar-google` (`M1`);
   - zero `!important` in the calendar stylesheets (`M6`);
   - `main.tsx` import order is Mantine-first (`M13`);
   - exactly one token import (`M2`).
4. **Bundle delta.** Compare the `build apps/web` output against the measured baseline and record the
   delta. The baseline is **reported, not enforced** — no bundle-budget script exists and none may be
   added (`P-4`):
   | Metric | Baseline |
   | --- | --- |
   | JS raw | 524.63 kB |
   | JS gzip | 156.62 kB |
   | CSS raw | 6.40 kB |
   | CSS gzip | 1.75 kB |
   | Modules transformed | 158 |
   Mantine is expected to increase the bundle materially; a decrease with Mantine present would
   indicate a stylesheet or module is not actually being imported.
5. **Final full gate, in order, on a fresh dev server:**
   1. `run-package-command build packages/ui` → expect **PASS**
   2. `run-package-command build apps/web` → expect **PASS**
   3. `run-package-command test apps/web` → expect **9/9 PASS**
   Confirm no stale dev server is bound to the dev port first (`M14`); a run on a reused server is
   **invalid**. Record the run conditions.
6. **Write the audit report** to
   `.opencode/plan/ui-ux-modernization/tasks/TASK-15-final-audit-report.md` containing every section
   above, the contrast table with measured ratios, the bundle delta table, the three gate results, and
   the constraint-verification checklist.
7. **Do not close the delivery** if any item above is unresolved.

---

## Acceptance Criteria

- [x] No source file in `apps/web/src` or `packages/ui/src` exceeds 400 lines
- [x] Every audited opaque event/brand base-token pair meets its `B4` ratio in both tiers, with measured values recorded; overlays not exhaustively audited
- [x] `#3CC4BD` is not paired with white text in inspected event/brand styling
- [x] All ten structural constraints in step 3 verified and recorded (one aside/panel at four widths)
- [x] Bundle delta recorded against the 524.63 kB / 156.62 kB baseline for all four metrics
- [x] `build packages/ui` → **PASS**
- [x] `build apps/web` → **PASS**
- [x] `test apps/web` → **9/9 PASS** on a fresh dev server, conditions documented by final QA
- [x] Report written to `TASK-15-final-audit-report.md`
- [x] No E2E spec, config or manifest modified in TASK-15 recovery
- [x] Recovery edits authorized within closed-task ownership and permanent QA tooling; scope deviation recorded in the audit report

---

## Dependencies

### Required Tasks

- TASK-14

### Dependency Notes

TASK-14's cleanup must land first — auditing before the dead CSS is removed would report violations
that cleanup then silently fixes.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** (fresh dev server — `M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **A 400+ line file reappears** — a Phase 6 lane's Mantine rewrite grew a file past the cap. Split
  inside that file's folder and re-run the audit; do not raise the limit.
- **Contrast measured only in one tier** — light mode passes while dark mode fails, or the token is
  defined once and overridden per tier. Audit both tiers independently.
- **Bundle shrinks unexpectedly** — a stylesheet or module is no longer imported. Verify
  `main.tsx` and the partial imports before accepting the number.
- **Bundle grows far beyond the expected Mantine delta** — likely the whole of Mantine plus
  duplicate token sheets, or `@mantine/dates`/unneeded modules pulled in transitively. Check the
  dependency list against TASK-01's manifest changes.
- **Final Playwright run fails after a green TASK-14 run** — a stale dev server, or a real
  regression introduced by cleanup. Do not accept the earlier run.
- **`build apps/web` passes but E2E fails** — expected risk class; specs are outside the TypeScript
  program (`M12`).
- **`packages/ui/dist` stale** — `build packages/ui` first, always (`R-05`).
- **Audit finds a defect owned by a closed task** — fix it inside that task's file set or record it
  as an open defect. Do not widen this task's file list to reach it.
- **The delivery is signed off with a "probably green" claim** — every item in steps 1–5 needs recorded
  evidence, not an assertion.

---

## Testing

### Unit Tests

- [ ] None — audit task

### Integration Tests

- [x] The three ordered build/build/test commands passed; final QA established fresh-server evidence by HTTP probe before rerun
- [x] Line-count enumeration summarized in report; complete per-file output emitted by permanent QA runner

### Manual Validation

- [ ] Both tiers visually verified for event, brand, grid and status regions
- [x] Automated Chromium at 1440, 768, 576, 375 confirms exactly one `<aside>` and task panel and no document-width overflow after resize settles
- [x] Automated real-Chromium view switching, desktop/mobile slot clicks, task CRUD/external mouse drag, preview/discard, mocked-API save, settings and dev mode work on an owned fresh server (not human visual review or live OAuth/API)

### E2E Test Authoring Guidance (Playwright)

- [x] **No E2E spec may be created or modified** (`P-5`); existing specs unchanged
- [x] Final acceptance run passed and QA approved
- [x] Report pass/fail per spec file

---

## Definition of Done

- [x] Line-count audit clean
- [x] Opaque event/brand base-token contrast audit clean in both tiers, with measured values
- [x] Structural constraints verified
- [x] Bundle delta recorded
- [x] Full gate green on a fresh dev server
- [x] Report written
- [x] Validation completed; QA approved (automated browser evidence, no human visual-review claim)

---

## References

- [`planning/dispatch-plan.md` — Phase 7 Gate](../planning/dispatch-plan.md#phase-7-gate--cleanup-and-final-validation)
- [`planning/feasibility.md` — B4](../planning/feasibility.md)
- [`planning/risk-register.md`](../planning/risk-register.md)
- [`planning/file-impact.md` — Totals](../planning/file-impact.md#totals)
