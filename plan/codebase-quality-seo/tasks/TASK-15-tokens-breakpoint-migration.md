# Task — Token Foundation & Breakpoint Migration

## Task Information

### ID

TASK-15

### Title

Declare the 5 breakpoint tokens and layout/colour tokens in `:root`, migrate all 28 media queries, delete the 6 order-dependent landscape blocks, tokenise the 10 layout literals

### Owner

senior-frontend

### Status

In Progress — CSS migration complete; visual acceptance not verified (browser checks prohibited by user)

### Stage

Stage 3 — Responsive & Design-System Consolidation

### Work Items Covered

| W-ID | Sev | Work item | Targets |
|---|---|---|---|
| **W3.1** | High | Declare the 5 breakpoint tokens and all layout/colour/radius tokens once in `:root` | `src/index.css:1-15` |
| **W3.2** | High | Migrate all 28 `@media` blocks to the token system; introduce `--bp-md: 768px` for tablet portrait; fix the overlapping `600px` pair | 12 CSS files |
| **W3.3** | High | Delete the 6 order-dependent `max-width:1000px and (orientation: landscape)` blocks; re-express intent with `max-height` + `--bp-sm` | `src/components/AboutMe/AboutMe.css:76`, `src/components/ContactForm/ContactForm.css:111`, `src/components/DefaultCard/DefaultCard.css:188`, `src/pages/HomePage/HomePage.css:76`, `src/components/MenuBar/MenuBar.css:117`, `src/components/MenuSectionButton/MenuSectionButton.css:79` |
| **W3.4** | High | Tokenise the 10 duplicated layout literals + the `calc(100vh - 100px - 60px)` expression | `src/pages/HomePage/HomePage.css:7,43,64,72,78`, `src/components/MenuBar/MenuBar.css:10,72,100,113,119,133`, `src/components/MenuSectionButton/MenuSectionButton.css:14` |

---

## Must Not Change (Project-Wide Invariants)

These five invariants from `index.md` apply to every work item in this task. A change that cannot satisfy all five is out of scope and must be re-specified.

1. **Functionality** — no behaviour, no interaction, no animation timing, no data, no link target, no form semantics change.
2. **Visual branding** — rendered output must be pixel-identical. **This is the task where invariant 2 is most at risk.** Every token is a *rename* of an existing value, never a value edit. The single approved exception is `--bp-md: 768px`, which introduces a breakpoint that did not previously exist (see below).
3. **Copy** — unchanged (no copy is touched by CSS tokenisation).
4. **Routing** — unchanged.
5. **DOM class names consumed by JS** — `.atual-section__content h2`, `.atual-section__content h2`, `data-section`, `.group-cards`, `section` element names, `#root` may not be renamed. **Do not rename any class while tokenising its rules** — the class names are the contract, the values are what moves.

**The `--bp-md: 768px` exception:** widths 6–8 (768×1024, 820×1180, 834×1194) are documented as **currently undesigned**. Introducing `--bp-md` means those widths will get rules they do not have today, so a diff there is *expected*. Every such diff must be reviewed and explicitly accepted, not waved through.

---

## Description

The plan's largest and highest-risk task. `feasibility.md` rates W3.2 **Large** and W3.4 **Medium**.

**W3.1 — token foundation.** Declare in `:root` (`index.css:1-15`): the 5 breakpoint tokens, header height, footer height, card width, the 4 purples, surface/overlay radii, and the neutral text colour.

**Platform limitation, mandatory handling:** CSS custom properties **cannot be used inside `@media` conditions** without `@custom-media`, which is not natively supported. `feasibility.md` is explicit: *"the 5 breakpoint values must also be documented in a comment block as the canonical source, while media queries reference the literal values."* This is a known limitation, not a blocker — but it means **the tokens are documentation and single-source-of-truth for tooling, not a functional indirection** at the media-query level.

