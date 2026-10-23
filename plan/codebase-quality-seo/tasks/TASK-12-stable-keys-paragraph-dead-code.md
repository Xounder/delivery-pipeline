# Task — Stable List Keys, AboutMe Paragraph Rendering & Dead Code/Asset Removal

## Task Information

### ID

TASK-12

### Title

Replace index keys with domain keys, render descriptions as paragraphs, delete dead code and the unused asset

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 2 — Organisation, DRY & KISS

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W2.8** | Medium | Replace index keys with stable domain keys (`link`, `title`, icon identity, text); guard against duplicates | `src/components/DefaultCardGroup/DefaultCardGroup.tsx:42`, `src/components/ProjectCardInfo/ProjectCardInfo.tsx:18`, `src/components/MenuBar/MenuBar.tsx:35`, `src/components/AboutMe/AboutMe.tsx:23` |
| **W2.9** | Medium | Replace the array-of-one-plus-`<br><br>` hack with a proper paragraph render | `src/components/AboutMe/AboutMe.tsx:1, 21-29`, `src/data/data.ts:27-29` |
| **W2.14** | Low | Delete dead code and dead assets: `me.png`, `React` default import under the automatic JSX runtime, unused `React.Fragment`, the stray `</Link>;` semicolon, non-exported interfaces in `data.ts`, the `return`-of-`void` in `navigateToStart` | `src/assets/images/me.png`, `src/components/AboutMe/AboutMe.tsx:1,23`, `src/components/MenuBar/MenuBar.tsx:40,61`, `src/data/data.ts:32,171` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change. **For W2.8, every card/menu item must still be clickable and must still navigate to the same target.**
2. **Visual branding** — rendered output must be pixel-identical. **For W2.9, the rendered text and line breaks must be identical** — `<br><br>` and a `<p>` render differently in flow.
3. **Copy** — all visible text unchanged. **The `description` strings in `data.ts` are data, not copy** — W2.9 changes how they render, not what they say.
4. **Routing** — `/` and `/scroll` only. No new routes. Card link targets are unchanged.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **W2.8 changes React `key` props, which are not DOM attributes — but the DOM element names stay.**

**Required by TASK-08's Stage 1 work:** the re-levelled heading at `AboutMe.tsx:20` must survive. W2.9 rewrites `:21-29`, immediately below it.

**Required by TASK-11's Stage 2 work:** the consolidated button structure and TASK-05's `type="button"` at `MenuBar.tsx:61` must survive. W2.14 edits the same lines.

---

## Description

Three DRY/KISS cleanups.

**W2.8 — index keys.** Four lists use the array index as `key`: `DefaultCardGroup.tsx:42`, `ProjectCardInfo.tsx:18`, `MenuBar.tsx:35`, `AboutMe.tsx:23`. Index keys are unstable across reorders and cause React to remount — losing focus, animation state, and (in this app) potentially framer-motion state. Replace with domain keys.

The duplicate-key hazard is documented: `logoDio` appears **3×** and `logoAPI` **3×**, so an image-URL key would collide. `feasibility.md` confirms **`link` is unique for all 14 cards** and is therefore safe. Menu icons must use *icon identity* (name), not the asset URL, for the same reason.

**W2.9 — array-of-one-plus-`<br><br>`.** `description: string[]` holds exactly one string, and `AboutMe.tsx:21-29` maps it, appending two `<br>` elements after each entry. A single-element array rendered with `<br><br>` is a layout hack. Render a proper `<p>`.

**W2.14 — dead code and assets.** Six independent deletions: the never-imported `me.png`; the `React` default import under the automatic JSX runtime; the unused `React.Fragment`; a stray `</Link>;` semicolon; two non-exported interfaces in `data.ts`; and a `return`-of-`void` in `navigateToStart`.

---

## Acceptance Criteria

