# Task — HomePageScroll Observer Lifecycle, Dead DOM-Poke & Footer Suspense

## Task Information

### ID

TASK-03

### Title

Fix the self-retriggering IntersectionObserver, delete the dead DOM-poking block, unwrap the footer Suspense

### Owner

senior-frontend

### Status

Pending

### Stage

Stage 1 — Toolchain Baseline & Correctness Fixes

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W1.4** | Critical | `useEffect` must not depend on `currentSection`; observer created once | `src/pages/HomePageScroll/HomePageScroll.tsx:43-83` (dep array `:83`) |
| **W1.5** | Critical | Delete dead block querying `.atual-section__name h2` / `.atual-section__icons` | `src/pages/HomePageScroll/HomePageScroll.tsx:58-68` |
| **W1.12** | High | Remove the `Suspense` wrapper around static footer text | `src/pages/HomePageScroll/HomePageScroll.tsx:145-148` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. Same colours (`#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`), same fonts, same border-radius, same shadows, same spacing, same z-index stacking, same hover states. Token introduction is a *rename* of existing values, never a value change.
3. **Copy** — all visible text, `alt` strings, `title`, `pattern`, JSON-LD field wording are unchanged.
4. **Routing** — `/` and `/scroll` only. No new routes. `vercel.json`'s SPA rewrite stays.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content__icons`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed.

**Explicitly protected by OOS-2:** `HomePageScroll`'s progressive per-section reveal (`HomePageScroll.tsx:104-135`) — the `visibleSections.has(key)` gate and the `y: 300` entry animation — is **accepted out of scope and must not change**. Only the observer *lifecycle* (which is a defect independent of the reveal design) and the dead code around it are in scope.

---

## Description

Three defects in one file, one concern each, all at `src/pages/HomePageScroll/HomePageScroll.tsx`.

**W1.4 — self-retriggering observer.** `useEffect(..., [currentSection])` at `:83` declares `currentSection` as a dependency, but `currentSection` is *written inside the observer callback* at `:51`. Every section change therefore:

1. tears down the `IntersectionObserver`,
2. re-runs `document.querySelectorAll("section")` at `:79`,
3. re-creates the observer,
4. fires the callback again.

This is a loop. `feasibility.md` confirms an empty dependency array is sufficient: the callback writes state and reads the latest value through closure on each tick, so no ref is needed.

**W1.5 — dead DOM-poking block.** `:58-68` runs `document.querySelector('.atual-section__name h2')` and `document.querySelector('.atual-section__icons')`. **Neither class exists in the codebase** — the real classes are `.atual-section__content h2` and `.atual-section__content__icons` (invariant 5). Both queries return `null`; the `if` guard silently swallows it. The block also reaches outside the component via `document.querySelector`, which is the same coupling class as the T4 defect. `MenuBar.tsx:53-54` already owns this state through React (`isVisible`).

**W1.12 — pointless `Suspense`.** `:145-148` wraps static footer text in a `Suspense` boundary with no lazy component underneath. It delays indexable content for no benefit.

`risks.md` **T3** mitigation is explicit: *"W1.4 (empty dependency array) must land with W1.5 in the same change — removing the dead `querySelector` block (`:58-68`) is part of the same file and the same concern."* Hence one task.

---

## Acceptance Criteria

- [ ] `useEffect` at `:43-83` has a dependency array that does **not** include `currentSection`.
- [ ] The observer is constructed exactly once per mount. Verified: no repeated `new IntersectionObserver(...)` during a full scroll of `/scroll` (instrument with a temporary counter or DevTools allocation tracking, then remove the instrumentation).
- [ ] The dead block at `:58-68` is deleted in full — both `document.querySelector` calls and the `if` guard.
- [ ] No remaining `document.querySelector` in `HomePageScroll.tsx` that targets a class outside the component's own subtree. (`document.querySelectorAll("section")` at `:79` is **still present** — it is replaced by a container ref in TASK-10, W2.12, not here.)
- [ ] The `Suspense` wrapper at `:145-148` is removed; the footer text renders as direct children.
- [ ] The footer text renders identically — same text, same spacing, same position. Screenshot diff clean at all 13 widths.
- [ ] The progressive reveal is unchanged: a section still animates in from `y: 300` only after `visibleSections.has(key)`, with the same timing.
- [ ] `threshold: 0.5` at `:76` is unchanged. (`threshold` evaluation is W4.17, TASK-21 — a documentation-only note.)
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.
- [ ] Zero uncleared timers and zero repeated observer construction introduced (Stage 1 exit criterion).

---

## Dependencies

### Required Tasks

- TASK-01

### Dependency Notes

TASK-01 is required because `npm run build` / `npm run lint` cannot run until the self-referential dependency is removed.

**No dependency on TASK-02.** These tasks touch disjoint files (`HomePageScroll.tsx` vs `index.css` / `HomePage.css`).

**Downstream (not blocking, but must be planned for):**

