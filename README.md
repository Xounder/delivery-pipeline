# Delivery Pipeline

> A harness for running an AI agentic pipeline focused on structured development, continuous validation, and self-learning (SRDD as base).

> 🔒 **Human-in-the-Loop**: the pipeline runs autonomously, but critical decisions (requirement clarification and approval of self-learning doc updates) require explicit human approval before any change is applied.

---

## Overview

This repository contains the **harness for the `delivery-pipeline` AI agentic pipeline**, designed as a reusable and codebase-agnostic workflow integrated via **Git Submodule** at `.opencode/` in each consuming project.

The goal is a common structure allowing multiple codebases to use the same AI agent development pipeline while maintaining: a centralized workflow, independent harness evolution, reusability, standardization of agentic processes, separation between workflow and project-specific rules, and the ability to evolve and self-learn.

> 💡 **Want to know more? See: [.docs/01-what-is-this.md](.docs/01-what-is-this.md)** — objectives, SRDD approach, influences, and the conceptual structure.

---

## How the Workflow Works

The pipeline is driven by the **orchestrator** (the main agent), which loads the `delivery-pipeline` skill and routes work through the **Workflow Router** to specialized agents via the `Task` tool (primary agents are invoked using their own workflow).

### Operation Modes

| Mode | Trigger | Flow |
|------|---------|------|
| **New Execution** | `delivery-pipeline:new` command or "start the pipeline" | Workflow Router → Solution Designer / Planning Analyst / Tech Lead → Dev (FE + BE parallel) → QA (FE + BE parallel) → corrections loop → conclusion |
| **Resume** | `delivery-pipeline:resume` command | Loads `pipeline.yaml` → identifies incomplete steps → continues from the last persisted state without re-executing completed work |

### Workflow Routing

The `workflow-router` skill classifies the incoming request and selects the entry point:

| Route | When |
|-------|------|
| **Solution Designer** | No codebase exists; requirements unclear; design exploration needed |
| **Planning Analyst** | Codebase exists; medium/large changes; feasibility/impact analysis needed |
| **Tech Lead** | Requirements defined; scope small; implementation approach obvious |

### Execution Flow

1. **Solution Designer** (when no codebase or design needed) — researches approaches, discusses trade-offs with the user, and only after approval generates design documents.
2. **Planning Analyst** — validates the design against the codebase (or analyzes independently), evaluates impact/risks, and only after approval generates planning documents.
3. **Tech Lead** — transforms approved design/planning into implementation tasks with ownership, dependencies, and execution order.
4. **Development (parallel)** — `Senior Frontend` and `Senior Backend` implement assigned tasks simultaneously when dependencies allow.
5. **QA Review (parallel)** — one `QA Reviewer` per layer (frontend/backend) validates implementation against tasks, acceptance criteria, and git diff.
6. **Corrections Loop** — if QA requests corrections, the implementation agent is re-invoked per layer until approval.
7. **Conclusion** — all tasks approved, `pipeline.yaml` marked `completed`, then the STOP chain runs.

> 💡 **Want to know more? See: [.docs/02-how-it-works.md](.docs/02-how-it-works.md)** — detailed phase walkthrough, state propagation, and failure handling.

---

## Pipeline State

The execution state lives in:

| Artifact | Location | Purpose |
|---|---|---|
| **`pipeline.yaml`** | `.opencode/pipeline.yaml` | Single source of truth for progress: `current_step`, per-step `status`, `notes`, `updated_at`, `errors`, `concerns`, `history`, `handoff`, and `resume` metadata. **Only the orchestrator writes it.** |
| **Planning folder** | `.opencode/plan/<context>/` | Generated artifacts per context: `design-docs/` (Solution Designer), `planning/` + `index.md`/`feasibility.md`/`impact-analysis.md`/`risks.md` (Planning Analyst), and `tasks/` (Tech Lead). |

Template variables are resolved from `.opencode/template/variables.md` (e.g. `[PLAN_FOLDER_LOCATION]`, `[PIPELINE_FILE]`).

---

## Self-Learning Flow

At the end of each completed implementation (triggered by the **STOP hook**), a chain of 3 skills runs in sequence:

```text
learning-improvement → continuous-learning → session-save
```

Learnings are session evaluations (DONE / WRONG / IMPROV / LEARN / NEXT) that update the docs in `.opencode/` — **always after explicit human approval** — so the pipeline needs less and less human intervention each cycle.

> 💡 **Want to know more? See: [.docs/03-self-learning.md](.docs/03-self-learning.md)** — the learning cycle, qualification, validation, storage, and knowledge separation.

---

## Human-in-the-Loop

There are **4 critical points** where human intervention is mandatory:

