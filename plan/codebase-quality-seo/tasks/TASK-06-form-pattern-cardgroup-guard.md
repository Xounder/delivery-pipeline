# Task — Phone Input Pattern & DefaultCardGroup Guard Reorder

## Task Information

### ID

TASK-06

### Title

Replace the unsatisfiable phone `pattern`, add `inputMode="tel"`, and move the null guard before the dereference

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.13** | High | Replace the unsatisfiable phone `pattern` with a usable rule and add `inputMode="tel"` — must accept the same or a superset of today's input space | `src/components/ContactForm/ContactForm.tsx:23-25` |
| **W1.10** | High | Move the `selectedData &&` guard before the `selectedData.cards.length` dereference | `src/components/DefaultCardGroup/DefaultCardGroup.tsx:33,35,39` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged — **with one approved exception: W1.13 changes the `pattern` value** (that is the fix; invariant 3's copy clause governs *visible text and alt strings*, not validation constraints).
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Explicitly protected by OOS-1:** the contact form's submit behaviour. `ContactForm.tsx:8-11,31` — the `preventDefault()` + `dialogRef.current?.showModal()` simulated submit and the "Sua mensagem foi enviada!" message — is **out of scope and must not change**. W1.13 changes only the input's validation constraint and keyboard hint.

---

## Description

Two independent defects with a shared theme: **code that runs in an order that makes it unreachable or unsatisfiable.**

**W1.13 — unsatisfiable phone `pattern`.** `ContactForm.tsx:23-25` declares a `pattern` that no real phone number can satisfy, so the browser's native constraint validation always fails and the form cannot be submitted at all. The fix is a usable rule plus `inputMode="tel"` so mobile keyboards show digits.

The hard constraint: **the accepted input space must be the same as today, or a superset.** Narrowing what users may type is a functionality regression. Since today's accepted set is *empty*, almost any real pattern is a superset — but the intent must be a Brazilian phone rule consistent with the app's `pt-BR` strings, and it must be verified against the browser's native validation UI.

**W1.10 — guard after dereference.** `DefaultCardGroup.tsx` reads `selectedData.cards.length` at `:35` *before* the `selectedData &&` check at `:39`. JavaScript evaluates top-down, so the null/undefined check never protects the property access. The dereference throws `TypeError: Cannot read properties of undefined (reading 'cards')` whenever `selectedData` is nullish.

Stage 1 delivers **only the reorder**. The unsafe `as keyof typeof` cast at `:33` is *not* removed here — that depends on W2.3's discriminated union and lands in TASK-09 (Stage 2). `feasibility.md` is explicit: *"Land the reorder in Stage 1, the cast removal in Stage 2."*

---

## Acceptance Criteria

- [ ] `ContactForm.tsx:23-25` declares a `pattern` that a real Brazilian phone number satisfies.
- [ ] The accepted input space is the same as, or a superset of, today's. Verified by entering a valid number and confirming the native validation UI passes.
- [ ] `inputMode="tel"` is present on the phone input.
- [ ] A mobile device (or device emulation with a virtual keyboard) shows a numeric/telephone keypad for the phone field.
- [ ] The `title` attribute on the phone field is consistent with the new rule (visible on hover / to assistive tech). Any wording change is minimal and its purpose is recorded.
- [ ] **The form's submit path is unchanged** — `preventDefault()` + `showModal()` + the "Sua mensagem foi enviada!" message all behave exactly as before. OOS-1 respected.
- [ ] `DefaultCardGroup.tsx` evaluates its `selectedData` nullish guard **before** any property access on `selectedData`.
- [ ] The `as keyof typeof` cast at `:33` is **still present** — removing it is TASK-09's job, not this task's.
- [ ] No rendering change on either route. Screenshot diff clean at all 13 widths (the form is inside a `<dialog>`, so verify the dialog at widths 1, 2 and 3 specifically).
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01

### Dependency Notes

TASK-01 is required so `npm run lint` / `npm run build` can run.

No dependency on any other Stage 1 task — this task's files are disjoint from TASK-02, TASK-03, TASK-04, TASK-05.

**Downstream overlaps:**
- **TASK-08** (W1.17) — `ContactForm.tsx:19` heading re-levelling. Sequential.
- **TASK-11** (W2.4) — `ContactForm.tsx:27,32` button consolidation. Sequential (Stage 2).
- **TASK-20** (W4.8, W4.9) — `ContactForm.tsx:21-26,30,31,32` labels and `lang` marking. Sequential (Stage 4).
- **TASK-09** (W2.3) — `DefaultCardGroup.tsx:11-33` discriminated union; **removes the cast** that this task deliberately leaves in place. Sequential (Stage 2).
- **TASK-12** (W2.8) — `DefaultCardGroup.tsx:42` index key. Sequential (Stage 2).
- **TASK-17** (W3.8) — `DefaultCardGroup.tsx:38` inline style. Sequential (Stage 3).

---

## File Overlap Warning

`src/components/ContactForm/ContactForm.tsx` (4 work items: W1.13, W2.4, W4.8, W4.9) and `src/components/DefaultCardGroup/DefaultCardGroup.tsx` (4 work items: W1.10, W2.3, W2.8, W3.8).

| File | Other task | W-IDs / lines | Merge strategy |
|---|---|---|---|
| `ContactForm.tsx` | **TASK-08** | W1.17 at `:19` | **Sequential.** Adjacent lines to W1.13's `:23-25`. |
| `ContactForm.tsx` | **TASK-11** | W2.4 at `:27,32` | **Sequential** — Stage 2. |
| `ContactForm.tsx` | **TASK-20** | W4.8 `:21-26,30`, W4.9 `:24,31,32` | **Sequential** — Stage 4. **High merge risk at `:24`** (the phone `title`, same attribute W1.13 touches). Preserve the new `pattern`/`inputMode` when adding `lang="pt-BR"`. |
| `DefaultCardGroup.tsx` | **TASK-09** | W2.3 `:11-33`, W2.8 `:42` | **Sequential** — Stage 2. TASK-09 removes the cast this task leaves in place; verify the guard reorder survives. |
| `DefaultCardGroup.tsx` | **TASK-12** | W2.8 `:42` | **Sequential** — Stage 2. |
| `DefaultCardGroup.tsx` | **TASK-17** | W3.8 `:38` | **Sequential** — Stage 3. Adjacent to W1.10's `:35,39`. |
| `ContactForm.css` | **TASK-15**, **TASK-16** | W3.2, W3.3, W3.5, W3.11 | **Sequential** — Stage 3. This task does **not** modify the CSS. |

Within Stage 1 this task is parallel-safe with TASK-02, TASK-03, TASK-04, TASK-05.

---

## Technical Context

### Relevant Components

- `ContactForm` — name, email, phone inputs and a `<dialog>` (OOS-1: the simulated submit is out of scope)
- `DefaultCardGroup` — card group container with a typed lookup

### Relevant Modules

- `src/components/ContactForm/ContactForm.tsx:8-11, 19, 21-26, 30, 31`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:11-33, 33, 35, 38, 39, 42`

### Relevant APIs

- HTML constraint validation — `pattern`, `inputMode`, `title` (the `title` attribute feeds the native validation message)
- `requestSubmit` / native form submission (must remain unchanged)
- `<dialog>.showModal()` (must remain unchanged)

### Relevant Types

- `selectedData` — possibly-nullish card data in `DefaultCardGroup`
- `cardDataMap` — keyed lookup guarded by `as keyof typeof` (cast removal deferred to W2.3)

---

## Implementation Guidance

### Expected Changes

1. **W1.13** — replace the `pattern` on the phone input with a rule that accepts real Brazilian numbers, and add `inputMode="tel"`. Keep the anchor form (`^…$`) explicit so partial matches are not silently accepted. Adjust the `title` minimally so the native validation message describes the actual rule.
2. **W1.10** — move the `selectedData` nullish check above the first property access. Straight reorder; no logic change.
3. **Leave the `as keyof typeof` cast at `:33` in place.** Add no comment claiming it is safe; TASK-09 removes it.

### Constraints

- **Never narrow the accepted input space.** A pattern that rejects a number a Brazilian user would legitimately type is a functionality regression (invariant 1). Test with: `11987654321`, `(11) 98765-4321`, `11987654321` with a leading `+55`, and an empty field (the `required` attribute governs that case, not `pattern`).
- **Do not add, remove, or reorder any form field.** W4.8 (TASK-20) adds `<label>` elements; that is Stage 4.
- **Do not touch the submit handler, the `<dialog>`, or the success message.** OOS-1.
- **Do not change `type` on any input.** `type="tel"` and `inputMode="tel"` are different things — the plan calls for `inputMode`, not a `type` change.
- **Do not remove the cast at `:33`.** Stage 2 owns it.
- **Do not rename `cards`, `cardDataMap`, or the `.group-cards` class.** Invariant 5.
- Do not add validation logic, error messages, or a custom validity API. The fix is the constraint value only.

### Validation Rules

- Native constraint validation must be exercised in a real browser — the validation bubble does not render identically across engines, and a green `tsc` says nothing about it.
- Verify at widths 1, 2 and 3 (360×640, 390×844, 430×932) that the numeric keypad hint appears on a real mobile device or device emulation.
- Verify the success path still works end-to-end after the pattern change: fill all three fields with valid data, submit, confirm the `<dialog>` opens and the modal copy appears.
- Screenshot diff at all 13 widths (the form lives in a `<dialog>`, so the dialog's open state must be captured at widths 1–3 at minimum).
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **The `pattern` is anchored or unanchored.** An unanchored pattern (`\d{2,5}`) matches *any substring*, so the field would accept nearly anything — which may be why the current rule "looks broken". The replacement must be explicitly anchored (`^…$`) or the constraint is meaningless.
- **Brazilian phone numbers have variable shapes.** `(11) 98765-4321`, `11 98765-4321`, `11987654321`, `+55 11 98765-4321`, and landlines (`11 3333-4444`) all exist. A pattern that accepts only mobile-with-DDD rejects valid landline input — a narrowing of the *intended* input space even though today's accepted set is empty. Verify against real formats, not just mobile.
- **A pattern that is too permissive** silently disables validation without anyone noticing. Confirm the native validation bubble still fires for an obviously invalid value (`123`, `abcdefghijk`).
- **`title` feeds the native validation message.** It is user-visible copy on invalid submit. Invariant 3 governs visible text — a minimal, accurate rewording is required by the fix, but do not rewrite it into marketing copy or add new instructions beyond what the constraint needs. `feasibility.md` W1.13 flags that verifying the accepted set "needs a browser check of the native constraint-validation UI on widths 1–3".
- **`inputMode="tel"` vs `type="tel"`.** `inputMode` only affects the virtual keyboard; `type="tel"` affects both keyboard and semantics. The plan specifies `inputMode`. Do not also change `type` — that is a semantic change beyond the work item.
- **The `pattern` only applies to non-empty values.** An empty field fails `required`, not `pattern`. Changing `pattern` must not interfere with the `required` path or with the success dialog.
- **W4.9 (TASK-20) will add `lang="pt-BR"` to the phone strings.** The `pattern` should be written with Brazilian conventions regardless; do not defer the decision to that task.
- **The guard reorder may be a no-op in practice** if `selectedData` is never actually nullish — the compiler may already narrow it. If TypeScript reports the guard as unreachable, that is itself a finding: report it rather than forcing the reorder. The defect is latent, not live (`risks.md` T7: *"Currently unreachable"*).
- **Reordering the guard changes short-circuit semantics.** If any code between the dereference and the guard has a side effect, moving the guard ahead of it changes behaviour. Read the full block (`:33-42`) before moving anything.
- **`DefaultCardGroup` renders inside `<dialog>`?** No — it is the main card group. Confirm which component owns which markup so the screenshot diff targets the right region.
- **`cardDataMap` lookup returns `undefined` even after the guard.** A truthy `selectedData` does not guarantee `cardDataMap[selectedData.type]` exists. The guard reorder fixes the null crash; it does **not** make the lookup total. That is W2.3 (TASK-09) and W1.11 (TASK-07). Do not over-fix here.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Enter `11987654321` → native validation passes.
- [ ] Enter `(11) 98765-4321` → native validation passes.
- [ ] Enter a landline (`11 3333-4444`) → confirm the decision (accept or reject) is deliberate and recorded.
- [ ] Enter `123` and `abcdefghijk` → native validation fails (the constraint is not vacuous).
- [ ] Empty field → `required` fires, `pattern` does not interfere.
- [ ] Fill all fields with valid data, submit, confirm the `<dialog>` opens and the modal copy is unchanged (OOS-1).
- [ ] Real mobile device / device emulation at 360×640, 390×844, 430×932 → numeric keypad for the phone field.
- [ ] Read `DefaultCardGroup.tsx:30-45` and confirm the guard precedes every `selectedData` property access; confirm the `as keyof typeof` cast is still present.
- [ ] Screenshot diff at all 13 widths, including the open `<dialog>` at widths 1–3.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; the form walkthrough takes ~3 min plus ~7 min for the 13-width diff.

---

## Definition of Done

- [ ] Phone `pattern` accepts real Brazilian numbers, rejects obviously invalid input
- [ ] Accepted input space not narrowed; decision recorded
- [ ] `inputMode="tel"` added
- [ ] `DefaultCardGroup` guard reordered ahead of the dereference
- [ ] `as keyof typeof` cast still present (deferred to TASK-09)
- [ ] Form submit path unchanged (OOS-1 respected)
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.10, W1.13
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 rows for W1.10 / W1.13 (W1.10: "Land the reorder in Stage 1, the cast removal in Stage 2"; W1.13: "must accept a **superset**")
- `.opencode/plan/codebase-quality-seo/risks.md` — T7, OOS-1
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 3 overlap map
- `src/components/ContactForm/ContactForm.tsx:8-11, 19, 21-26, 30, 31`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:33, 35, 39`