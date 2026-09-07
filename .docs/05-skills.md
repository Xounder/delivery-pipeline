# Skills Description

All skills used by the `delivery-pipeline` live in `.opencode/skills/<name>/SKILL.md` and are loaded through the skill tool. They are organized into three groups: **orchestration**, **learning chain**, and **support**.

---

## Catalog

| Skill | Group | Purpose |
|-------|-------|---------|
| `jobfindr-pipeline` | Orchestration | Main orchestrator — full pipeline or direct task routing |
| `jobfindr-pipeline-next` | Orchestration | Resumes a stopped pipeline from `pipeline.yaml` |
| `learning-improvement` | Learning chain | Evaluates a completed session (DONE/WRONG/IMPROV/LEARN/NEXT) |
| `continuous-learning` | Learning chain | Proposes `.opencode/` doc updates based on learnings; requires user approval |
| `session-save` | Learning chain | Persists the session file in `.opencode/sessions/` |
| `codebase-analysis` | Support | Scans the codebase/docs and generates structural reports |
| `doc-audit` | Support | Audits `.opencode/` docs for duplicates and similarities |
| `06-branding` | Support | Maintains JobFindr branding guidelines (palette, typography, tokens) |

---

## Orchestration Skills

### `jobfindr-pipeline`

| | |
|---|---|
| Purpose | Orchestrate the development pipeline from scratch (Full Pipeline Mode) or route a direct task (Direct Task Mode) |
| Inputs | User requirement / `/start` command; existing `active.txt` + plan artifacts (for skip/resume decisions) |
| Outputs | Updated `pipeline.yaml`; planning artifacts; implemented + QA-approved code; commit; learning chain execution |
| When to use | Starting the pipeline or a direct task |
| When not to use | Pipeline already started (use `jobfindr-pipeline-next`) |
| Dependencies | Invokes the specialized agents (`Product Manager`, `Tech Lead`, `Senior Frontend`, `Senior Backend`, `QA Reviewer`) |
| Relationship | Parent of `jobfindr-pipeline-next`; ends by chaining to the learning skills |
| Allowed agents | Orchestrator only |

### `jobfindr-pipeline-next`

| | |
|---|---|
| Purpose | Continue a pipeline from where it stopped |
| Inputs | `pipeline.yaml`, `.opencode/plan/active.txt`, existing plan artifacts |
| Outputs | Resume the remaining phases until completion |
| When to use | A pipeline already started was interrupted |
| When not to use | First run (use `jobfindr-pipeline`) |
| Dependencies | Follows `jobfindr-pipeline` rules |
| Relationship | Continuation of `jobfindr-pipeline` |
| Allowed agents | Orchestrator only |

---

## Learning Chain Skills (STOP Hook)

The three skills below are an **atomic sequence** executed at the end of every completed implementation:

```text
learning-improvement → continuous-learning → session-save
```

### `learning-improvement`

| | |
|---|---|
| Purpose | Review the finished session and extract the evaluation |
| Inputs | Session history (tool calls, errors, results) |
| Outputs | Text evaluation: DONE, WRONG, IMPROV, LEARN, NEXT (5–7 lines) |
| When to use | End of a complete implementation (STOP hook / complex tasks) |
| When not to use | Trivial tasks; Direct Task Mode sessions without QA |
| Relationship | First of the chain — MUST hand off directly to `continuous-learning` |
| Allowed agents | Orchestrator (main agent) |

### `continuous-learning`

| | |
|---|---|
| Purpose | Decide which `.opencode/` docs to improve based on the evaluation |
| Inputs | Evaluation from `learning-improvement`; `.opencode/docs-catalog.md` |
| Outputs | Change plan (file, why, best location, modification) presented for approval; approved changes applied |
| When to use | Right after `learning-improvement` |
| When not to use | In isolation (never without the evaluation); never edits application source code |
| Constraints | Only `.md` files under `.opencode/`; always with explicit user approval |
| Relationship | Second of the chain — hands off to `session-save` |
| Allowed agents | Orchestrator (main agent) |

### `session-save`

| | |
|---|---|
| Purpose | Persist the session evaluation |
| Inputs | Evaluation from `learning-improvement` |
| Outputs | `.opencode/sessions/YYYYMMDD-HH-MM-<description>-session.tmp` (only the 2 most recent files retained) |
| When to use | Last step of the STOP hook |
| When not to use | Trivial Direct Task Mode sessions |
| Dependencies | `save-session` custom tool (`.opencode/tools/save-session.ts`) |
| Relationship | Last of the chain — after saving, the pipeline ends |
| Allowed agents | Orchestrator (main agent) |

---

## Support Skills

### `codebase-analysis`

| | |
|---|---|
| Purpose | Extract structural information from source and docs using tree-sitter (regex fallback) |
| Inputs | Project source (`.ts`/`.tsx`) or `.opencode/` docs |
| Outputs | JSON + Markdown reports in `scripts/output/` (implementation, docs, combined) |
| When to use | Codebase maps, dependency audits, doc inventories; used by `doc-audit` and planning/QA |
| When not to use | Quick ad-hoc queries |
| Dependencies | `tree-sitter` packages (`scripts/package.json`) |
| Allowed agents | Planning Analyst, Tech Lead, QA Reviewer (read-only) |

### `doc-audit`

| | |
|---|---|
| Purpose | Find duplicate/similar content and intra-file prompt duplication in `.opencode/` docs |
| Inputs | All `.md` files in `.opencode/` (excluding `plan/`); `docs-catalog.md`; `codebase-analysis` docs report |
| Outputs | Findings per issue; consolidation only after human approval |
| When to use | Maintaining documentation quality in `.opencode/` |
| When not to use | Without human approval for any change |
| Dependencies | `codebase-analysis` (docs mode) |
| Constraints | 400-line max per file (AGENTS.md); always update `docs-catalog.md` after changes |
| Allowed agents | Orchestrator / docs maintenance sessions |

### `06-branding`

| | |
|---|---|
| Purpose | Define and maintain the visual identity (colors, typography, design tokens) |
| Inputs | Project components/themes (Tailwind config, `index.css`) |
| Outputs | Consistent palette and typography decisions for visual layers |
| When to use | Any visual change (colors, layout, typography, dark mode) |
| When not to use | Backend/logic-only changes |
| Relationship | Consumed by `Senior Frontend` and frontend QA |
| Allowed agents | `Senior Frontend`, QA Reviewer (frontend) |

---

## Skill Relationship Diagram

```text
                   jobfindr-pipeline (orchestrator)
                             │
              ┌──────────────┴───────────────┐
              ▼                              ▼
   jobfindr-pipeline-next         Direct Task Mode
   (resume from pipeline.yaml)    (no PM/TL ceremony)
                             │
                             ▼
                   Implementation + QA
                             │
                             ▼
                    learning-improvement
                             │
                             ▼
                    continuous-learning
                             │
                             ▼
                         session-save
                             │
                             ▼
                        Pipeline end

Support (available to planning, QA and docs work):
  codebase-analysis ──────► doc-audit
  06-branding ────────────► Senior Frontend / QA FE
```