**W3.2 — 28 media queries across 12 files.** Migrate to the 5-token system. Introduce `--bp-md: 768px` handling for tablet portrait (which does not exist anywhere today). Fix the 2 overlapping `max-width: 600px` / `min-width: 600px` pairs that both match at exactly 600px.

**W3.3 — 6 order-dependent blocks.** `max-width: 1000px and (orientation: landscape)` currently wins over `min-width: 600px` **purely by source order** (`risks.md` **R2** — likelihood High, impact High). Replace with `--bp-sm` width rules + `max-height` queries. Phone-landscape shortest common width ≈ 667px, so `--bp-sm` (600px) already covers those devices. **This is the single most likely source of regression in the entire plan** — `impact-analysis.md` flags widths 4, 5 and 10 explicitly.

**W3.4 — 10 duplicated layout literals.** Header `90px`/`100px`, footer `60px`, and the `calc(100vh - 100px - 60px)` expression, plus the `top: 75px/85px/45px` button offsets.

---

## Acceptance Criteria

- [ ] All 5 breakpoint tokens are declared once in `:root` and documented in a comment block as the canonical source (per the `@custom-media` limitation).
- [ ] Header height, footer height, card width, the 4 purples, surface/overlay radii and the neutral text colour are each declared **once** in `:root`.
- [ ] **Zero hardcoded header/footer height literals** remain outside `:root`.
- [ ] **All colour tokens this stage needs are declared in `:root`** — including the brand ramp, the header/footer surface tokens, and the neutral ramp used by W3.5.
- [ ] **No colour literal outside `:root` has been touched by this task** — literal replacement is W3.5 (TASK-16), which runs after this task. Grep confirms the literals are still present, unchanged.
- [ ] **Zero `max-width … orientation: landscape` blocks remain.**
- [ ] All 28 `@media` blocks reference one of the 5 documented breakpoint values.
- [ ] The overlapping `max-width: 600px` / `min-width: 600px` pair no longer both match at exactly 600px. Verified at 599px and 600px.
- [ ] `min-width: 768px` is now defined and exercised for tablet portrait.
- [ ] Every token value is **byte-identical** to the literal it replaces. Verified by grepping for each original literal → expect zero hits outside `:root`.
- [ ] `calc(100vh - 100px - 60px)` now derives from tokens.
- [ ] **Screenshots at all 13 widths are pixel-identical to the TASK-14 baseline**, for both routes, except for explicitly-reviewed diffs at widths 6–8 where `--bp-md` is newly introduced.
- [ ] Widths **4 (667×375), 5 (844×390) and 10 (1180×820)** are diffed **individually and explicitly** — `risks.md` R2's designated regression widths.
- [ ] Horizontal-overflow check (`scrollWidth > clientWidth`) passes at widths 1–5.
- [ ] Every diff is classified: **regression** (revert) vs **intended `--bp-md` introduction** (record and accept).
- [ ] `npm run lint` does not regress above the TASK-01 baseline; `npm run build` passes.

---

## Dependencies

### Required Tasks

- TASK-01
- TASK-02
- TASK-14

### Dependency Notes

- **TASK-01** — so `npm run lint` / `npm run build` can run.
- **TASK-02** — `feasibility.md`: W3.2 *"Depends on W1.7"*; W3.4 *"Interacts with W1.7/W1.8 — sequence after both."* The `dvh` root fix and the height-math reconciliation **must** already be in place, or this task's layout tokens will be built on the wrong box model. **Also: this task is the sole owner of `HomePage.css`'s `:16,17,68` region that TASK-02 just fixed — do not revert it.**
- **TASK-14** — the **hard gate**. R6: *"No Stage 3 task may start without it."* Without the baseline there is nothing to diff against, making R1/R2/R3/R4 undetectable.

**No dependency on TASK-09 through TASK-13** — this task is CSS-only and touches none of their files' logic.

---

## File Overlap Warning

