# Task — Data-Layer Dependency Inversion & Discriminated Card Union

## Task Information

### ID

TASK-09

### Title

Move data-facing types out of component modules and collapse the two card prop types into a discriminated union

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 2 — Organisation, DRY & KISS

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W2.1** | High | Break the inverted dependency: `data.ts` imports prop types from components. Move data-facing shapes into a types module | `src/data/data.ts:1-4` |
| **W2.3** | High | Collapse `ExperienceCardsProps` + `ProjectCardsProps` into a discriminated union so `DefaultCardGroup`'s `cardDataMap` needs no cast | `src/data/data.ts:32-35,171-174`, `src/components/DefaultCardGroup/DefaultCardGroup.tsx:11-33` |
| **W2.7** | Medium | Remove the duplicate `.group-cards` rule block | `src/components/DefaultCardGroup/DefaultCardGroup.css:1-13` vs `:15-18` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged. **All 14 card entries' text is unchanged.**
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **`.group-cards` in W2.7 is a CSS declaration to delete, not a class to rename — the class itself stays.**

---

## Description

**W2.1 — inverted dependency.** `data.ts` imports `AboutMeProps`, `ExperienceCardInfoProps`, `ProjectCardInfoProps` and `DefaultCardProps` **from component files**. The data layer therefore depends on the presentation layer — backwards, and it means a component refactor breaks the data module's types. The fix relocates the data-facing shapes into a types module that components import *from*.

**W2.3 — two near-identical prop types.** `ExperienceCardsProps` and `ProjectCardsProps` (`data.ts:32-35,171-174`) differ only by a discriminant. Collapsing them into one union (`kind: 'project' | 'experience'`) lets `DefaultCardGroup`'s `cardDataMap` (`DefaultCardGroup.tsx:11-33`) be typed without the unsafe `as keyof typeof` cast at `:33`.

This is the enabling change for two Stage 1 deferrals:
- **W1.10's cast removal** (TASK-06 deliberately left the cast in place) — lands here.
- **W1.11's exhaustive lookup** (TASK-07 deliberately added only a safe fallback) — completed here.

**W2.7 — duplicate CSS block.** `DefaultCardGroup.css` declares `scrollbar-width` / `scrollbar-color` verbatim twice (`:1-13` and `:15-18`). Four duplicated lines with no effect.

---

## Acceptance Criteria

- [ ] `src/data/data.ts` imports **no** type from any file under `src/components/`. Grep-verified.
- [ ] A single new types module holds the data-facing shapes. Components import their prop types from it (or from their own file, re-exporting from it — but the *definition* lives in one place).
- [ ] `ExperienceCardsProps` and `ProjectCardsProps` are replaced by one discriminated union with a `kind: 'project' | 'experience'` discriminant.
- [ ] `DefaultCardGroup.tsx:33`'s `as keyof typeof` cast is **removed** and the `cardDataMap` lookup type-checks without it.
- [ ] `tsc -b` passes with **zero** type errors and no new `@ts-expect-error` or `@ts-ignore` comments.
- [ ] The **14 card entries** and all 6 recognitions in `data.ts` render byte-identically: same titles, descriptions, links, images, `yearPeriod` strings, `description: string[]` shapes.
- [ ] `DefaultCardGroup.css` contains exactly one declaration of `scrollbar-width` and one of `scrollbar-color`; the four duplicated lines are deleted.
- [ ] The `.group-cards` class name still exists and still selects the same element (invariant 5).
- [ ] Screenshot diff clean at all 13 widths (the card grid is the largest visual region on `/`).
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- Stage 1 exit criteria (all Stage 1 tasks complete, `index.md` Stage 1 exit criteria met)

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **Stage 1 exit** — `index.md` Stage 2 entry criteria: *"All Stage 1 exit criteria met."* Stage 1 must be green (lint ≤ baseline, build passing, 13-width diff clean, no uncleared timers, no observer re-creation) before any structural refactor begins.

**Downstream enablement (this task unblocks):**
- **TASK-11** (W2.4, button consolidation) — `feasibility.md`: *"W2.4 Depends on W2.1."* The new types module is where button props should live.
- **TASK-12** (W2.8, stable keys) — `feasibility.md`: *"W2.8 Depends on W2.3."* Keys are derived from the union's fields.
- **TASK-19** (W4.10) — `feasibility.md`: *"W4.10 Depends on W2.1."* `<time datetime>` mapping needs `yearPeriod` typed from the data layer.

---

## File Overlap Warning

`src/data/data.ts` is a **max-1-concurrent-editor** file carrying **6 work items** across the plan (W2.1, W2.3, W4.3, W4.6, W4.10, W4.16). `risks.md` **R21** mandates serialisation.

