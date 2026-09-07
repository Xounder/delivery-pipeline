# Delivery Pipeline

> A harness for running an AI agentic pipeline focused on structured development, continuous validation, and self-learning (SRDD as base).

---

> ## ⚠️ IMPORTANT NOTE
>
> **This first implementation of the `delivery-pipeline` was built specifically for the `jobfindr-pipeline` workflow.**
>
> The current artifacts (skills, agents, commands, and plugins) were created with the JobFindr workflow exclusively in mind — a concrete project where the monorepo structure (`apps/frontend`, `apps/backend`, `packages/*`), the stack (`Fastify`, `React`, `pnpm --filter`), the endpoints (`GET /jobs/search`), and even the domain (matchmaking, ranking, trust, branding) were embedded directly into the harness.
>
> **At this stage, there was no concern about improvements, extensions, or the possibility of the `delivery-pipeline` becoming a reusable lib independent of codebases.**
>
> This means the harness today is **NOT agnostic** as the rest of this document describes. It serves JobFindr's immediate needs and reflects decisions specific to that project.
>
> **What will be done:** in future commits, we will remove all project-specific references and leave the lib **completely codebase-agnostic**, following the separation of responsibilities documented in the sections below (generic harness vs. project context). Each consuming project will be responsible for providing its own context (architecture, conventions, technologies, business rules).
>
> Until then, treat this state as an **evolving implementation** and be aware of the coupling limitations before reusing the harness in another project.

---

## Overview

This repository contains the **harness for the `delivery-pipeline` AI agentic pipeline**.

The `delivery-pipeline` was designed as a reusable and codebase-agnostic workflow that can be integrated as a **Git Submodule** into different projects.

The goal is to provide a common structure that allows multiple codebases to use the same AI agent development pipeline while maintaining:

- A centralized workflow;
- Independent harness evolution;
- Reusability across different codebases;
- Standardization of agentic processes;
- Separation between the workflow and project-specific rules;
- The ability for the system to evolve and self-learn.

---

# What is this repository?

This is the central repository for the harness of an **AI agentic pipeline with self-learning capabilities**.

The approach is inspired by the concept of:

**SRDD — Spec-Roundtrip Driven Development**

Where development happens through structured cycles of:

```text
Specification
      │
      ▼
Planning
      │
      ▼
Implementation
      │
      ▼
Validation
      │
      ▼
Feedback
      │
      ▼
Knowledge Update
      │
      └───────────────┐
                      │
                      ▼
                 New Iteration
````

The goal is not simply to execute AI agents sequentially, but to create a structure capable of:

1. Receiving a requirement;
2. Transforming it into structured context;
3. Planning its implementation;
4. Executing tasks through specialized agents;
5. Validating the results;
6. Capturing learnings;
7. Using those learnings in future executions.

---

## Influences

The `delivery-pipeline` has strong influences from AI-assisted development workflows and frameworks, especially:

* BMAD;
* Superpowers.

These references primarily influenced concepts such as:

* Structured workflows;
* Separation of responsibilities;
* Agent specialization;
* Context-driven execution;
* Planning before implementation;
* Validation after execution;
* Process reuse across projects.

However, the `delivery-pipeline` has its own structure and objectives.

---

# Workflow: `delivery-pipeline`

The main workflow in this repository is called:

```text
delivery-pipeline
```

It represents the central pipeline used to drive tasks through the agentic system.

Conceptually:

```text
                    ┌──────────────────┐
                    │      Input       │
                    │                  │
                    │ Feature / Bug    │
                    │ Requirement      │
                    └────────┬─────────┘
                             │
                             ▼
                  ┌─────────────────────┐
                  │  delivery-pipeline  │
                  │                     │
                  │  Central Workflow   │
                  └──────────┬──────────┘
                             │
                             ▼
                 ┌───────────────────────┐
                 │  Workflow Stages      │
                 │                       │
                 │ • Contextualization   │
                 │ • Planning            │
                 │ • Execution           │
                 │ • Validation          │
                 │ • Learning            │
                 └───────────┬───────────┘
                             │
                             ▼
                    ┌──────────────────┐
                    │      Output      │
                    │                  │
                    │ Code / Docs /    │
                    │ Knowledge        │
                    └──────────────────┘
```

---

# Agnostic Harness

The `delivery-pipeline` contains only **project-agnostic components**.

This means that the harness should not contain rules specific to a particular codebase.

For example, the harness may contain concepts such as:

```text
✓ Plan a change

✓ Analyze context

✓ Delegate implementation

✓ Execute validation

✓ Review results

✓ Record learnings

✓ Re-run stages when necessary
```

But it should not contain specific information such as:

```text
✗ Internal structure of Project A

✗ Project B-specific conventions

✗ Business rules from Project C

✗ Architecture specific to one application

✗ Dependencies exclusive to a single codebase
```

The conceptual separation is:

```text
┌─────────────────────────────────────────────┐
│                                             │
│              delivery-pipeline              │
│                                             │
│               AGNOSTIC HARNESS              │
│                                             │
│  • Workflow                                 │
│  • Stages                                   │
│  • Agents                                   │
│  • Contracts                                │
│  • Processes                                │
│  • Generic Validations                      │
│                                             │
└──────────────────────┬──────────────────────┘
                       │
                       │ Git Submodule
                       │
        ┌──────────────┼──────────────┐
        │              │              │
        ▼              ▼              ▼
