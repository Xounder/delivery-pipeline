# TASK-14 — Phase 7: Dead-CSS Cleanup and Final Import Order

## Task Information

### ID

TASK-14

### Title

Remove dead responsive rules from `global.css`, prune superseded calendar rules, and tidy the final `main.tsx` import list

### Phase

Phase 7a — Cleanup

### Owner

senior-frontend

### Status

Pending

---

## Description

Mantine replaces the hand-rolled responsive rules and the inline-style debt, which leaves CSS that
nothing renders any more. Phase 7 removes it and finalises the stylesheet import list.

This task runs **strictly sequentially**, after TASK-13's consolidated gate is green. It touches the
three stylesheet files that have `OVERLAP-5`, `OVERLAP-6` and `OVERLAP-7` flags, plus
`main.tsx` (`OVERLAP-2`) — all cross-phase targets that could never be parallelised.

---

## Files This Task May Create / Modify

| # | File | Lines now | Action |
| --- | --- | --- | --- |
| 1 | `apps/web/src/styles/global.css` | ~42 | modify — remove dead responsive rules (**OVERLAP-6**) |
| 2 | `apps/web/src/styles/design-tokens.css` | ~84 | modify — prune superseded token declarations (**OVERLAP-5**) |
| 3 | `apps/web/src/styles/CalendarStyles.css` | ~100 after TASK-09 | modify — prune rules superseded by the partials (**OVERLAP-7**) |
| 4 | `apps/web/src/styles/calendar/calendar-tokens.css` | TASK-09 | modify — prune duplicated declarations |
| 5 | `apps/web/src/styles/calendar/calendar-events.css` | TASK-09 | modify — prune rules for selectors that no longer exist |
| 6 | `apps/web/src/styles/calendar/calendar-dark.css` | TASK-09 | modify — prune overrides for selectors that no longer exist |
| 7 | `apps/web/src/main.tsx` | ~60 | modify — final import order (**OVERLAP-2**) |

### File count

7 modified, 0 created

### Do Not Touch