| File | Plan items | This task's scope | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/data/data.ts` | **6** | `:1-4` imports, `:32-35`, `:171-174` types | **TASK-12** (W2.9 `:27-29`), **TASK-19** (W4.3 `:177-277`, W4.6 `:99,117,135,182,194,206,221,245,268`, W4.10 `:53,75,94,111,129,147,166`, W4.16 `:99,117,135,182,194,206`) | **Strictly sequential, every time.** This task (Stage 2) → TASK-12 (Stage 2) → TASK-19 (Stage 4). **R21 applies: do not let two editors touch this file in the same batch, ever.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.tsx` | 4 (W1.10, W2.3, W2.8, W3.8) | `:11-33` | TASK-06 (W1.10 `:33,35,39`, done), **TASK-12** (W2.8 `:42`), **TASK-17** (W3.8 `:38`) | **Sequential.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.css` | 3 (W2.7, W3.2, W3.5) | `:1-13` vs `:15-18` | **TASK-15** (W3.2), **TASK-16** (W3.5 `:11,17,29`) | **Sequential.** W2.7 deletes duplicate lines at the top of the file; W3.2 rewrites media queries and W3.5 tokenises colours at `:11,17,29`. The ranges do not overlap, but **W3.2/W3.5 are Stage 3 and this task is Stage 2 — the stage boundary serialises them regardless. `impact-analysis.md` also requires W3.5 to follow W3.2 per file.** |
| New types module (e.g. `src/data/types.ts`) | — | created here | **TASK-19** (W4.14 `src/data/seo.ts`, W4.10) | **Sequential.** TASK-19 may extend this module, so it must read the post-TASK-09 shape. Stage 4 also starts only after Stage 3 exits. |

**Do not touch** `src/components/AboutMe/`, `src/components/ExperienceCardInfo/`, `src/components/ProjectCardInfo/`, `src/components/DefaultCard/` for W2.1 beyond updating their *type imports*. Their prop types must now come from the data layer, so their import lines change — but no runtime code changes.

---

## Technical Context

### Relevant Components

- `DefaultCardGroup` — the consumer whose `cardDataMap` loses its cast
- `AboutMe`, `ExperienceCardInfo`, `ProjectCardInfo`, `DefaultCard` — the components whose prop types move

### Relevant Modules