┌──────────────┐ ┌──────────────┐ ┌──────────────┐
│  Codebase A  │ │  Codebase B  │ │  Codebase C  │
│              │ │              │ │              │
│ Context      │ │ Context      │ │ Context      │
│ Rules        │ │ Rules        │ │ Rules        │
│ Skills       │ │ Skills       │ │ Skills       │
│ Architecture │ │ Architecture │ │ Architecture │
└──────────────┘ └──────────────┘ └──────────────┘
```

---

# What belongs to `delivery-pipeline`?

The central repository should contain reusable elements shared across different projects.

Example:

```text
delivery-pipeline/
│
├── workflows/
│   └── delivery-pipeline/
│
├── agents/
│
├── skills/
│
├── contracts/
│
├── validation/
│
├── learning/
│
├── templates/
│
└── documentation/
```

The final structure may evolve as the project matures.

---

# What belongs to the Codebases?

Each codebase using the `delivery-pipeline` should maintain its own context.

Example:

```text
codebase/
│
├── src/
│
├── tests/
│
├── docs/
│
├── project-context/
│
├── skills/
│
├── .git/
│
└── delivery-pipeline/
```

The codebase is responsible for providing project-specific information about:

* Architecture;
* Conventions;
* Technologies;
* Dependencies;
* Business rules;
* Directory structure;
* Code patterns;
* Project-specific validation rules;
* Domain context.

---

# How does the `delivery-pipeline` work?

## Workflow overview

The pipeline is driven by the **orchestrator** (the main agent), which loads the `jobfindr-pipeline` skill. The orchestrator does not implement code itself — it routes work to specialized agents through the `Task` tool and updates the pipeline state.

The pipeline runs in **two operation modes**:

| Mode | Trigger | Flow |
|------|---------|------|
| **Full Pipeline Mode** | `/start` command or "start the pipeline", "begin development" | PM → Tech Lead → Dev (FE + BE parallel) → QA (FE + BE parallel) → corrections loop → conclusion |
| **Direct Task Mode** | Direct/specific request ("add an endpoint", "fix bug in X") | Single layer (or both) → Dev → QA → report. Skips PM and Tech Lead entirely; QA is skipped only for trivial tasks (e.g., change a color) |

## Pipeline state — how context is propagated

The execution state lives in three artifacts:

* **`pipeline.yaml`** (project root) — the single source of truth for progress. Tracks `current_step` and, per step: `status` (`pending` / `in_progress` / `completed` / `skipped`), `notes`, `updated_at`, and `problems`. **Only the orchestrator writes it**; agents never touch it.
* **`.opencode/plan/active.txt`** — points to the active planning context folder (`{active: [<folder>], date: <ISO-8601>}`). Created by the PM, read by all phases, **deleted at pipeline conclusion**.
* **`.opencode/plan/<context>/`** — the planning folder: `epics/` (PM output) and `tasks/` (Tech Lead output).

The orchestrator reads `active.txt` at startup and passes the active context folder to all subagents via the `Task` tool prompt, so subagents know where to read their working context.

### Phase 0: Context check / resume

Before any phase, the orchestrator checks for existing context (`glob(".opencode/plan/active.txt")`, analysis files, epics, `pipeline.yaml`). If the pipeline was already started, it resumes from the last incomplete step instead of restarting (see the `jobfindr-pipeline-next` skill).

### Phase 1: Product Manager (conditional)

| | |
|---|---|
| Input | Requirement / existing analysis document |
| Agent | `Product Manager` (serial, Full Pipeline Mode only) |
| Rules | Asks questions **only if no plan exists**. If `active.txt` + analysis/epics already exist, the phase is **skipped** (`status: "skipped"`). |
| Output | `.opencode/plan/<context>/epics/` with `index.md` + one `.md` per epic (Objective, Deliverables, Tasks, Acceptance Criteria); writes `active.txt`; updates `pipeline.yaml`. |

### Phase 2: Tech Lead

| | |
|---|---|
| Input | Epics from `.opencode/plan/<context>/epics/` |
| Agent | `Tech Lead` (serial, Full Pipeline Mode only) |
| Output | `.opencode/plan/<context>/tasks/` with `index.md` + one `TASK-NN-<context-task>.md` per task (dependencies, execution order, mermaid dependency graph, and frontend/backend allocation) |

### Phase 3: Development (parallel)

| | |
|---|---|
| Input | Tech Lead tasks |
| Agents | `Senior Frontend` + `Senior Backend` — triggered **simultaneously** via `Task` tool |
| Output | Implementation of the assigned tasks + layer validation (lint, build, start/stop the app with HTTP checks) |

### Phase 4: QA Review (parallel)

| | |
|---|---|
| Input | The implemented code + Tech Lead tasks (+ epics/recommendations if they exist) |
| Agent | `QA Reviewer` — **one instance per layer** (frontend and backend in parallel) |
| Output | Structured summary with a verdict: **Approved** or **Corrections needed** |

### Phase 5: Corrections loop (isolated per layer)

For each layer independently:

1. If QA approved → mark the layer `completed`.
2. If QA requested corrections → reopen the implementation step as `in_progress`.
3. The orchestrator re-invokes the implementation agent via `Task` tool with the QA issues as input — **never fixes code directly**.
4. The agent implements the corrections and QA revalidates.
5. Repeat until approval. After 3 consecutive failures of the same action, the agent must stop and escalate to the orchestrator.

### Phase 6: Conclusion

1. Confirm both layers are approved.
2. Update `pipeline.yaml` to `current_step: "completed"`.
3. Summarize and **commit** the changes following the project conventions.
4. **Remove `.opencode/plan/active.txt`**.
5. Load the learning skills in sequence: `learning-improvement` → `continuous-learning` → `session-save`.
6. Update `AGENTS.md` if necessary (tests, commands, scripts).

## How agents communicate

* Agents are invoked through the `Task` tool by `subagent_type` (see the agent routing table in `agent-routing`):
  `Product Manager` → `Tech Lead` → `Senior Frontend` / `Senior Backend` → `QA Reviewer`.
* Every agent must return a **non-empty structured summary** (what was implemented, validation results, errors) back to the orchestrator.
* `pipeline.yaml` is the single source of truth for state transitions — only the orchestrator updates it.

## How validations are performed

* QA review per layer, comparing the code against Tech Lead tasks (functional requirements, acceptance criteria) and, when present, PM epics and planning recommendations.
* Layer-specific checks: lint, build, tests, and app start/stop with HTTP verification.
* Architectural checks (e.g., no business logic in the frontend, provider isolation and statelessness in the backend).

## How failures are handled

* QA corrections reopen the implementation step, followed by re-invocation of the implementation agent and QA revalidation (per layer).
* Non-code failures (tests, spawn issues, port conflicts, build problems) are reported in the agent summaries and recorded in the `problems` array of `pipeline.yaml`.
* Apps started during validation are stopped (cleanup) after the checks.

## Workflow diagram (actual)

```text
                     ┌───────────────────┐
                     │      Input        │
                     │  Feature / Bug    │
                     │  Requirement      │
                     └─────────┬─────────┘
                               │
                ┌──────────────┴──────────────┐
                │        Operation Mode       │
                ▼                             ▼
     Full Pipeline Mode              Direct Task Mode
     (/start, epics, resume)         (specific request)
                │                             │
                ▼                             │
  Phase 0 — Context Check / Resume            │
  (pipeline.yaml, active.txt)                 │
                │                             │
                ▼                             │
  Phase 1 — Product Manager                   │
  (conditional — skips if plan        ┌───────┴──────┐
   already exists)                     ▼              ▼
                │                Senior FE      Senior BE
                ▼                (+ QA FE)      (+ QA BE)
  Phase 2 — Tech Lead                 │              │
  (epics → tasks)                     └──────┬───────┘
                │                            │
                ▼                     Corrections loop
  Phase 3 — Development               (per layer,
  FE + BE parallel                    until approval)
                │                            │
                ▼                            │
  Phase 4 — QA Review                        │
  FE + BE parallel                           │
                │                            │
                └─────────┬──────────────────┘
                          │
                          ▼
             Phase 6 — Conclusion
             (commit, remove active.txt,
              learning skills)
                          │
                          ▼
                       Output