- [ ] Zero index keys remain. Grep for `key={i}`, `key={index}`, `key={idx}` across `src/` → expect zero hits.
- [ ] `DefaultCardGroup.tsx:42` keys cards by a domain field. `link` is used (unique across all 14 cards) or an equally unique combination.
- [ ] `ProjectCardInfo.tsx:18` keys by a domain field, not an image URL. Verified: no duplicate-key React warning for the three projects sharing `logoAPI`.
- [ ] `MenuBar.tsx:35` keys menu icons by icon identity (name), not asset path. Verified: no duplicate-key warning for icons sharing an asset.
- [ ] `AboutMe.tsx:23` keys description paragraphs by text.
- [ ] **Zero React duplicate-key warnings** in the console on both routes.
- [ ] Cards still navigate to exactly the same targets. Exercise every one of the 14 cards.
- [ ] `description: string[]` renders as proper paragraph markup. Rendered text and line breaks are byte-identical to the `<br><br>` version at all 13 widths.
- [ ] The single-string-per-array shape is either preserved as data or the type narrowed to match reality — and the choice is recorded. **Do not silently change the data shape**: `data.ts:27-29` is consumed by W4.3 (JSON-LD) and W4.6 (alt) in TASK-19.
- [ ] `src/assets/images/me.png` is deleted. Grep confirms zero references.
- [ ] `React` default import removed from `AboutMe.tsx:1` (and anywhere else it is unused under the automatic JSX runtime).
- [ ] `React.Fragment` usage removed where unused.
- [ ] The stray `</Link>;` semicolon at `MenuBar.tsx:61` is removed — **without restructuring the `<Link>`-wrapping-`<button>` nesting** (OOS-4).
- [ ] The `return`-of-`void` in `navigateToStart` is fixed so the function's control flow is correct.
- [ ] Non-exported interfaces at `data.ts:32,171` removed (TASK-09 must have already replaced them via W2.3 — re-verify rather than double-delete).
- [ ] Screenshot diff clean at all 13 widths.
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-09
- TASK-10
- TASK-11
- Stage 1 exit criteria

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **Stage 1 exit** — `index.md` Stage 2 entry criteria.
- **TASK-09** — `feasibility.md`: *"W2.8 Depends on W2.3."* The discriminated union defines the fields available as keys, and `DefaultCardGroup.tsx`'s typing changed. **Also:** TASK-09 already rewrote `data.ts:32,171` for W2.3 — W2.14's non-exported-interface deletion must **re-read** the file rather than apply a pre-written patch.
- **TASK-10** — owns `MenuBar.tsx:8-13,40,61` for W2.11. This task edits `MenuBar.tsx:35,40,61`. **Same file, overlapping lines at `:40,61`.** Sequential.
- **TASK-11** — owns `ProjectCardInfo.tsx:19-23,28-32` for W2.4. This task edits `ProjectCardInfo.tsx:18`. **Adjacent lines.** Sequential.

**This is the tail task of Stage 2.** It depends on three earlier Stage 2 tasks and is the most contended task in the stage.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/data/data.ts` | **6 — serialise all (R21)** | `:27-29`, `:32`, `:171` | TASK-09 (done), **TASK-19** (W4.3, W4.6, W4.10, W4.16 — Stage 4) | **Strictly sequential.** Order: TASK-09 → **TASK-12** → TASK-19. **R21: never two editors.** |
| `src/components/MenuBar/MenuBar.tsx` | 5 | `:35`, `:40`, `:61` | TASK-05, TASK-07, TASK-08 (done), **TASK-10** (`:8-13,40,61`), **TASK-13** (W2.17 `:15-19`), TASK-17, TASK-20 (`:29,32`) | **Sequential.** **`:40,61` is the plan's highest-contention pair** — TASK-10 (W2.11 route derivation), this task (W2.14 stray semicolon), TASK-05 (W1.19 `type`). Verify after every one of them that the others' changes survive. |
| `src/components/DefaultCardGroup/DefaultCardGroup.tsx` | 4 | `:42` | TASK-06 (done), TASK-09 (done), TASK-17 (W3.8 `:38`) | **Sequential.** `:38` and `:42` are 4 lines apart — adjacent. |
| `src/components/ProjectCardInfo/ProjectCardInfo.tsx` | 2 | `:18` | TASK-11 (W2.4 `:19-23,28-32`) | **Sequential.** Adjacent lines. |
| `src/components/AboutMe/AboutMe.tsx` | 4 | `:1`, `:20-29`, `:23` | TASK-08 (W1.17 `:20`, done), TASK-20 (W4.7 `:17`) | **Sequential.** **`:20` is TASK-08's `h1`/`h2` — verify it survives** the `:21-29` rewrite. |
| `src/assets/images/me.png` *(deleted)* | 1 | deletion | — | n/a |

---

## Technical Context

### Relevant Components

- `DefaultCardGroup` — the 14-card list
- `ProjectCardInfo` — per-project icon list
- `MenuBar` — menu icon list
- `AboutMe` — the description paragraph render

### Relevant Modules

- `src/data/data.ts:27-29, 32, 171`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:42`
- `src/components/ProjectCardInfo/ProjectCardInfo.tsx:18`
- `src/components/MenuBar/MenuBar.tsx:35, 40, 61`
- `src/components/AboutMe/AboutMe.tsx:1, 20-29`
- `src/assets/images/me.png`