- **TASK-07** (W1.11) edits `HomePageScroll.tsx:96` — the same `iconsData[sections.indexOf(...)]` lookup family. Sequenced **after** this task.
- **TASK-10** (W2.2, W2.12) replaces `document.querySelectorAll("section")` at `:79` with a scoped container ref and rewrites the inline section array at `:104-110`. Sequenced after this task (Stage 2).
- **TASK-08** (W1.17) edits the footer heading at `:146` — adjacent lines. Sequenced after this task.
- **TASK-21** (W4.17) documents `threshold: 0.5` at `:76`. Documentation only, and a different line range — but **Stage 4 cannot start until Stage 3 exits, so this is an ordering constraint from the stage boundary, not a parallel opportunity.**

---

## File Overlap Warning

`src/pages/HomePageScroll/HomePageScroll.tsx` is a **max-1-concurrent-editor** file (9 work items across the plan: W1.4, W1.5, W1.11, W1.12, W2.2, W2.12, W3.10, W4.13, W4.17).

| Other task | W-IDs | Lines touched | Merge strategy |
|---|---|---|---|
| **TASK-07** | W1.11, W1.20 | `:96`, `:146-147` | **Sequential** — TASK-07 runs after this task. |
| **TASK-08** | W1.17 | `:146` | **Sequential** — adjacent to W1.20's `:146-147`. Runs after this task. |
| **TASK-10** | W2.2, W2.12 | `:79`, `:104-110` | **Sequential** — Stage 2, after Stage 1 exit. Rewrites the effect this task fixes. |
| **TASK-17** | W3.10 | `:92`, `:117`, `:143` | **Sequential** — Stage 3. |
| **TASK-22** | W4.13 | `:12` (import) | **Parallel is unsafe** (import statement vs effect body). Sequential — Stage 4. |
| **TASK-21** | W4.17 | `:76` | **Parallel with non-overlapping sections** — a comment/note at `:76` vs effect changes at `:43-83`. Permitted because Stage 4 batches are otherwise independent; if in doubt, serialise. |

No other Stage 1 task touches `HomePageScroll.tsx`, so within Stage 1 this task is parallel-safe with TASK-02, TASK-04, TASK-05, TASK-06.

---

## Technical Context

### Relevant Components

- `HomePageScroll` (`src/pages/HomePageScroll/`) — the scroll-driven page

### Relevant Modules

- `src/pages/HomePageScroll/HomePageScroll.tsx:43-83` — the observer effect
- `src/pages/HomePageScroll/HomePageScroll.tsx:58-68` — dead DOM-poke block
- `src/pages/HomePageScroll/HomePageScroll.tsx:145-148` — footer Suspense

### Relevant APIs

- `IntersectionObserver` — constructed at `:44`
- `document.querySelectorAll("section")` — `:79`
- `document.querySelector` — `:58-68` (to be deleted)
- React `Suspense` — `:145-148` (to be removed)
- `MenuBar` — already renders the visible-section state via `isVisible` (`MenuBar.tsx:53-54`)

### Relevant Types

- `currentSection` — component state, written inside the observer callback
- `visibleSections` — `Set<string>`, drives the reveal (OOS-2, unchanged)

---

## Implementation Guidance

### Expected Changes

1. **W1.4** — remove `currentSection` from the `useEffect` dependency array at `:83`. Keep the callback reading `currentSection` via closure; React re-creates the callback each render, and the effect only needs to run once. If the linter's `react-hooks/exhaustive-deps` complains, add a narrow inline suppression with a comment explaining the loop — **do not** re-add `currentSection` to satisfy the rule. (`feasibility.md` confirms this is sufficient.)
2. **W1.5** — delete `:58-68` entirely. Before deleting, verify by grep that `.atual-section__name` and `.atual-section__icons` appear nowhere else in the repo. Verify also that no CSS rule targets `.atual-section__name` / `.atual-section__icons`; `MenuBar.tsx:53-54` already applies the visible class through React state.
3. **W1.12** — remove the `Suspense` wrapper at `:145-148`, keeping the footer children in place with identical structure. Check whether the now-unused `Suspense` import (`:14-30` lazy imports are separate) becomes unused and remove only the `Suspense` specifier if so — the `lazy` imports at `:14-30` are used by TASK-22 and must survive.

### Constraints

- **Do not touch `threshold: 0.5` at `:76`.** The latent content-visibility risk it creates is accepted (OOS-2) and documented by W4.17 in TASK-21. Changing it would alter the reveal design.
- **Do not touch `:104-135`** — the progressive reveal and its `y: 300` animation. OOS-2.
- **Do not replace `document.querySelectorAll("section")` at `:79`.** That is W2.12 in TASK-10. Doing it here would create a Stage 1 → Stage 2 scope violation.
- **Do not rename any DOM class.** Invariant 5. The dead classes being deleted are the *wrong* ones; the real classes `.atual-section__content h2` / `.atual-section__content__icons` must keep working.
- **Do not add a ref.** `feasibility.md` rules a ref out as unnecessary; an empty dep array is sufficient.
- Do not change the observer `root`, `rootMargin`, or `threshold` options.
- Do not change animation timings, `stagger`, or the `y: 300` offset.