```

> The sections below describe the *Self-Learning Flow* and *Skills Description* in detail.

---

# Self-Learning Flow

> The self-learning flow was already implemented and runs at the end of each completed implementation, triggered by the **STOP hook**. It is a chain of 3 skills executed in sequence:

```text
learning-improvement → continuous-learning → session-save
```

## What qualifies as a learning

A learning is the **session evaluation** extracted after a complete implementation: what was done, what went wrong, what can be improved, discoveries, and suggested next steps. It is structured in one line per topic (English, 5–7 lines):

```text
DONE:  What was accomplished in the session
WRONG: What went wrong (bugs, bad decisions, rework)
IMPROV: Improvements to apply in future sessions
LEARN:  Discoveries and lessons learned
NEXT:   Suggested next steps
```

## Where learnings are extracted from

At the end of a completed pipeline (or a complex Direct Task), the `learning-improvement` skill reviews the **session history** (tool calls, errors, results) and produces the evaluation.

## How learnings are validated

The `continuous-learning` skill analyzes the evaluation against `.opencode/docs-catalog.md`, builds a **change plan** (which file, why, what to modify) and **always requires explicit human approval** before editing any document. An incorrect or unsupported learning is never written to disk on its own — it only persists after the user accepts the proposed change.

## Where they are stored

The `session-save` skill persists the raw evaluation via the `save-session` custom tool (`.opencode/tools/save-session.ts`) into:

```text
.opencode/sessions/YYYYMMDD-HH-MM-<description>-session.tmp
```

Only the **2 most recent** session files are retained (newly created + previous one); older files are deleted automatically.

## How learnings are used

Approved learnings update the documentation that drives future sessions, via `continuous-learning`:

* `project-structure.md` — if important folders/files changed
* `docs-catalog.md` — if new `.md` files were created or removed in `.opencode/`
* `AGENTS.md` — commands, scripts, conventions
* `INDEX.md` — central guide and references
* `skills/**/SKILL.md` — skill adjustments
* `commands/*.md` — chat commands
* `architecture/*.md` — architecture docs

## How knowledge is separated (global vs local)

* **Local / codebase-specific knowledge** lives in each consuming project's `.opencode/` folder and is versioned with that codebase.
* **Global / generic knowledge** belonging to the shared harness is promoted to the `delivery-pipeline` repository through the branch + Pull Request flow (`dlvr-ppln/<project>` → `main`), following the same rules as any other generic improvement.

```text
Global Knowledge
        │
        ▼
delivery-pipeline
        │
        │
        ├──────────────┐
        │              │
        ▼              ▼
Codebase A         Codebase B
Local Context      Local Context
```

## Chain enforcement

* The STOP hook expects **all 3 skills** to run — producing only the evaluation without chaining is considered a failure.
* `learning-improvement` hands off directly to `continuous-learning`, which must then call `session-save`.
* `session-save` is the **last** skill in the chain; after saving, the pipeline reports to the user and stops. No further skills are loaded.

---

# Skills Description

All skills used by the `delivery-pipeline` live in `.opencode/skills/<name>/SKILL.md` and are loaded through the skill tool. They are organized into three groups: **orchestration**, **learning chain**, and **support**.

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

## Orchestration skills

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

## Learning chain skills (STOP hook)

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

## Support skills

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

## Skill relationship diagram

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

---

# Using as a Git Submodule

The `delivery-pipeline` should be used by codebases through **Git Submodule**.

The idea is that the harness exists in a single central repository:

```text
delivery-pipeline
```

And is shared across multiple codebases:

```text
                     ┌─────────────────────┐
                     │                     │
                     │  delivery-pipeline  │
                     │                     │
                     │   Base Repository   │
                     │                     │
                     └──────────┬──────────┘
                                │
                 ┌──────────────┼──────────────┐
                 │              │              │
                 ▼              ▼              ▼
          ┌────────────┐  ┌────────────┐ ┌────────────┐
          │ Codebase A │  │ Codebase B │ │ Codebase C │
          │            │  │            │ │            │
          │ Submodule  │  │ Submodule  │ │ Submodule  │
          └────────────┘  └────────────┘ └────────────┘
```

---

# Project-Specific Branches

The `delivery-pipeline` is a shared and reusable harness used across multiple codebases.

Although the central workflow should remain agnostic, individual projects may need to experiment with, adapt, or extend the harness before a change is considered suitable for all codebases.

For this reason, each project that needs to modify the `delivery-pipeline` should first create its own dedicated branch.

## Branch Naming Convention

All project-specific branches must follow this naming convention:

```text
dlvr-ppln/<project-name>
````

Examples:

```text
dlvr-ppln/market-app

dlvr-ppln/project-alpha

dlvr-ppln/my-backend

dlvr-ppln/codebase-a
```

The prefix:

```text
dlvr-ppln/
```

is mandatory and identifies branches created specifically for project-level work on the `delivery-pipeline`.

---

# Why Use Project-Specific Branches?

The shared `delivery-pipeline` may be used by multiple independent codebases:

```text
                        delivery-pipeline

                              main
                               │
                ┌──────────────┼──────────────┐
                │              │              │
                ▼              ▼              ▼

     dlvr-ppln/project-a  dlvr-ppln/project-b  dlvr-ppln/project-c
                │              │              │
                ▼              ▼              ▼

           Codebase A      Codebase B      Codebase C
```

This allows each project to work independently on the harness without immediately affecting other projects.

For example:

```text
Codebase A
    │
    ▼
dlvr-ppln/codebase-a
    │
    │ Project-specific improvements
    ▼
Changes
```

Meanwhile:

```text
Codebase B
    │
    ▼
dlvr-ppln/codebase-b
    │
    │ Different experiments
    ▼
Changes
```

Neither project directly affects the other.

---

# Creating a Project Branch

Before modifying the `delivery-pipeline`, create a branch for the project.

Example:

```bash
cd delivery-pipeline
```

Create the branch:

```bash
git checkout -b dlvr-ppln/<project-name>
```

Example:

```bash
git checkout -b dlvr-ppln/market-app
```

Push the branch to the remote repository:

```bash
git push -u origin dlvr-ppln/market-app
```

The project can now safely work with its own version of the harness.

---

# ⚠️⚠️⚠️ WARNING — PAY ATTENTION WHEN CLONING THE SUBMODULE ⚠️⚠️⚠️

```text
╔══════════════════════════════════════════════════════════════════════╗
║                                                                      ║
║                         ⚠️  IMPORTANT WARNING  ⚠️                     ║
║                                                                      ║
║   DO NOT simply clone or initialize the delivery-pipeline and       ║
║   start making changes immediately.                                 ║
║                                                                      ║
║   BEFORE MAKING ANY CHANGES, ALWAYS CHECK WHICH BRANCH THE           ║
║   SUBMODULE IS CURRENTLY USING.                                    ║
║                                                                      ║
║   EACH PROJECT MUST USE ITS OWN PROJECT-SPECIFIC BRANCH:             ║
║                                                                      ║
║              dlvr-ppln/<project-name>                               ║
║                                                                      ║
║   Example:                                                          ║
║                                                                      ║
║              dlvr-ppln/market-app                                   ║
║                                                                      ║
║   DO NOT ACCIDENTALLY MAKE PROJECT-SPECIFIC CHANGES DIRECTLY ON:     ║
║                                                                      ║
║              main                                                   ║
║                                                                      ║
║   OR ON ANOTHER PROJECT'S BRANCH.                                   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

When cloning or initializing a codebase that contains the submodule, always verify the current branch:

```bash
cd delivery-pipeline
git branch --show-current
```

Expected result:

```text
dlvr-ppln/<project-name>
```

Example:

```text
dlvr-ppln/market-app
```

If the submodule is not using the correct project branch, switch to it:

```bash
git checkout dlvr-ppln/<project-name>
```

Example:

```bash
git checkout dlvr-ppln/market-app
```

---

# Recommended Submodule Setup Flow

The recommended flow for adding the `delivery-pipeline` to a project is:

```text
Start
  │
  ▼
Create Project Repository
  │
  ▼
Create Project Branch
  │
  │
  │ dlvr-ppln/<project-name>
  ▼
Push Branch to delivery-pipeline
  │
  ▼
Add / Clone Submodule
  │
  ▼
⚠️ VERIFY CURRENT BRANCH ⚠️
  │
  ▼
Switch to Project Branch
  │
  ▼
Verify Again
  │
  ▼
Start Using the delivery-pipeline
```

Conceptually:

```text
                       delivery-pipeline

                              main
                               │
                               │
               ┌───────────────┼───────────────┐
               │               │               │
               ▼               ▼               ▼

     dlvr-ppln/project-a dlvr-ppln/project-b dlvr-ppln/project-c

               │               │               │
               │               │               │
               ▼               ▼               ▼

          Codebase A      Codebase B      Codebase C
```

---

# Working on the Project-Specific Branch

Once the correct branch is configured, changes can be made safely.

Example:

```bash
cd delivery-pipeline
```

Verify the branch:

```bash
git branch --show-current
```

Expected:

```text
dlvr-ppln/market-app
```

Make the changes:

```bash
git add .
git commit -m "feat: improve delivery pipeline workflow"
git push
```

These changes will affect:

```text
dlvr-ppln/market-app
```

They will not directly affect:

```text
main

dlvr-ppln/project-a

dlvr-ppln/project-b
```

---

# Promoting Changes to the Shared Harness

A change created for a specific project may eventually become useful for other codebases.

The recommended flow is:

```text
                    Codebase A

                        │
                        ▼

              dlvr-ppln/project-a

                        │
                        │
                 New Improvement
                        │
                        ▼

                   Validation
                        │
                        ▼

               Is it Generic?
                        │
              ┌─────────┴─────────┐
              │                   │
              ▼                   ▼

             NO                   YES
              │                   │
              ▼                   ▼

      Keep in Project       Create Pull Request
          Branch                   │
                                  ▼

                                main

                                  │
                                  ▼

                       Shared Improvement
```

Only changes that are sufficiently agnostic and reusable should be promoted to the shared branch.

Project-specific behavior should remain isolated in the corresponding project branch whenever possible.

---

# Relationship Between `main` and Project Branches

The branches have different responsibilities.

## `main`

The `main` branch represents:

```text
✓ Shared Harness

✓ Generic Workflow

✓ Reusable Skills

✓ Generic Agents

✓ Shared Contracts

✓ Cross-Codebase Improvements
```

It should not contain behavior exclusive to a single project.

---

## `dlvr-ppln/<project-name>`

Project-specific branches represent:

```text
✓ Project Experiments

✓ Project Adaptations

✓ Project-Specific Improvements

✓ Temporary Divergences

✓ Features Being Validated

✓ Changes Not Yet Generic Enough for main
```

---

# Branch Flow

```text
                              main

                                │
                                │
                     Shared Generic Harness
                                │
                ┌───────────────┼───────────────┐
                │               │               │
                ▼               ▼               ▼

       dlvr-ppln/project-a dlvr-ppln/project-b dlvr-ppln/project-c

                │               │               │

                ▼               ▼               ▼

           Codebase A      Codebase B      Codebase C

                │
                │ New Improvement
                ▼

       dlvr-ppln/project-a
                │
                │
                ▼

        Is it reusable?
                │
         ┌──────┴──────┐
         │             │
        No            Yes
         │             │
         ▼             ▼

     Stay in       Pull Request
     Branch             │
                        ▼

                       main
```

---

# When to Create a Pull Request

A Pull Request to promote changes from a project-specific branch to the shared `main` branch must **not** be created immediately after implementing a new flow or improvement.

A new flow must first be tested and used within the project-specific branch.

Example:

```text
dlvr-ppln/<project-name>
````

The recommended process is:

```text
New Flow or Improvement
          │
          ▼
Implementation
          │
          ▼
Testing
          │
          ▼
Real Project Usage
          │
          ▼
Iteration and Improvements
          │
          ▼
Is the Flow Stable?
          │
      ┌───┴───┐
      │       │
     NO      YES
      │       │
      ▼       ▼
 Continue   Is it Useful
 Iterating  Beyond This Project?
              │
          ┌───┴───┐
          │       │
         NO      YES
          │       │
          ▼       ▼
     Keep in      Pull Request
 Project Branch        │
                       ▼
                      main
```

## Requirements Before Creating a Pull Request

A Pull Request should only be created when the new flow has been demonstrated to be:

```text
✓ Stable

✓ Useful

✓ Tested in real usage

✓ Validated through actual workflow execution

✓ Mature enough to be reused

✓ Not dependent on project-specific assumptions
```

The flow must not be promoted to `main` simply because it works once.

It should be used and validated over time within the project-specific branch.

---

## Recommended Promotion Flow

The complete promotion process should follow:

```text
                         Project

                            │
                            ▼

                dlvr-ppln/<project-name>

                            │
                            ▼

                 New Flow / Improvement

                            │
                            ▼

                      Implementation

                            │
                            ▼

                         Testing

                            │
                            ▼

                    Real World Usage

                            │
                            ▼

                   Iteration / Refinement

                            │
                            ▼

                   Proven Stable and Useful?

                            │
                    ┌───────┴────────┐
                    │                │

                   NO               YES

                    │                │

                    ▼                ▼

             Continue Using      Create PR

             and Improving           │

                    │                ▼

                    └──────────► Review

                                      │
                                      ▼

                                     main

                                      │
                                      ▼

                            Shared Improvement

                                      │
                         ┌────────────┼────────────┐
                         │            │            │
                         ▼            ▼            ▼

                     Codebase A    Codebase B    Codebase C
```

---

## Important Rule

> A new flow should only be proposed to the shared `main` branch after it has been proven stable and useful through actual usage in a project-specific branch.

The process should be:

```text
Experiment
    │
    ▼
Use
    │
    ▼
Validate
    │
    ▼
Improve
    │
    ▼
Stabilize
    │
    ▼
Prove Usefulness
    │
    ▼
Create Pull Request
    │
    ▼
main
```

Do not create a Pull Request for:

```text
✗ Untested flows

✗ Experimental ideas

✗ One-time successful executions

✗ Project-specific assumptions

✗ Unstable workflows

✗ Changes whose usefulness has not yet been demonstrated
```

These changes should remain in:

```text
dlvr-ppln/<project-name>
```

until they are sufficiently mature.

---

# Final Decision Flow

```text
New Change
    │
    ▼

Is it Project-Specific?

    │
 ┌──┴──┐
 │     │
YES     NO
 │      │
 ▼      ▼

Project    Can it be
Branch     Tested Safely?

 │           │
 │        ┌──┴──┐
 │        │     │
 ▼       NO    YES
          │     │
          ▼     ▼

       Keep in  Test and Use
       Project      │
       Branch       ▼
              Is it Stable?
                    │
                 ┌──┴──┐
                 │     │
                NO    YES
                 │     │
                 ▼     ▼

              Iterate  Is it Useful
                       for Other Projects?
                              │
                           ┌──┴──┐
                           │     │
                          NO    YES
                           │     │
                           ▼     ▼

                        Keep    Create PR
                        Local       │
                                    ▼

                                   main
```

# Recommended Rules

## Before modifying the Submodule

Always verify:

```bash
git branch --show-current
```

---

## Before creating a Project Branch

Always follow:

```text
dlvr-ppln/<project-name>
```

---

## Never Make Project-Specific Changes Directly on `main`

Project-specific changes should first be developed in:

```text
dlvr-ppln/<project-name>
```

---

## Promote Generic Improvements

If a project-specific improvement becomes useful for multiple codebases:

```text
Project Branch
      │
      ▼
Validation
      │
      ▼
Pull Request
      │
      ▼
main
      │
      ▼
Available to All Codebases
```

---

# Summary

```text
╔═══════════════════════════════════════════════════════════════╗
║                                                               ║
║                    delivery-pipeline                          ║
║                                                               ║
║                           main                                ║
║                            │                                  ║
║                  Shared Generic Harness                       ║
║                            │                                  ║
║             ┌──────────────┼──────────────┐                   ║
║             │              │              │                   ║
║             ▼              ▼              ▼                   ║
║                                                               ║
║  dlvr-ppln/project-a  dlvr-ppln/project-b  dlvr-ppln/project-c║
║             │              │              │                   ║
║             ▼              ▼              ▼                   ║
║                                                               ║
║        Codebase A      Codebase B      Codebase C             ║
║                                                               ║
╚═══════════════════════════════════════════════════════════════╝
```

> ⚠️ **Always verify the branch after cloning or initializing the submodule.**
>
> The expected branch must be:
>
> ```text
> dlvr-ppln/<project-name>
> ```
>
> Never assume that the submodule is already using the correct branch.

# Commit Convention

All commits must follow the **Conventional Commits** specification.

The commit type must describe the nature of the change, such as:

* `feat` — a new feature or capability
* `fix` — a bug fix
* `chore` — maintenance or tooling changes
* `docs` — documentation changes
* `refactor` — code or workflow restructuring without changing behavior
* `test` — adding or modifying tests
* `ci` — CI/CD changes
* `perf` — performance improvements
* `build` — build system or dependency changes

## Project-Specific Commits

When a change is related to a specific project, the project name must be included in the commit scope.

The format is:

```text
<type>(<project-name>): <description>
```

Examples:

```text
feat(market-app): add flow validation
fix(market-app): correct skill loading
chore(market-app): update pipeline configuration
docs(market-app): document project-specific flow
test(market-app): add workflow validation tests
```

For changes that are **not specific to a single project**, the project scope should be omitted:

```text
feat: add flow validation
fix: correct skill loading
chore: update pipeline dependencies
docs: improve delivery-pipeline documentation
ci: add harness validation
```

The project name used in the commit scope should match the project associated with the `dlvr-ppln/<project-name>` branch whenever the change is project-specific.

### Commit Rules

* All commits must follow the Conventional Commits format.
* Project-specific changes must include the project name as the scope.
* Generic harness changes must not use a project-specific scope.
* Keep the commit subject concise and descriptive.
* The project name in the commit scope should match the project associated with the `dlvr-ppln/<project-name>` branch.
* Invalid commit formats should fail CI validation.
* Use a valid Conventional Commit type.
* Use the project name as the scope for project-specific changes.
* Keep the commit subject concise and descriptive.
* Use the imperative mood when possible.
* Do not use arbitrary or undocumented commit types.
* Do not omit the project scope when the change is specific to a consuming project.

### Examples

**Project-specific change:**

```text
feat(market-app): add flow validation
```

**Generic harness change:**

```text
feat: add reusable flow validation
```

**Project-specific bug fix:**

```text
fix(market-app): correct validation step
```

**Generic CI change:**

```text
ci: enforce required validation checks
```

# Installing the Submodule in a Codebase

Go to the repository where you want to use the `delivery-pipeline`.

Example:

```bash
cd my-codebase
```

Add the submodule:

```bash
git submodule add <DELIVERY_PIPELINE_REPOSITORY_URL> delivery-pipeline
```

Example resulting structure:

```text
my-codebase/
│
├── src/
│
├── tests/
│
├── docs/
│
├── delivery-pipeline/
│
│   ├── workflows/
│   ├── agents/
│   ├── skills/
│   ├── contracts/
│   └── ...
│
├── .gitmodules
│
└── .git/
```

Then register the change:

```bash
git add .
git commit -m "chore: add delivery-pipeline submodule"
git push
```

> After adding or updating the submodule, install the opencode plugin dependencies (`.opencode/node_modules`). See [Installing the opencode Plugin Dependencies](#installing-the-opencode-plugin-dependencies).

---

# Cloning a Codebase that Contains the Submodule

When cloning a codebase that uses the `delivery-pipeline`, use:

```bash
git clone --recurse-submodules <CODEBASE_URL>
```

This will clone:

```text
Codebase
   │
   ├── Its own files
   │
   └── delivery-pipeline
          │
          └── Submodule contents
```

---

## If the repository has already been cloned

Run:

```bash
git submodule update --init --recursive
```

This initializes the existing submodules.

---

# Installing the opencode Plugin Dependencies

The `.opencode/` directory contains opencode plugins (`.opencode/plugins/*.plugin.ts`) that import the `@opencode-ai/plugin` package.

These dependencies are declared in `.opencode/package.json` and installed into `.opencode/node_modules`. Since `node_modules` is git-ignored, the dependencies must be installed after cloning or updating the submodule:

```bash
cd .opencode
npm install
```

Alternatively, if you use [Bun](https://bun.sh):

```bash
cd .opencode
bun install
```

> **Note:** `.opencode/package.json` and `.opencode/package-lock.json` are versioned in the repository — only `node_modules`, `bun.lock`, and project-specific files are git-ignored. This ensures the exact pinned version (`@opencode-ai/plugin@1.15.10`) is available to anyone cloning the harness.

---

# Relationship Between Codebases

The codebases share the same harness repository but remain independent.

```text
                         ┌─────────────────────┐
                         │                     │
                         │ delivery-pipeline   │
                         │                     │
                         │ Shared Harness      │
                         │                     │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼

        ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
        │                │ │                │ │                │
        │   Codebase A   │ │   Codebase B   │ │   Codebase C   │
        │                │ │                │ │                │
        │ Context A      │ │ Context B      │ │ Context C      │
        │ Skills A       │ │ Skills B       │ │ Skills C       │
        │ Rules A        │ │ Rules B        │ │ Rules C        │
        │ Architecture A │ │ Architecture B │ │ Architecture C │
        │                │ │                │ │                │
        └────────────────┘ └────────────────┘ └────────────────┘
```

The `delivery-pipeline` provides:

```text
HOW to work
```

While each codebase provides:

```text
WHAT to work with
```

---

# Separation of Responsibilities

## `delivery-pipeline`

Responsible for:

```text
✓ Workflow

✓ Stages

✓ Processes

✓ Agents

✓ Agnostic Skills

✓ Contracts

✓ Generic Validations

✓ Global Self-Learning

✓ Templates

✓ Reusable Structure
```

---

## Codebases

Responsible for:

```text
✓ Project Context

✓ Architecture

✓ Business Rules

✓ Technologies

✓ Dependencies

✓ Project-Specific Skills

✓ Conventions

✓ Project-Specific Validations

✓ Local Knowledge
```

---

# General Flow Between Harness and Codebase

```text
                    CODEBASE

            ┌─────────────────────┐
            │                     │
            │ Project Context     │
            │                     │
            │ Architecture        │
            │ Rules               │
            │ Local Skills        │
            │ Local Knowledge     │
            │                     │
            └──────────┬──────────┘
                       │
                       │ Context
                       ▼

            ┌─────────────────────┐
            │                     │
            │ delivery-pipeline   │
            │                     │
            │ Agnostic Workflow   │
            │                     │
            └──────────┬──────────┘
                       │
                       │ Execution
                       ▼

            ┌─────────────────────┐
            │                     │
            │ Agents / Skills     │
            │                     │
            └──────────┬──────────┘
                       │
                       │ Result
                       ▼

            ┌─────────────────────┐
            │                     │
            │     Codebase        │
            │                     │
            │ Code / Tests        │
            │ Docs / Changes      │
            │                     │
            └─────────────────────┘
```

---

# Harness Evolution Flow

The `delivery-pipeline` can evolve independently from the codebases.

```text
                     ┌─────────────────────┐
                     │ delivery-pipeline   │
                     └──────────┬──────────┘
                                │
                                ▼
                         Development
                                │
                                ▼
                           New Commit
                                │
                                ▼
                           Validation
                                │
                                ▼
                            Release
                                │
                                ▼
              ┌─────────────────┼─────────────────┐
              │                 │                 │
              ▼                 ▼                 ▼

         Codebase A        Codebase B        Codebase C

              │                 │                 │
              ▼                 ▼                 ▼

           Updates           Updates          Keeps
          Submodule         Submodule       Current Version
```

Each codebase is free to decide when to update.

---

# Recommended Update Flow

The recommended flow is:

```text
Change in delivery-pipeline
             │
             ▼
      Commit to Repository
             │
             ▼
      Harness Validation
             │
             ▼
       New Version Available
             │
             ▼
       Optional Update
             │
      ┌──────┼──────┐
      ▼      ▼      ▼
    Repo A Repo B Repo C
```

In the future, this process may be automated through:

* CI/CD;
* Automated Pull Requests;
* Dependabot;
* Renovate;
* Internal scripts;
* Specialized bots.

---

# Conceptual Structure

```text
                        ┌──────────────────────┐
                        │                      │
                        │   delivery-pipeline  │
                        │                      │
                        │   Agnostic Harness   │
                        │                      │
                        └──────────┬───────────┘
                                   │
              ┌────────────────────┼────────────────────┐
              │                    │                    │
              ▼                    ▼                    ▼

       ┌──────────────┐      ┌──────────────┐     ┌──────────────┐
       │   Workflow   │      │    Agents    │     │    Skills    │
       └──────┬───────┘      └──────┬───────┘     └──────┬───────┘
              │                     │                    │
              └─────────────────────┼────────────────────┘
                                    │
                                    ▼
                          ┌──────────────────┐
                          │                  │
                          │ Git Submodule    │
                          │                  │
                          └─────────┬────────┘
                                    │
                    ┌───────────────┼───────────────┐
                    │               │               │
                    ▼               ▼               ▼

                Codebase A      Codebase B      Codebase C
```

---

# Principles

The `delivery-pipeline` should follow the following principles:

### Agnosticism

The harness should not depend on a single codebase.

---

### Reusability

Processes should be reusable across multiple projects.

---

### Context Separation

Project-specific context belongs to the codebase.

Generic workflow logic belongs to the harness.

---

### Independent Evolution

The `delivery-pipeline` should evolve without requiring immediate changes across all codebases.

---

### Controlled Versioning

Each codebase should control which version of the harness it is using.

---

### Learning

The system should be able to capture and reuse knowledge from previous executions.

---

### Observability

Workflow stages should be understandable and traceable.

---

# Summary

```text
                    delivery-pipeline

                           │

                    Agnostic Harness

                           │

                Workflow + Agents + Skills

                           │

                    Git Submodule

                           │

           ┌───────────────┼───────────────┐
           │               │               │

           ▼               ▼               ▼

       Codebase A      Codebase B      Codebase C

           │               │               │

      Context A      Context B      Context C

           │               │               │

           └───────────────┼───────────────┘
                           │

                     Same Harness

                           │

                  Controlled Versions
```

The final goal is to have a single, centralized, reusable, and evolving harness capable of driving AI agentic workflows across multiple codebases without being directly coupled to the particularities of any individual project.

Thank you for reading this far! ;D