1. **Requirements Clarification** — the primary agent (Solution Designer / Planning Analyst) asks before assuming.
2. **Improvement Plan Review** (Continuous Learning) — doc changes require human approval.
3. **QA ↔ Developer Correction Loop** — autonomous, no human input required.
4. **Prior Session Context Injection** (Autoloader Plugin) — indirect HITL via session context injection.

> 💡 **Want to know more? See: [.docs/04-human-in-the-loop.md](.docs/04-human-in-the-loop.md)** — detailed diagrams of each intervention point.

---

## Skills Description

All skills live in `.opencode/skills/<name>/SKILL.md` and are loaded through the skill tool.

| Skill | Group | Purpose |
|-------|-------|---------|
| `delivery-pipeline` | Orchestration | Main orchestrator — new execution and resume modes |
| `workflow-router` | Orchestration | Classifies requests and routes to the correct entry point |
| `learning-improvement` | Learning chain | Evaluates a completed session (DONE/WRONG/IMPROV/LEARN/NEXT) |
| `continuous-learning` | Learning chain | Proposes `.opencode/` doc updates based on learnings; requires user approval |
| `session-save` | Learning chain | Persists the session file in `.opencode/sessions/` |
| `codebase-analysis` | Support | Scans the codebase/docs and generates structural reports |
| `doc-audit` | Support | Audits `.opencode/` docs for duplicates and similarities |

> 💡 **Want to know more? See: [.docs/05-skills.md](.docs/05-skills.md)** — full catalog details and relationship diagram.

---

## Agents Description

Agent instruction files live in `.opencode/agents/`.

| Agent | Mode | Responsibility |
|-------|------|----------------|
| **Solution Designer** | primary | Design analysis, trade-offs, web research, design documents |
| **Planning Analyst** | primary | Feasibility, impact, risk analysis, planning documents |
| **Tech Lead** | subagent | Task decomposition, ownership, dependencies, execution order |
| **Senior Frontend** | subagent | Frontend implementation + validation (build/lint/test) |
| **Senior Backend** | subagent | Backend implementation + validation (build/lint/test) |
| **QA Reviewer** | subagent | Reviews one implementation area (frontend/backend) against tasks and acceptance criteria |

QA reviews are dispatched as `subagent_type: "QA Reviewer"` — one instance per review area. The area is passed in the prompt, not as an agent type.

---

## Commands

Custom opencode commands live in `.opencode/commands/`.

| Command | Purpose |
|---------|---------|
| `delivery-pipeline:new` | Starts a new delivery workflow |
| `delivery-pipeline:resume` | Resumes the current delivery workflow from persisted state |
| `doc-audit` | Runs the doc-audit skill |
| `learning-improvement` | Runs the learning-improvement skill (STOP hook) |

---

## Templates

Templates live in `.opencode/template/`.

| File | Purpose |
|------|---------|
| `design-template.md` | Design document template (Solution Designer) |
| `planning-template.md` | Planning document template (Planning Analyst) |
| `task-template.md` | Task artifact template (Tech Lead) |
| `pipeline-template.yaml` | Initial `pipeline.yaml` shape |
| `variables.md` | Shared template variables (`[PLAN_FOLDER_LOCATION]`, `[PIPELINE_FILE]`, etc.) |

---

## Separation of Responsibilities

The harness provides **HOW** to work (agile, generic components); each codebase provides **WHAT** to work with (context, architecture, business rules, conventions).

```text
┌─────────────────────────────────────────────┐
│              delivery-pipeline              │
│              AGNOSTIC HARNESS               │
│  Workflow • Stages • Agents • Contracts     │
└──────────────────────┬──────────────────────┘
                       │ Git Submodule
        ┌──────────────┼──────────────┐
        ▼              ▼              ▼
  Codebase A      Codebase B      Codebase C
  Context         Context         Context
  Skills          Skills          Skills
  Architecture    Architecture    Architecture
```

---

## Git Submodule Usage

The `delivery-pipeline` is shared across codebases through **Git Submodule**, mounted at the `.opencode/` directory of each consuming project. Each project uses its own **project-specific branch** (`dlvr-ppln/<project-name>`), never `main` directly.

> 💡 **Want to know more? See: [.docs/06-submodule-branches.md](.docs/06-submodule-branches.md)** — branch conventions, warning about cloning, PR/promotion flow, and the merge auxiliary branch.

> 💡 **Want to install it? See: [.docs/07-submodule-install.md](.docs/07-submodule-install.md)** — installation (`git submodule add <url> .opencode`), cloning, plugin dependencies, and commit conventions.

---

## Summary

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

The final goal is a single, centralized, reusable, and evolving harness capable of driving AI agentic workflows across multiple codebases without being coupled to the particularities of any individual project.

> ⚠️ **Always verify the branch after cloning or initializing the submodule.** The expected branch must be `dlvr-ppln/<project-name>`.

Thank you for reading this far! ;D