- `apps/web/e2e/**` (`P-5`)
- `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json`, `vite.config.ts` (`P-4`, `P-6`)
- `apps/web/src/components/**`, `apps/web/src/pages/**`, `apps/web/src/features/**`, `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `packages/ui/**` including `dist/**` (**OVERLAP-10**)
- `apps/api/**`, `packages/shared/**`, `packages/backend/**`

---

## Constraints Satisfied

- `M2` — the reset lives in `global.css`; exactly one token import survives
- `M4` — remaining `global.css` media-query values stay numerically identical to the Mantine theme breakpoints
- `M6`, `M16`, `M17` — no `!important` reintroduced, no competing sticky CSS, partial structure preserved
- `M13` — the final import order is Mantine first, then tokens, then app styles, then calendar partials
- `B4` — contrast-corrected palette survives; no hardcoded literal reintroduced
- `P-9` — every stylesheet stays under 400 lines

---

## Implementation Steps

1. **`global.css` (dead responsive rules).** Identify every media-query rule whose selectors no longer
   match any rendered element after the Mantine migration, and delete it. Keep the rules that still
   apply. **Preserve:**
   - the `* { box-sizing: border-box }` / `body { margin: 0 }` reset (`M2`);
   - the numeric breakpoint values, which must remain **identical** to the Mantine theme breakpoints
     (`M4`). Verify them against `createTheme` in `apps/web/src/theme/` and align if they drifted.
   Do not delete the `@import "./design-tokens.css"` line **and** the `main.tsx` import in the same
   edit — exactly one token load must remain (`M2`).
2. **`design-tokens.css`.** Prune token declarations that no component references any more (for
   example tokens that only the deleted inline styles consumed). Keep the three-tier structure and
   every `--brk-*` name referenced by `CalendarStyles.css`, the calendar partials, `MainLayout`,
   `Header`, `Sidebar`, `StatusBar`, `CalendarToolbar` or any feature file. Verify by grepping the
   codebase for each token name before removing it — an unreferenced-looking token is often still
   referenced through a CSS-only consumer.
3. **`CalendarStyles.css` and the three partials.** Remove rules whose selectors no longer exist:
   - `.fc-event-title`, `.fc-event-time` (TASK-09 already removed these — confirm);
   - any `.brk-event-delete-btn` inline-style leftovers superseded by the Mantine button;
   - any duplicate of a declaration now present in a partial.
   **Never** delete the `--fc-*` layer, the project-owned event classes, the workday tint, the
   `min-height: 100%` / `overflow: visible` pair, or the dark-tier block (`B3`, `B4`).
   Keep the partial split (`M17`) — do not merge them back into one file.
4. **`main.tsx` import order (final, `M13`).** Confirm the order is exactly:
   ```
   @mantine/core/styles.css
   @mantine/notifications/styles.css
   ./styles/design-tokens.css
   ./styles/global.css
   ./styles/CalendarStyles.css
   ./styles/calendar/calendar-tokens.css
   ./styles/calendar/calendar-events.css
   ./styles/calendar/calendar-dark.css
   ```
   Then any application imports. Remove duplicates only. Mantine must load **before** app styles so
   app-level rules win.
5. **Verify no `!important` and no hardcoded Google reds** were reintroduced by any cleanup edit.
6. **Record the bundle delta** (TASK-15 does the formal comparison; record raw values here).

---

## Acceptance Criteria

- [ ] Every deleted CSS rule had no matching element in the rendered app
- [ ] The global reset still exists in `global.css` and is absent from `design-tokens.css`
- [ ] Exactly **one** token import remains across `main.tsx` and `global.css`
- [ ] Remaining `global.css` media-query values match the Mantine theme breakpoints numerically
- [ ] Zero `!important` declarations in the four calendar stylesheets
- [ ] No `.fc-event-title` / `.fc-event-time` selectors anywhere
- [ ] The `--fc-*` layer, project event classes, workday tint, `min-height`/`overflow` pair and dark tier all survive
- [ ] The partial split is preserved (`M17`)
- [ ] `main.tsx` import order matches the list in step 4 exactly
- [ ] Every stylesheet is under 400 lines
- [ ] No file outside this task's table was modified
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-13

### Dependency Notes

TASK-13's consolidated gate must be green before cleanup begins — deleting rules before the merged
tree is validated would confuse dead code with regressions.

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** on a fresh dev server (`M14`)
4. No lint step (`P-4`)

---

## Edge Cases

- **Over-deletion** — a token or rule removed while still consumed by a CSS-only reference. Grep the
  codebase for each token name before deleting; if a name appears only in CSS, that is a real
  consumer, not dead code.
- **The calendar loses its styling after "cleanup"** — a rule that FullCalendar needs was deleted.
  Verify the grid, the event variants and the dark tier visually.
- **Sunday slot becomes unclickable** — the workday tint or the weekend tint rule was pruned (`B3`).
- **`Loading calendar events...` never clears** — an unrelated selector was deleted; check for
  collateral damage rather than assuming the E2E failure is the cause.
- **Mantine styles start winning again** — `main.tsx` import order was disturbed during the tidy.
- **Token sheet loaded twice** — both the `main.tsx` import and the `global.css` `@import` survived
  (`M2`); the bundle grows silently.
- **Breakpoint drift** — a `global.css` media query was "tidied" to a different value than the Mantine
  theme (`M4`). Values must match, not merely be close.
- **Cleanup hides a real defect** — a selector that stopped matching because Phase 6 broke a class name
  is mistaken for dead CSS. If a rule "suddenly" has no match, verify the component still renders the
  expected class before deleting the rule.
- **`keepMounted`-style leftovers** — not applicable here; duplicates are TASK-13's finding.
- **Stale dev server** — false green against the pre-cleanup bundle (`M14`).

---

## Testing

### Unit Tests

- [ ] None — stylesheet-only change

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, including the Sunday `14:00` slot click, `.fc-timegrid-*` geometry and all five `.fc-event:has-text('…')` assertions

### Manual Validation

- [ ] Both tiers still render correctly after cleanup
- [ ] Day / Week / Month switching still works and the toolbar is intact
- [ ] At least one breakpoint width each side of a theme breakpoint renders correctly
- [ ] Settings and login pages still styled

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] A Playwright failure after a cleanup edit is evidence of over-deletion until proven otherwise
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] Dead rules removed with evidence for each deletion
- [ ] Import order finalised
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/dispatch-plan.md` — Phase 7 Gate](../planning/dispatch-plan.md#phase-7-gate--cleanup-and-final-validation)
- [`planning/file-impact.md` — Styles table and OVERLAP flags](../planning/file-impact.md#styles)
- [`planning/impact-analysis.md` — M2, M4, M6, M13, M17, OVERLAP-5/6/7](../planning/impact-analysis.md#material-corrections-m1m18)
