# Design 07 — Risk Register, E2E Regression Register, Open Questions

## Context

### Problem Statement

This delivery replaces the visual layer of an application whose acceptance criteria are two
Playwright specs that assert on FullCalendar internals, specific DOM elements and exact copy.
A redesign that looks correct but fails the suite is a failed delivery.

### Goals

- Enumerate every risk with likelihood, impact and mitigation.
- Provide the E2E regression register: every assertion, the DOM/copy it depends on, and the
  specific change that could break it.
- Record residual open questions.

### Constraints

- `apps/web/e2e/TASK-01.spec.ts` and `TASK-02.spec.ts` must not be modified.
- No user-visible copy changes in this delivery.
- FullCalendar is not replaced.

---

## Options Considered

### Option A (Selected): Front-load the E2E constraint analysis into the design

#### Description

Derive the regression register directly from the two spec files before writing any component
design, and treat it as a binding contract on every phase.

#### Advantages

- Turns an invisible constraint into an explicit, checkable list.
- Catches non-obvious traps (Playwright strict mode on duplicate text) before implementation.
- Gives every parallel lane a copy-safety checklist.

#### Disadvantages

- Adds a document that must be kept in sync as specs evolve.

#### Effort Estimate

Small

---

### Option B (Rejected): Rely on running the suite after each phase

**Rejected because** running the suite detects breakage but does not explain it, and two
failure classes here (strict-mode duplicate matches, `businessHours` slot visibility) are
non-obvious enough that an implementer would likely misdiagnose them as layout bugs.

---

## Decision

### Selected Option

Option A.

### Reasoning

The E2E suite is unusually entangled with implementation details for this app. Making that
entanglement explicit up front is cheaper than debugging it seven times.

### Trade-offs Accepted

- One more document to maintain.

---

## Impact

### Documentation Affected

- All other design documents reference this register.

### Architecture Affected

None — this document is a constraint record.

---

## E2E Regression Register

Every row is a hard requirement. **No copy changes in this delivery.**

### Copy assertions

| # | Asserted string | Spec location | Owner after redesign | Risk if violated |
|---|---|---|---|---|
| C-01 | `Loading calendar events...` | TASK-01:138 | `StatusBar` (priority 4) | Load-state assertion times out |
| C-02 | `Create in this slot` | TASK-01:160,171; TASK-02:146,157 | `CreateActionModal` | Slot-create flow unlocatable |
| C-03 | `Preview` | TASK-01:177 | `PreviewToolbar` → `"Preview: {n} events"` | **Strict-mode violation** if the new toolbar or `ConflictDialog` also matches |
| C-04 | `Schedule saved successfully!` | TASK-01:184 | `StatusBar` (priority 3) | Save confirmation unlocatable |
| C-05 | `Failed to sync calendar events` | TASK-01:195 | `StatusBar` (priority 2) | Negative assertion passes vacuously, or a real error hides |
| C-06 | `Tasks (0)` / `(1)` / `(2)` | TASK-02:56,80,115,167 | `TaskPanel` | **Strict-mode violation** if the label is split across elements |
| C-07 | `New Task` | TASK-02:64,93,121,173 | `TaskModal` (non-edit branch) | Create-task flow broken |
| C-08 | `Edit Task` | TASK-02:187,196 | `TaskModal` (edit branch) | Edit flow broken |
| C-09 | `Sidebar Test Task`, `Idempotency Test`, `Editable Task`, `Sidebar Task`, `Inline Slot Task`, `Edited Task` | TASK-02:82,87,100,111,128,161,180,204 | Event title + sidebar list | See D-02 |

### DOM assertions

| # | Selector | Spec location | Requirement |
|---|---|---|---|
| D-01 | `.fc-timegrid-slots` | TASK-01:142; TASK-02:137 | Slot container must exist |
| D-02 | `.fc-event:has-text('…')` | TASK-02:87,128,161,180,204 | Title text must be **inside** `.fc-event`; `eventContent` must render it |
| D-03 | `.fc-event` count | TASK-02:100,111 | Exactly one `.fc-event` per event — `eventContent` replaces children, never adds elements |
| D-04 | `.fc-timegrid-slot:has-text('14:00')` | TASK-01:146; TASK-02:132 | Slot labelled `14:00` must exist **and be clickable** — constrains `businessHours` |
| D-05 | `.fc-timegrid-body` | TASK-01:151; TASK-02:137 | Used for `boundingBox()` mouse math → real geometry required |
| D-06 | `aside` | TASK-02:82 | `<aside>` must remain; `AppShell.Navbar component="aside"` |
| D-07 | drag source in `TaskPanel` | TASK-02:128 | Expand the Tasks accordion before dragging; Mantine 8.3.18 `AccordionPanel` has no `keepMounted` prop |