- `src/data/data.ts:1-4, 27-29, 32-35, 171-174`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:11-33, 33, 42`
- `src/components/DefaultCardGroup/DefaultCardGroup.css:1-13, 15-18`

### Relevant APIs

- TypeScript `import type`, discriminated unions, `satisfies`
- CSS `scrollbar-width`, `scrollbar-color`

### Relevant Types

- `AboutMeProps`, `ExperienceCardInfoProps`, `ProjectCardInfoProps`, `DefaultCardProps` (moving out of components)
- `ExperienceCardsProps`, `ProjectCardsProps` → collapsed into one discriminated union
- `cardDataMap` in `DefaultCardGroup.tsx:11-33`

---

## Implementation Guidance

### Expected Changes

1. **W2.1** — create a types module under `src/data/` (e.g. `src/data/types.ts`). Move the four data-facing prop shapes there. Change `data.ts:1-4` to import from it (or, better, have `data.ts` import nothing type-wise from components at all). Update the four components to import their prop types from the new module. Every shape's **fields and optionality must be identical** — this is a move, not a redesign.
2. **W2.3** — replace `ExperienceCardsProps` and `ProjectCardsProps` with one discriminated union carrying `kind: 'project' | 'experience'`. Add the discriminant to the corresponding entries in `data.ts`. Retype `cardDataMap` in `DefaultCardGroup.tsx:11-33` so the lookup type-checks **without** the cast.
3. **W2.7** — delete the duplicated `scrollbar-width` / `scrollbar-color` declarations from `DefaultCardGroup.css`. Keep exactly one copy, in the block that wins today (verify by checking which block comes later in the cascade and whether any other rule overrides either property).

### Constraints

- **This is a move, not a redesign.** Do not add, remove, rename, or change the type of any field while relocating it. Do not "improve" optionality.
- **Do not rename any data field** — `link`, `title`, `image`, `description`, `yearPeriod`, `cardType` and the rest are consumed by `DefaultCard`, `ProjectCardInfo`, `ExperienceCardInfo`, W4.6's alt rewriting, and W4.10's `<time>` mapping.
- **Do not change any of the 14 card values.** Same titles, same descriptions, same links, same images, same `yearPeriod` strings.
- **Do not add `@ts-expect-error` or `@ts-ignore`** to make the union work. If it does not type-check, the union shape is wrong.
- **Do not rename the `.group-cards` class.** W2.7 deletes a *declaration*, not the class.
- **Do not touch `data.ts:27-29`** (`description: string[]`) — that is W2.9, TASK-12.
- **Do not touch `data.ts:177-277`** (the 8 project entries' alt text) — that is W4.6/W4.3, TASK-19.
- **Do not add runtime code to the data layer.** The types module must contain types only, or the module graph gains a cycle risk.
- **Do not introduce a `kind` value that is not `'project'` or `'experience'`.** The union must be exhaustive.

### Validation Rules

- `tsc -b` must pass with zero errors — this task is a **type-level** refactor, so the type-checker is the primary verification tool.
- Grep `data.ts` for `from "../components` → expect zero hits. This is the definitive test for W2.1.
- The 14-card grid screenshot diff at all 13 widths is blocking — it is the largest visual region and the union change touches every card's data path.
- Verify the `cardDataMap` lookup returns the right component for both `kind` values by exercising both card groups on `/` (the project section and the experience section).
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **Type-only imports can become runtime imports.** With `isolatedModules` / `verbatimModuleSyntax` (check `tsconfig*.json`), a plain `import { Props } from "..."` for a type is erased only if the compiler can tell. Prefer explicit `import type`. If a type-only module ends up importing a component at *runtime*, the data layer's dependency inversion is broken even though the types "moved".
- **Moving a type into `src/data/` can create a cycle.** If a component imports its type from `data/types.ts` and `data.ts` imports from `data/types.ts`, there is no cycle — but if `data/types.ts` imports anything from a component, the cycle returns. Verify the final module graph has `components → data/types` and never `data/types → components`.
- **The discriminated union may break `HomePageScroll.tsx:123-131`.** `feasibility.md` flags that this site passes `cardType`/`image`/`title`/`description` to a `Component` union that includes `ContactForm: () => JSX.Element` — resolved today only by TypeScript's JSX prop leniency. A stricter union may turn that latent typing hole into a **compile error**. This is expected and is the point of the task: fix it by typing the union properly, not by loosening the new types. If it surfaces, treat it as in scope (it is the same file's typing defect) and record it.
- **Both card groups must keep rendering.** A discriminated union that narrows too aggressively can make one `kind` unreachable at the type level while still rendering at runtime. Exercise both sections on `/`.
- **`kind` is a new field in the data.** It is internal and never rendered — but confirm no card consumer spreads its props onto a DOM element (which would emit `kind="project"` as an unknown attribute). `DefaultCardGroup.tsx` may spread `cardData`; if so, exclude `kind`.
- **`data.ts` also holds non-exported interfaces** (`W2.14`, TASK-12) at `:32,171`. TASK-12 removes those. Do not pre-empt it here — but be aware that W2.3's union rewrite overlaps those exact lines. **Coordinate:** this task's W2.3 edit and TASK-12's W2.14 edit both target `data.ts:32,171`. TASK-12 must re-read the file after this task, not apply a pre-written patch.
- **The duplicate `.group-cards` block may not be a pure duplicate.** Check whether `:1-13` and `:15-18` declare the properties with different values or different vendor prefixes. If they differ, deleting one is a **behaviour change**, not a cleanup. Verify the surviving declaration is the one whose value wins today.
- **`scrollbar-width` / `scrollbar-color` are progressive enhancements.** Firefox honours them; Chrome/Safari do not. A wrong "winner" is invisible in Chrome-based verification. Check the values against the intent, not just against the last-declared-wins rule.
- **Data file size.** `data.ts` is the largest file touched. Any typo in a card entry is a content regression. Diff the *values*, not just the structure — compare the rendered strings.
- **The new types module's location matters for W4.14 (TASK-19).** W4.14 requires the data layer to be **serialisable** — no component instances, no DOM references. Types are erased at build time and therefore serialisable by construction. Do not add runtime values to the types module, or prerender-readiness regresses.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep `src/data/data.ts` for `components` imports → expect zero hits.
- [ ] `npx tsc -b` → zero errors, zero new suppression comments.
- [ ] Render `/` and compare every card's title, description, link, image and `yearPeriod` against the pre-refactor render.
- [ ] Exercise both card groups (projects and experience) — confirm the correct component renders for each `kind`.
- [ ] Verify no `kind` attribute leaks into the DOM.
- [ ] Screenshot diff of the card grid at all 13 widths.
- [ ] Grep `DefaultCardGroup.css` → exactly one `scrollbar-width`, one `scrollbar-color`.
- [ ] Check Firefox rendering of the scrollbar colours (the properties are Firefox-only).
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; the card-grid 13-width diff is ~7 min, the type/data checks ~10 min.

---

## Definition of Done

- [ ] `data.ts` imports nothing from `src/components/`
- [ ] Data-facing prop types live in one types module
- [ ] `ExperienceCardsProps` + `ProjectCardsProps` collapsed into one discriminated union
- [ ] `as keyof typeof` cast removed from `DefaultCardGroup.tsx`
- [ ] Duplicate `.group-cards` block removed
- [ ] All 14 card entries and 6 recognitions unchanged
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W2.1, W2.3, W2.7; Stage 2 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 rows (W2.3 "Enables W1.10's cast removal and W1.11's total lookup"; W2.4 depends on W2.1; W2.8 depends on W2.3)
- `.opencode/plan/codebase-quality-seo/risks.md` — R21, T7, D5
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 1 (`data.ts`, 6 items, serialise all)
- `src/data/data.ts:1-4, 27-29, 32-35, 171-174`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:11-33`
- `src/components/DefaultCardGroup/DefaultCardGroup.css:1-13, 15-18`