### Relevant APIs

- React `key` prop semantics (reconciliation identity)
- JSX automatic runtime (why the `React` default import is unused)
- `<p>` block flow vs `<br><br>` line-break flow
- `navigateToStart` — scroll navigation with a `return`-of-`void`

### Relevant Types

- `description: string[]` in `data.ts:27-29`
- `AboutMeProps`, `ProjectCardInfoProps` (relocated by TASK-09)
- Non-exported interfaces at `data.ts:32,171`

---

## Implementation Guidance

### Expected Changes

1. **W2.8** — replace index keys:
   - `DefaultCardGroup.tsx:42` → key by `link` (unique across all 14 cards).
   - `ProjectCardInfo.tsx:18` → key by a field unique per entry. **Not** the image URL: `logoAPI` is shared by 3 projects.
   - `MenuBar.tsx:35` → key by icon identity/name. **Not** the asset path, for the same reason.
   - `AboutMe.tsx:23` → key by the paragraph text.
2. **W2.9** — replace the `<br><br>` map with a proper paragraph render. Confirm the visual result is byte-identical at all 13 widths, particularly widths 1–3 where line breaking is tightest.
3. **W2.14** — six deletions. For `navigateToStart`, the `return`-of-`void` must be corrected so control flow is right — read the whole function before changing it; it may be related to the scroll-restoration hack (OOS-3, `HomePage.tsx:36-53`), which is **not** in scope.

### Constraints

- **Never key by a duplicated value.** `logoAPI` × 3, `logoDio` × 3. `link` is unique for all 14 cards — use it.
- **Do not change the `description` data shape** unless the narrowing is recorded and verified against W4.3/W4.6 consumers. `feasibility.md` treats `data.ts` as the source of truth for JSON-LD; a shape change ripples to Stage 4.
- **Do not restructure `<Link>`-wrapping-`<button>`** at `MenuBar.tsx:61`. OOS-4 — delete only the stray semicolon.
- **Do not touch the scroll-restoration fake `<div>`** or its 500ms timer. OOS-3. Fix `navigateToStart`'s control flow only if it is genuinely independent.
- **Do not re-add a `React` default import** if the JSX runtime is automatic. Check `tsconfig*.json` for `"jsx": "react-jsx"` before removing.
- **Do not delete a file that becomes reachable.** Grep for `me.png` before deleting; it is believed unused, but verify.
- **Do not break TASK-08's heading** at `AboutMe.tsx:20`.
- **Do not break TASK-05's `type="button"`** at `MenuBar.tsx:61`.
- Do not add a runtime dependency or a key-generation library.

### Validation Rules

- **Zero duplicate-key React warnings** on both routes — this is the definitive test for W2.8.
- Exercise all 14 cards: each must navigate to its original target.
- Screenshot diff at all 13 widths, **widths 1–3 especially** for W2.9's line-break equivalence.
- `npm run lint` and `npm run build` green — and expect the build to **catch** the dead-code deletions if any reference survives.

---

## Edge Cases

