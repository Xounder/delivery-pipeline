# TASK-10 — Phase 6 Lane 6-A: Tasks and Blocks Feature Migration

## Task Information

### ID

TASK-10

### Title

Migrate `features/tasks/**` and `features/blocks/**` to Mantine while preserving the native form controls the E2E suite binds to

### Phase

Phase 6 lane 6-A — Tasks + Blocks (≈758 lines, 6 files)

### Owner

senior-frontend

### Status

Pending

---

## Description

Six files carry the same inline-style debt as the rest of `apps/web`. They migrate to Mantine in this
lane, with two hard exceptions that the E2E suite forbids relaxing:

- `TaskModal.tsx` (342 lines) keeps a **native `<select>`** nested inside its
  `<label>Duration (minutes)</label>` and a **native `<input>`** for `#task-name`.
- `TaskPanel.tsx` keeps `Tasks ({tasks.length})` as a **single contiguous text node**.

`TaskCard.tsx` and `BlockCard.tsx` are in scope despite being absent from the design's Phase 6 list
(`M10`).

**Parallel dispatch:** this lane runs concurrently with TASK-11 and TASK-12. The three lanes share
zero files. Any edit outside this lane's list invalidates the dispatch.

---

## Files This Task May Create / Modify

| # | File | Lines now | Action |
| --- | --- | --- | --- |
| 1 | `apps/web/src/features/tasks/TaskPanel.tsx` | 60 | modify — Mantine panel |
| 2 | `apps/web/src/features/tasks/TaskModal.tsx` | 342 | modify — Mantine layout, **native controls preserved** |
| 3 | `apps/web/src/features/tasks/TaskCard.tsx` | 69 | modify — Mantine card (`M10`) |
| 4 | `apps/web/src/features/blocks/BlockPanel.tsx` | 54 | modify — Mantine panel |
| 5 | `apps/web/src/features/blocks/BlockModal.tsx` | 168 | modify — Mantine modal |
| 6 | `apps/web/src/features/blocks/BlockCard.tsx` | 65 | modify — Mantine card (`M10`) |

### File count

6 modified, 0 created

### Do Not Touch — forbidden for lane 6-A

- `apps/web/src/features/preview/**`, `features/completion/**`, `features/shared/**` (TASK-11)
- `apps/web/src/components/**` — **including** `components/calendar/**` (TASK-07 / TASK-08)
- `apps/web/src/pages/**` — `DashboardPage.tsx` is **OVERLAP-1**, owned by TASK-06's agent (`P-8`)
- `apps/web/src/styles/**` (TASK-02 / TASK-09 / TASK-14)
- `apps/web/src/main.tsx` (**OVERLAP-2**), `apps/web/src/App.tsx` (TASK-12, **OVERLAP-8**)
- `apps/web/src/contexts/**`, `apps/web/src/hooks/**`
- `packages/ui/src/**`, `packages/ui/package.json`, `packages/ui/dist/**` (**OVERLAP-9 / OVERLAP-10**)
- `apps/web/e2e/**`, `apps/web/tsconfig.json`, `apps/web/playwright.config.ts`, any `package.json` (`P-4`, `P-5`, `P-6`)
- `apps/api/**`, `packages/shared/**`, `packages/backend/**`, `packages/domain/**`, `packages/calendar/**`

### Files this lane may **consume but not edit**

- `@brkroutnxdle/ui` exports (`Button`, `Modal`, `ButtonVariant`, `ButtonSize`, `FAB`, `BottomSheet`)

---

## Constraints Satisfied

- `B2` — native `<select>` inside the `Duration (minutes)` `<label>`; native `#task-name` input
- `M7` — `TaskPanel` mounts exactly once; exactly one `Tasks (N)` panel at every viewport
- `M10` — `TaskCard.tsx` and `BlockCard.tsx` are in scope
- `M16` — no sticky-position CSS added
- `P-5`, `P-6`, `P-8`, `P-9` — specs immutable, config immutable, no `DashboardPage` write, 400-line cap

---

## Implementation Steps

1. **`TaskPanel.tsx`** — replace the inline-styled panel container with Mantine `Paper` /
   `ScrollArea` + `Stack` / `Group`. **Hard constraint:** the heading must render as a single
   contiguous text node:
   ```tsx
   <Title order={4}>Tasks ({tasks.length})</Title>
   ```
   Do **not** split it into `Tasks` + a separate count `Text`, and do not wrap it in a nested
   element that breaks the text run. `TASK-02.spec.ts` locates the panel by this exact string.
   Keep the existing empty-state copy verbatim. No inline `style={{ }}` may survive.
2. **`TaskModal.tsx`** — restyle with `Modal` (from `@brkroutnxdle/ui`, whose public prop stays
   `isOpen`), `Group`, `Stack`, `TextInput`-adjacent layout, `Divider`. **`B2` is non-negotiable:**
   - the duration control MUST remain a native
     `<select>` **inside** the `<label>` whose text contains `Duration (minutes)`; the spec uses
     `label:has-text("Duration (minutes)") select` and `selectOption("#task-duration")`;
   - the name control MUST remain a native `<input id="task-name">`; the spec uses `fill("#task-name")`;
   - `Mantine Select`, `NumberInput` and `TextInput` are **forbidden** in this file for these two
     fields. Mantine `Select` emits a `div[role="listbox"]` and breaks both locators.
   - Keep every existing copy string, the submit/cancel button labels, and the `isOpen` prop name.
   - The file is 342 lines; the Mantine rewrite must not push it over 400 (`P-9`). Extract an inner
     section component into the same folder if it approaches the limit.
3. **`TaskCard.tsx` / `BlockCard.tsx` / `BlockPanel.tsx`** — Mantine `Card`, `Badge`, `Group`,
   `ActionIcon`, `Text`. Preserve every existing label, badge text and click handler. Both card
   files are in scope per `M10`.