| File | Plan items | This task | Other tasks | Merge strategy |
|---|---|---|---|---|
| `src/index.css` | 4 (W1.7, W3.1, W3.5, W4.15) | `:1-15` (`:root`) | TASK-02 (W1.7 `:26-27`, done), **TASK-16** (W3.5 colours), **TASK-21** (W4.15 `:2`) | **Sequential.** TASK-16 **must follow** this task per file (`impact-analysis.md`: *"W3.5 must follow W3.2 per file"*). |
| `src/components/DefaultCard/DefaultCard.css` | **7** | W3.2, W3.3 (`:188`), W3.4 | TASK-05 (W1.21 `:143`, done), **TASK-16** (W3.5 `:13,18,89`, W3.9, W3.11, W3.12) | **Sequential, strictly.** Max-1-concurrent-editor. |
| `src/components/MenuSectionButton/MenuSectionButton.css` | **9** | W3.2, W3.3 (`:79`), W3.4 (`:14`) | TASK-04 (done), TASK-16 (W3.5, W3.7, W3.12) | **Sequential, strictly.** Max-1-concurrent-editor. |
| `src/components/MenuBar/MenuBar.css` | 6 | W3.2, W3.3 (`:117`), W3.4 (`:10,72,100,113,119,133`) | TASK-16 (W3.5 `:79`, W3.13 `:73-74`) | **Sequential.** Max-1-concurrent-editor. |
| `src/pages/HomePage/HomePage.css` | 6 | W3.2, W3.3 (`:76`), W3.4 (`:7,43,64,72,78`), W3.11 | TASK-02 (done), TASK-16 (W3.5, W3.11) | **Sequential.** |
| `src/components/AboutMe/AboutMe.css` | 4 | W3.2, W3.3 (`:76`), W3.11 | TASK-16 (W3.5 `:48`, W3.11) | **Sequential.** |
| `src/components/ContactForm/ContactForm.css` | 4 | W3.2, W3.3 (`:111`), W3.11 | TASK-16 (W3.5, W3.11) | **Sequential.** |
| `src/components/DefaultCardGroup/DefaultCardGroup.css` | 3 | W3.2 | TASK-09 (W2.7, done), TASK-16 (W3.5 `:11,17,29`) | **Sequential** for W3.5; W3.2's media queries vs W2.7's deleted top-of-file lines were **parallel-safe** earlier, but W3.5 here is not. |
| `src/components/Button/Button.css` | 3 | W3.2 | TASK-04 (done), TASK-16 (W3.5 `:15`, W3.7 `:2`) | **Sequential.** |
| `src/components/InfoCard/InfoCard.css` | 3 | W3.2 | TASK-05 (W1.16, done), TASK-08 (read-only) | **Sequential.** |
| `src/components/ArrowSectionButton/ArrowSectionButton.css` | 1 | W3.2 | — | No overlap. |
| `src/components/DefaultSectionButton/DefaultSectionButton.css` | 1 | W3.2 | TASK-16 (W3.5 `:23`) | **Sequential** for W3.5. |
| `src/pages/HomePageScroll/HomePageScroll.css` | 1 | W3.2 | — | No overlap. |
| `src/components/ExperienceCardInfo/ExperienceCardInfo.css` | 0 in plan | not listed | — | No overlap. |

**This task owns 12 CSS files in Stage 3. TASK-16 must run strictly after it.**

---

## Technical Context

### Relevant Components

- Both routes — every component stylesheet is affected

### Relevant Modules

- `src/index.css:1-15` — `:root`
- 12 CSS files containing the 28 `@media` blocks
- The 6 order-dependent landscape blocks: `AboutMe.css:76`, `ContactForm.css:111`, `DefaultCard.css:188`, `HomePage.css:76`, `MenuBar.css:117`, `MenuSectionButton.css:79`

### Relevant APIs

- CSS custom properties (`var(--token)`)
- `@media` conditions — **cannot reference custom properties** (no native `@custom-media`)
- `orientation` media feature — to be **removed entirely**
- `max-height` media feature — the replacement mechanism