- **`logoAPI` used for 3 projects, `logoDio` for 3 recognitions.** Keying by image URL produces duplicate keys and React silently drops items. This is the exact trap `feasibility.md` flags. Use `link` (unique per card). If an entry lacks a `link`, use `link` + `title`.
- **Stable keys change remount behaviour.** Today, index keys mean React reuses DOM nodes by position. After the change, React matches by identity. If any component holds focus or framer-motion state, it will now be *correctly* preserved across a reorder — which means a visible behaviour **change** on hover during a reorder. Verify card hover still works. This is an improvement, but it is a change, and it must be observed.
- **Reordering the data to "fix" duplicate keys would change content.** Do not reorder `data.ts` entries.
- **Keys must be unique across the whole list, not per sub-group.** `ProjectCardInfo` may render several lists on one page (8 projects across multiple groups). Keys must be unique within each list React renders, which may be a subset. Verify.
- **W2.9's `<p>` may add default margins.** `<br><br>` contributes line breaks with no margins; `<p>` has `margin-block` by default. **Compensate in CSS** so the rendered spacing is identical. This is the most likely regression in the task.
- **`<p>` cannot contain block-level children.** If a description ever needs markup, wrapping it in `<p>` would be invalid. Today's strings are plain text, so this is fine — but do not introduce markup later without revisiting.
- **`description: string[]` holding exactly one string** — the array is a fiction. Narrowing the type to `string` would be the honest fix, but it ripples to W4.3/W4.6/W4.10 in TASK-19. Either narrow it now and notify, or keep the array and render `description[0]`. **Record the choice.**
- **Removing the `React` default import** breaks code that uses `React.Fragment`, `React.createElement`, or a `React.FC` type annotation. Grep for `React.` in each file before removing the import. TASK-08's h1 work may have added a `React` reference.
- **JSX runtime config must be checked, not assumed.** If `tsconfig.json` still has `"jsx": "react"`, removing the default import breaks the build. Verify before editing.
- **The stray `</Link>;` semicolon** suggests the surrounding JSX was hand-edited. Read the whole block — there may be a related defect. Fix only what is in scope; record anything else found.
- **`navigateToStart`'s `return`-of-`void`** may be intentional early-return control flow. "Fixing" it could change scroll behaviour. Read the whole function and its call sites; if the fix has any behavioural effect on navigation, that exceeds W2.14's "delete dead code" scope — record and escalate rather than guess.
- **`me.png` deletion and the import graph.** If any file imports it via a path that grep misses (dynamic import, CSS `url()`), the build fails. Run `npm run build` after deleting.
- **`MenuBar.tsx:40,61` merge risk.** Three tasks have edited these lines. After this task, re-verify: `type="button"` present (TASK-05), route derived not compared (TASK-10), no stray semicolon (this task).
- **`AboutMe.tsx:20-29` merge risk.** TASK-08 changed the tag at `:20`; this task rewrites `:21-29`. Verify the heading survives with its TASK-08 compensation CSS.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep `src/` for index keys (`key={i}`, `key={index}`) → expect zero hits.
- [ ] Console check on both routes → zero React duplicate-key warnings.
- [ ] Exercise all 14 cards → each navigates to its original target; hover still animates.
- [ ] `AboutMe` description: screenshot diff at widths 1, 2, 3 and 11 (line-break equivalence).
- [ ] Grep `src/` for `me.png` → expect zero references; then confirm the build still passes.
- [ ] Grep each edited file for `React.` before/after the import removal.
- [ ] Verify `MenuBar.tsx` still has `type="button"` (TASK-05) and route-derived destination (TASK-10), plus no stray semicolon (this task).
- [ ] Verify `AboutMe.tsx:20` heading tag and computed size (TASK-08).
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; ~15 min for the card exercise and greps, ~7 min for the 13-width diff.

---

## Definition of Done

- [ ] Zero index keys; zero duplicate-key warnings
- [ ] Keys are domain-based and unique per list
- [ ] All 14 cards navigate to their original targets
- [ ] `description` rendered as paragraphs, byte-identical output
- [ ] Data-shape decision for `description: string[]` recorded
- [ ] `me.png` deleted and unreferenced
- [ ] Unused `React` default import removed
- [ ] Unused `React.Fragment` removed
- [ ] Stray `</Link>;` semicolon removed, nesting unchanged (OOS-4)
- [ ] `navigateToStart` control flow corrected
- [ ] Non-exported `data.ts` interfaces removed
- [ ] TASK-05, TASK-08, TASK-10, TASK-11 work preserved
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W2.8, W2.9, W2.14; Stage 2 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 2 rows (W2.8 "Depends on W2.3"; W2.9 "Verify line breaks at widths 1–3")
- `.opencode/plan/codebase-quality-seo/risks.md` — R21 (`data.ts` serialisation), OOS-3, OOS-4, OOS-5
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 1 (`data.ts`), Tier 3 overlap maps
- `src/data/data.ts:27-29, 32, 171`
- `src/components/DefaultCardGroup/DefaultCardGroup.tsx:42`
- `src/components/ProjectCardInfo/ProjectCardInfo.tsx:18`
- `src/components/MenuBar/MenuBar.tsx:35, 40, 61`
- `src/components/AboutMe/AboutMe.tsx:1, 20-29`
- `src/assets/images/me.png`