### Copy-safety rules (binding on all phases)

1. **No new visible text may contain** `Preview`, `New Task`, `Edit Task`, `Tasks (`,
   `Create in this slot`, `Loading calendar events`, `Schedule saved successfully`,
   `Failed to sync calendar events`.
2. `Tasks (N)` and `Preview: N events` must each render as **one contiguous text node**.
3. `New Task` / `Edit Task` must remain mutually exclusive (the existing ternary is fine).
4. The new `CalendarToolbar` **replaces** the calendar-navigation part of `Header`; exactly one
   `Today` control may exist in the DOM.
5. Status banners become mutually exclusive by priority (`design-03`); QA must confirm no
   spec depends on two being visible simultaneously — inspection says none does.

---

## General Risk Register

| ID | Risk | Likelihood | Impact | Mitigation |
|---|---|---|---|---|
| R-01 | Mantine 9 pulled in by mistake → React 19.2 peer error at build | Medium | High | Pin `8.3.18` exactly; document the constraint in `design-01` |
| R-02 | PostCSS preset misconfiguration breaks the build | Medium | Medium | Phase 1 gate is a bare `build` with no UI change |
| R-03 | Two Mantine instances via pnpm peer resolution | Medium | Medium | Peer + dev dependency in `packages/ui`; verify a single `.pnpm` entry |
| R-04 | Dark tier fails contrast on event chips | Medium | High | Contrast pairs tabulated in `design-02`; QA screenshots both tiers |
| R-05 | Removing the hand-rolled reset regresses layout | Low | Medium | Phase 2 gate includes Playwright |
| R-06 | `AppShell` default padding breaks the calendar height chain | Medium | High | `padding={0}`; QA checks `.fc-timegrid-body` geometry |
| R-07 | `<aside>` lost → TASK-02:82 fails | Medium | High | `component="aside"` required; D-06 |
| R-08 | `Tasks (N)` split → strict-mode failure | Medium | High | Single text node; C-06 |
| R-09 | Duplicate `Preview` match → strict-mode failure | Medium | High | Copy-safety rule 1; C-03 |
| R-10 | `eventContent` drops the title → 6 assertions fail | Medium | High | D-02; dedicated `.brk-event-title` span |
| R-11 | `businessHours` de-emphasises the `14:00` slot | Medium | High | R-CAL-01 in `design-04`; QA runs both specs per colour scheme |
| R-12 | Sticky headers break the sticky time gutter | Low | Medium | R-CAL-04; fallback documented |
| R-13 | `eventContent` regresses month view chips | Medium | Low | Render `timeText` only for `timeGrid*` views |
| R-14 | Modal focus restoration breaks the drag-create flow | Low | Medium | QA verifies TASK-02:157 |
| R-15 | A rewritten file exceeds the 400-line project limit | Medium | Medium | `SettingsModal.tsx` and `TaskModal.tsx` are the candidates; extract |
| R-16 | Parallel lanes collide on a shared file | Medium | High | Overlap flags in `design-06`; `DashboardPage.tsx` single-owner |
| R-17 | Bundle growth from Mantine + Tabler icons | Low | Medium | Import icons individually (tree-shakeable); verify in `build` output |
| R-18 | Deferred React 19 upgrade later forces a Mantine 9 migration | Medium | Low | Documented in `design-01` as accepted debt |

---

## Open Questions

1. **Colour scheme persistence** — `localStorage` (proposed) vs. server-side via
   `SettingsContext`. Server-side requires a backend contract change, which is out of scope.
   *Default: `localStorage`.*
2. **Collapsed sidebar** — icon-only 56px rail (proposed) vs. current `0px` hide. *Default:
   icon-only.*
3. **Business-hours badge copy** — `"09:00 – 18:00"` (proposed) vs. `"Working hours"`. Neither
   collides with an asserted string.
4. **Week summary in the toolbar** — a derived "12 events · 4h 30m" line would strengthen the
   hierarchy but adds a derived-data concern. *Default: deferred.*
5. **Mantine 9 / React 19.2 upgrade** — explicitly out of scope, recorded as a prerequisite for
   any future move to `@mantine/schedule` (which would eventually replace FullCalendar, but
   would require rewriting both Playwright specs).

None of these block implementation.