4. **`BlockModal.tsx`** — Mantine layout. No spec asserts its internals, so native selects are
   optional here; if a select exists, keeping it native is the lower-risk choice.
5. **Scan all six files** for leftover `style={{` and `!important`-style inline overrides; all
   styling must move to Mantine props or the token classes.
6. **No new files** in this lane. If a shared helper seems necessary, report it instead of creating
   it — new shared files would create cross-lane overlap.

---

## Acceptance Criteria

- [ ] All six files migrated; no inline `style={{` remains in them
- [ ] `TaskModal.tsx` contains a native `<select>` nested inside the `Duration (minutes)` `<label>`, with `id="task-duration"`
- [ ] `TaskModal.tsx` contains a native `<input id="task-name">`
- [ ] `Tasks ({tasks.length})` is one contiguous text node in `TaskPanel.tsx`
- [ ] `TaskPanel` renders exactly once; the document has exactly one `Tasks (` panel and exactly one `<aside>`
- [ ] No new visible copy containing `Preview`, `New Task`, `Edit Task`, `Create in this slot`
- [ ] All six files remain under 400 lines
- [ ] Every file in this lane's table is modified; no file outside it is touched
- [ ] `build packages/ui` → PASS, `build apps/web` → PASS, `test apps/web` → **9/9 PASS**

---

## Dependencies

### Required Tasks

- TASK-09

### Dependency Notes

This lane may only start after Phase 5 is merged, because `packages/ui` must be built and the token
layer must exist before Mantine markup is authored. The three Phase 6 lanes run **concurrently**;
there are no intra-phase dependencies.

### Runs In Parallel With

- TASK-11 (lane 6-B) — file-disjoint
- TASK-12 (lane 6-C) — file-disjoint

---

## Validation Gate

1. `run-package-command build packages/ui` → **PASS**
2. `run-package-command build apps/web` → **PASS**
3. `run-package-command test apps/web` → **9/9 PASS** on a fresh dev server (`M14`)
4. No lint step (`P-4`)

**Merge rule:** after TASK-10, TASK-11 and TASK-12 all complete, the full gate runs once more on the
consolidated tree (TASK-13).

---

## Edge Cases

- **`label:has-text("Duration (minutes)") select` resolves to nothing** — the Mantine `Select` /
  `NumberInput` swap was made. Revert to a native `<select>` inside the `<label>` (`B2`).
- **`selectOption("#task-duration")` times out** — the `id` was moved to a wrapper, or the element
  is now a `div[role="listbox"]`.
- **`fill("#task-name")` resolves but the value never lands in state** — the input is no longer
  controlled, or its `onChange` was rewired.
- **Exactly one `Tasks (` panel assertion breaks** — `TaskPanel` was rendered twice (e.g. desktop +
  mobile variants), or the count was split into a sibling `Text` node (`M7`).
- **`text=Preview` now matches multiple elements** — lane 6-B owns that string; if 6-B is mid-flight
  this lane must not "fix" it by editing files outside its list. Report it.
- **Modal content duplicated in the DOM** — a Mantine modal with `keepMounted` behaviour left on by
  default; that is 6-B's `M8` concern for `ConflictDialog`, but if this lane's modals need it too,
  keep them unmounted when closed by default.
- **`@brkroutnxdle/ui` `Modal` prop mismatch** — the wrapper keeps `isOpen`, not Mantine's `opened`.
  Using `opened` here type-errors and may silently break open/close.
- **400-line ceiling** — `TaskModal.tsx` at 342 lines is the risk; split inside the folder rather
  than crossing into another lane's territory.
- **Stale dev server** — `reuseExistingServer: true` can make this lane's 9/9 a false pass against
  another lane's bundle (`M14`). Report the exact run conditions.
- **CSS specificity regressions** — the panel/card may lose styling because a stylesheet selector
  assumed the old class names; those selectors belong to TASK-09/TASK-14. Report, do not edit CSS.

---

## Testing

### Unit Tests

- [ ] None — no component test runner exists in `apps/web` and none may be added (`P-6`)

### Integration Tests

- [ ] `run-package-command test apps/web` → 9/9, in particular `TASK-02.spec.ts:41,62` (`aside` / `Tasks (` uniqueness) and `TASK-01.spec.ts` task creation, edit and delete flows

### Manual Validation

- [ ] Create a task inline from a slot → modal opens, name and duration editable, submit works
- [ ] Edit and delete a task from the panel
- [ ] Create/edit a block; drag a task onto a block if that interaction is reachable manually
- [ ] Panel count badge updates on add/remove

### E2E Test Authoring Guidance (Playwright)

- [ ] **No E2E spec may be created or modified** (`P-5`)
- [ ] Do not add FullCalendar drag/select automation
- [ ] Report pass/fail per spec file

---

## Definition of Done

- [ ] Implementation completed
- [ ] Acceptance criteria satisfied
- [ ] Lane forbidden lists respected
- [ ] `build packages/ui` passes, `build apps/web` passes
- [ ] `test apps/web` passes 9/9
- [ ] Validation completed

---

## References

- [`planning/dispatch-plan.md` — Lane 6-A](../planning/dispatch-plan.md#lane-6-a--tasks--blocks-758-lines-6-files)
- [`planning/file-impact.md` — Task / Block feature tables](../planning/file-impact.md#task-feature)
- [`planning/impact-analysis.md` — B2, M7, M10, P-8](../planning/impact-analysis.md#material-corrections-m1m18)
- [`design-docs/design-05-component-inventory.md`](../design-docs/design-05-component-inventory.md)
- [`design-docs/design-06-migration-plan.md`](../design-docs/design-06-migration-plan.md)