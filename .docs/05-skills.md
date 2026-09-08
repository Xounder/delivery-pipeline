# Skills Description

All skills used by the `delivery-pipeline` live in `.opencode/skills/<name>/SKILL.md` and are loaded through the skill tool. They are organized into three groups: **orchestration**, **learning chain**, and **support**.

---

## Catalog

| Skill | Group | Purpose |
|-------|-------|---------|
| `delivery-pipeline` | Orchestration | Main orchestrator — new execution and resume modes |
| `workflow-router` | Orchestration | Classifies requests and routes to the correct entry point |
| `learning-improvement` | Learning chain | Evaluates a completed session (DONE/WRONG/IMPROV/LEARN/NEXT) |
| `continuous-learning` | Learning chain | Proposes `.opencode/` doc updates based on learnings; requires user approval |
| `session-save` | Learning chain | Persists the session file in `.opencode/sessions/` |
| `codebase-analysis` | Support | Scans the codebase/docs and generates structural reports |
| `doc-audit` | Support | Audits `.opencode/` docs for duplicates and similarities |

---

## Orchestration Skills

### `delivery-pipeline`

| | |
|---|---|
| Purpose | Orchestrate the complete delivery workflow from request through planning, task creation, implementation, review, and completion |
| Inputs | User requirement / `delivery-pipeline:new` or `delivery-pipeline:resume` command |
| Outputs | Updated `pipeline.yaml`; planning artifacts; implemented + QA-approved code; STOP chain execution |
| When to use | Starting a new delivery workflow or resuming a stopped one |
| When not to use | A design/planning discussion handled directly by a primary agent |
| Dependencies | Invokes the `workflow-router` skill and the specialized agents (`Solution Designer`, `Planning Analyst`, `Tech Lead`, `Senior Frontend`, `Senior Backend`, `QA Reviewer`) |
| Relationship | Parent orchestrator; ends by chaining to the learning skills |
| Allowed agents | Orchestrator only |

### `workflow-router`

| | |
|---|---|
| Purpose | Classify the incoming request and select the workflow entry point |
| Inputs | User request; whether a codebase exists; requirement clarity |
| Outputs | Route decision: `solution-designer`, `planning-analyst`, or `tech-lead` |
| When to use | Every pipeline execution, before dispatching the selected agent |
| When not to use | During resume of a pipeline whose route was already decided |
| Dependencies | Invoked by the `delivery-pipeline` skill |
| Relationship | First decision point of the pipeline |
| Allowed agents | Orchestrator only |

---

## Learning Chain Skills (STOP Hook)

The three skills below are an **atomic sequence** executed at the end of every completed pipeline:

```text
learning-improvement → continuous-learning → session-save
```

### `learning-improvement`

| | |
|---|---|
| Purpose | Review the finished session and extract the evaluation |
| Inputs | Session history (tool calls, errors, results) |
| Outputs | Text evaluation: DONE, WRONG, IMPROV, LEARN, NEXT (up to 12 lines) |
| When to use | End of a complete pipeline (STOP chain) |
| When not to use | In isolation — it must always chain to `continuous-learning` |
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
| When to use | Last step of the STOP chain |
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
| Allowed agents | Solution Designer, Planning Analyst, Tech Lead, QA Reviewer (read-only) |

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

---

## Skill Relationship Diagram

```text
                 delivery-pipeline (orchestrator)
                             │
                             ▼
                    workflow-router
                    (routes the request)
                             │
              ┌──────────────┴───────────────┐
              │              │               │
              ▼              ▼               ▼
   Solution Designer  Planning Analyst  Tech Lead
   (design-docs/)     (planning/)       (tasks/)
              │              │               │
              └──────────────┼───────────────┘
                             │
                             ▼
                 Implementation (FE + BE parallel)
                             │
                             ▼
                    QA Review (FE + BE parallel)
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
  codebase-analysis ──────► Solution Designer / Planning Analyst / QA
```

---

## Relationship to Agents

The orchestration skills drive the agent pipeline. Primary agents (`Solution Designer`, `Planning Analyst`) run their full workflow with user-approval gates; subagents (`Tech Lead`, `Senior Frontend`, `Senior Backend`, `QA Reviewer`) execute dispatched tasks. See [`../agents/`](../agents/) for agent definitions.