### Relevant Types

- Breakpoint tokens: `--bp-sm: 600px`, `--bp-md: 768px`, `--bp-lg: 1024px`, `--bp-xl: 1440px`, `--bp-2xl: 1920px`
- Layout tokens: header height (`90px`/`100px`), footer height (`60px`), card width
- Colour tokens: `#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`, plus the neutral text colour

---

## Implementation Guidance

### Expected Changes

1. **W3.1** — in `src/index.css:1-15`, declare:
   - The 5 breakpoint tokens, **plus a comment block documenting them as the canonical source** (because `@media` cannot consume them).
   - Header height, footer height, card width, the 4 purples, surface/overlay radii, neutral text colour.
   - `--bp-2xl: 1920px` only if a real defect is found at ≥1920.
2. **W3.2** — for each of the 28 `@media` blocks: replace the literal with the documented token value (a comment reference or a literal matching the token), and introduce `--bp-md: 768px` handling for tablet portrait. Resolve the 600px overlap so exactly one rule matches at 600px.
3. **W3.3** — for each of the 6 landscape blocks: read its declarations, express the same intent as a `--bp-sm` width rule plus a `max-height` query, then delete the original. **Verify widths 4, 5 and 10 after each file.**
4. **W3.4** — replace the 10 layout literals and the `calc(100vh - 100px - 60px)` expression with token references.

### Constraints

- **Every token value must be byte-identical to the literal it replaces.** `risks.md` R1. A token that "rounds" a value is a regression.
- **Media queries must use literal values** with the token documented in a comment — they cannot use `var()`. Do not add `@custom-media` (not natively supported; would need a build plugin, which is out of scope).
- **Do not introduce `--bp-2xl` speculatively.** Only if a real defect appears at 1920×1080.
- **Do not remove `orientation: landscape` from any query that is not one of the 6 named blocks.** If you find a seventh, stop and report — it is not in the plan.
- **Do not touch `HomePage.css`'s `:16,17,68` height math** (TASK-02's W1.8). Tokenise *around* it.
- **Do not touch any colour outside `:root`** — that is TASK-16 (W3.5), and `#646cff` is separately gated there.
- **Do not change the `@media` count** unless a block is genuinely merged. The count is a verification anchor: 28 → 28 (minus the 6 deleted landscape blocks).
- **Do not consolidate `:hover` blocks** (W3.9, TASK-16).
- Do not introduce `vh`-simplification (W3.11, TASK-16).
- Do not change any `padding`/`margin`/`height` value, only its source.

### Validation Rules