### Validation Rules

- Observer-construction count must be exactly 1 per mount of `HomePageScroll`. Verify by scrolling the full page and confirming no repeated construction. This is a Stage 1 exit criterion (`index.md`: "No `IntersectionObserver` re-creation on state change").
- Screenshot diff clean at all 13 widths — the footer must be byte-identical after removing `Suspense`.
- Verify the reveal still fires: scroll slowly from top to bottom and confirm each section animates in once, in order.
- Verify `/` → `/scroll` → `/` navigation still works (O4: route-to-route scroll leakage).
- `npm run lint` and `npm run build` green.

---

## Edge Cases

- **Unmount mid-intersection.** If the component unmounts while an intersection callback is queued, the callback must not call `setState` on an unmounted component. The fix (`useEffect` cleanup calling `observer.disconnect()`) must be present. Note that today's code already has a cleanup — confirm it still covers the new dependency array shape.
- **A section added or removed dynamically.** With a one-shot observer and a snapshot of `document.querySelectorAll("section")` taken at mount, a section added after mount is **never observed**. Today the effect re-runs on every `currentSection` change, which accidentally re-queries and partially masks this. After the fix, a dynamically added section would stay `opacity: 0; y: 300` forever (the T5 failure mode). The app's sections are static, so this is not a live bug — but record the behaviour change in the task's completion notes so TASK-10 (container ref) can address it deliberately.
- **`currentSection` set to an unknown value.** `dataset.section || "About Me"` at `:48` silently mislabels any foreign `<section>` in the document (risk **T4**). After the fix, a bad value does not cause a re-query loop, but it can still produce an `iconsData[-1] → undefined` crash at `:96`. That is W1.11 in TASK-07 — do **not** fix it here; note that this task makes the crash *more* reachable by removing the accidental re-query, which is why TASK-07 is sequenced next.
- **React 18 StrictMode double-invocation.** In development, StrictMode mounts effects twice. Confirm the observer is disconnected and re-created cleanly (2 constructions in dev StrictMode is expected; the production build must show 1).
- **The `Suspense` removal changes when footer text enters the DOM.** Today it is behind a boundary; after removal it renders synchronously. If the footer shifts up by the height of a previously-resolved fallback, that is a regression — check widths 1–3 where vertical space is tightest.
- **Removing `Suspense` may leave an unused import**, which fails `tsc -b` under `noUnusedLocals` if enabled. Check `tsconfig*.json` and remove the specifier only if genuinely unused. The `lazy(...)` calls at `:14-30` are used and must be kept (TASK-22 depends on them).
- **Grep finds `.atual-section__name` or `.atual-section__icons` in a CSS file.** That would mean the "dead" block was not dead — some rule targets classes nothing renders. Report it rather than deleting the block; that is a separate finding.
- **Full-page scroll on a fast machine** may change `currentSection` many times per second. With the loop removed, the callback now runs once per intersection change rather than being torn down and rebuilt — scroll smoothness should measurably improve (this is the `impact-analysis.md` "Scroll smoothness: Improve" claim). If it does not, the callback is capturing something that should be a ref.
- **Footer text is indexable content.** W1.12's stated benefit is that the static text is no longer delayed. Verify it appears in the rendered HTML after hydration, not only after a client-side state change.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Full scroll of `/scroll` top-to-bottom: observer constructed once (instrument temporarily, then remove).
- [ ] Screenshot diff clean at all 13 widths — footer region specifically.
- [ ] Reveal animation still fires once per section, in order, with unchanged timing.
- [ ] `/` → `/scroll` → `/` navigation: no stuck scroll position, no blank section.
- [ ] Keyboard Tab through `/scroll`: footer text reachable, no focus trap.
- [ ] `npm run lint` and `npm run build` green.
- [ ] Grep confirms `.atual-section__name` / `.atual-section__icons` appear nowhere in the repo.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. Manual verification; cost ≈ 30s per width for the screenshot diff (13 widths ≈ 7 min) plus ~2 min for the scroll behaviour walkthrough.

---

## Definition of Done

- [ ] `currentSection` removed from the observer effect's dependency array
- [ ] Observer constructed exactly once per mount
- [ ] Dead DOM-poke block deleted in full
- [ ] Footer `Suspense` removed, footer visually identical
- [ ] Progressive reveal unchanged (OOS-2 respected)
- [ ] `threshold: 0.5` unchanged
- [ ] Screenshot diff clean at all 13 widths
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W1.4, W1.5, W1.12
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 1 rows for W1.4 / W1.5 / W1.12
- `.opencode/plan/codebase-quality-seo/risks.md` — T3 (observer lifecycle), T4 (global selector coupling), T5 (`threshold: 0.5`), OOS-2
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — Tier 2 overlap map (`HomePageScroll.tsx`, 9 items)
- `src/pages/HomePageScroll/HomePageScroll.tsx:43-83, 58-68, 76, 79, 145-148`