- **The 13-width screenshot diff against the TASK-14 baseline is blocking** (`risks.md` R22). Diff after **every file**, not just at stage exit (R1's mitigation: *"Diff after every file, not just at stage exit"*).
- **Widths 4, 5 and 10 must be diffed individually.** R2's designated regression widths. A clean aggregate is not sufficient.
- **Widths 6–8 diffs are expected** (`--bp-md` is new). Each must be reviewed and classified as intended, with the reason recorded.
- Horizontal-overflow check at widths 1–5 is blocking.
- Grep each original literal (e.g. `100px`, `60px`, `#220751`) → expect zero hits outside `:root`.
- Grep for `orientation: landscape` → expect zero hits.
- Verify 599px and 600px individually (the overlap fix).
- **R1's mitigation requires per-file diffing.** A 12-file change reviewed as one diff will hide the regression.

---

## Edge Cases

- **CSS custom properties cannot be used in `@media`.** This is the defining constraint of the task. The tokens are documentation plus a `var()` source for *declarations*, while media queries carry literals. A reviewer may expect `var(--bp-sm)` in a media query and "correct" it — which breaks the stylesheet. Document the limitation prominently in the comment block.
- **Media queries and specificity are order-independent; declarations are not.** Replacing literals inside `@media` blocks does not change cascade behaviour. But **deleting and re-adding the 6 landscape blocks (W3.3) absolutely does** — they currently win by source order, not by specificity. Deleting them can promote a `min-width: 600px` rule that was previously overridden. This is R2 and it is the highest-probability regression in the plan.
- **The 6 landscape blocks are 6 independent decisions.** Do not treat them as one pattern. Read each one's declarations and re-express its specific intent. `ContactForm.css:111` and `MenuBar.css:117` may need different `max-height` thresholds.
- **`orientation: landscape` also matches desktop windows** (e.g. 1280×800). Removing the feature affects widths 11 and 13, not only phones. Verify desktop widths too — `impact-analysis.md` only flags 4, 5, 10 because those are where the *phone* intent lives.
- **A 1024×768 desktop window matches `max-width: 1000px and (orientation: landscape)`? No** — 1024 > 1000. But 900×700 does. The replacement rules must not over- or under-match.
- **Phone landscape shortest common width ≈ 667px.** `--bp-sm` (600px) covers widths 4 and 5. But `max-height` is needed for the *other* axis (375px and 390px tall). Getting the `max-height` threshold wrong either misses short-landscape phones or catches short-desktop windows.
- **The 600px overlap.** Today both `max-width: 600px` and `min-width: 600px` match at exactly 600px. Whichever wins depends on order. **The fix must preserve the current winner** — changing which rule wins at 600px is a behaviour change at one specific width, invisible in the 13-width matrix.
- **`--bp-md: 768px` styles widths that are currently undesigned.** There is no prior art for what tablet portrait should look like. Every rule added at 768px is a **design decision**. Keep `--bp-md` additions minimal and mechanical (extend the existing `--bp-sm` rules), or the branding invariant is at risk. `feasibility.md` notes the plan's goal is that 768–1023 is simply the un-restyled base plus what `--bp-sm` already does — **verify that is actually what happens before authoring new rules.**
- **Header height appears as both `90px` and `100px`.** They are *different values*, so they are not one token. Determine which applies where before tokenising, or you will create a token that silently changes one of them.
- **`calc(100vh - 100px - 60px)` depends on `HomePage.css`'s height fix (TASK-02).** If the height math changed to `dvh` or `box-sizing`, this expression's meaning changed too. Re-derive it, do not copy it.
- **`top: 75px/85px/45px` offsets** are relative to the header. If the header height token is wrong by even 1px, every button in the menu shifts. Verify at widths 1–13.
- **Token names may collide with existing custom properties.** Grep for `--` in `src/index.css` before adding.
- **The Vite CSS pipeline** may reorder or minify rules. A diff that appears only in a production build (not dev) indicates an ordering change — investigate rather than accept.
- **Safari/WebKit** must be included for the `orientation`-removal verification; `risks.md` T8 warns Chrome-only is insufficient.

---

## Testing

### Unit Tests

- None. No test framework is configured (OOS-7).

### Integration Tests

- None.

### Manual Validation

- [ ] Grep `src/index.css` for `--bp-` → 5 tokens declared, each documented in the comment block.
- [ ] Grep all CSS for `orientation: landscape` → zero hits.
- [ ] Grep all CSS for each original literal (`100px`, `90px`, `60px`, `#220751`, `#1D0C3A`, `#2E1656`, `#2C1552`) → zero hits outside `:root`.
- [ ] **Per-file** screenshot diff at all 13 widths against the TASK-14 baseline, for both routes.
- [ ] Widths 4, 5, 10 diffed individually and explicitly.
- [ ] Widths 6, 7, 8 diffs reviewed and classified as intended `--bp-md` introductions.
- [ ] Horizontal-overflow check at widths 1–5.
- [ ] 599px and 600px spot-check.
- [ ] `min-width: 768px` confirmed present and exercised.
- [ ] Safari/WebKit pass if available.
- [ ] `npm run lint` and `npm run build` green.

### E2E Test Authoring Guidance

Not applicable — no test framework exists. **This is the task where the manual matrix is most expensive**: per-file × 13 widths × 2 routes. If diffing 12 files individually, budget ~45–90 min. If tooling can only do stage-level diffs, note that as a reduced-confidence deviation from R1's per-file requirement.

---

## Definition of Done

- [ ] 5 breakpoint tokens declared once and documented for the `@media` limitation
- [ ] Header/footer/card-width/radii/neutral-colour tokens declared once in `:root`
- [ ] Zero hardcoded header/footer height literals outside `:root`
- [ ] All required colour tokens declared in `:root`; no literal outside `:root` modified by this task (W3.5's replacement is TASK-16)
- [ ] Zero `orientation: landscape` blocks remain
- [ ] All 28 media queries reference the 5 documented breakpoints
- [ ] 600px overlap resolved with the current winner preserved
- [ ] `--bp-md: 768px` defined and exercised
- [ ] `calc(100vh - 100px - 60px)` derives from tokens
- [ ] Every token value byte-identical to the literal it replaced
- [ ] 13-width screenshot diff clean (except reviewed `--bp-md` diffs)
- [ ] Widths 4, 5, 10 individually verified
- [ ] Build passes
- [ ] Lint does not regress above the TASK-01 baseline
- [ ] Validation completed

### Implementation evidence (static-only; browser verification NOT VERIFIED)

- Current source started with **28** real `@media` rules (the `@media` mention in `index.css` is a comment). Six former orientation blocks are retained as six width + max-height rules; one 768px mechanical inheritance rule and one exact-600px compatibility rule make **30** real rules now. The old 1000px upper bound is retained deliberately to avoid applying compact treatment to 1001–1023px screens.
- All six compact rules require 600–1000px width and at most 600px height: AboutMe constrains image/copy; ContactForm compresses fields; DefaultCard compacts card/text; HomePage shortens header; MenuBar shortens header and moves its toggle together; MenuSectionButton shortens its menu list. This covers 667×375 and 844×390 while excluding 1180×820; 900×700 landscape previously matched but no longer receives compact styling. These decisions require visual review when browser checks are allowed.
- The 600px wide form rule remains the winner. The card group originally inherited mobile width constraints *and* the later desktop flow at exactly 600px; its exclusive mobile rule now ends at 599.98px, and a 600px-only compatibility rule preserves those mobile constraints alongside desktop flow. The 768px InfoCard rule explicitly carries forward the existing 600px card width without a new design.
- `npm run build` passes (three existing mixed dynamic/static import warnings); `npm run lint` reports the existing seven TSX a11y errors and zero warnings. No tests were run (no test script). No GUI/headless browser, server, screenshots, 13-width comparison, overflow check or Safari/WebKit check was run, per user constraint. TASK-14 baseline was explicitly skipped; no visual PASS is claimed.

---

## References

- `.opencode/plan/codebase-quality-seo/index.md` — W3.1, W3.2, W3.3, W3.4; the approved 5-token breakpoint table; the 13-width matrix; Stage 3 exit criteria
- `.opencode/plan/codebase-quality-seo/feasibility.md` — Stage 3 rows (W3.1's `@custom-media` note; W3.2 Large; W3.2 depends on W1.7; W3.4 sequences after W1.7/W1.8)
- `.opencode/plan/codebase-quality-seo/risks.md` — **R1**, **R2**, R3, R4, **R6**, R22, T1, T8
- `.opencode/plan/codebase-quality-seo/impact-analysis.md` — § *Non-performance risks introduced* (W3.2); § *Verification Plan*
- `src/index.css:1-15`
- `src/pages/HomePage/HomePage.css:7,16,17,43,64,68,72,76,78`
- `src/components/MenuBar/MenuBar.css:10,72,100,113,117,119,133`
- `src/components/MenuSectionButton/MenuSectionButton.css:14,79`
