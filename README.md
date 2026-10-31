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

The pipeline is driven by the **orchestrator** (the main agent), which loads the `delivery-pipeline` skill, verifies the architecture documentation set, and routes work through the **Workflow Router** to specialized agents using their named workflows. The orchestrator manages state and handoffs; implementation and validation belong to the specialized agents.

### Operation Modes

| Mode | Trigger | Flow |
|------|---------|------|
| **New Execution** | `delivery-pipeline:new` command or "start the pipeline" | Inspect existing state → initialize missing architecture docs → Workflow Router → design/planning as needed → Tech Lead → bounded session batches with development, QA, corrections, and closeout |
| **Resume** | `delivery-pipeline:resume` command | Load `pipeline.yaml` and durable handoff → reconcile Git, validation/QA evidence, and session identity → resume the active batch or missing closeout checkpoints without replaying completed work |

An incomplete pipeline defaults to resume. Replacing existing state requires explicit user approval and verified, non-overwriting preservation of its state, history, and referenced artifacts inside the repository.

### Workflow Routing

Before routing, the **Architecture Initializer** bootstraps or backfills an incomplete architecture documentation set after user approval: `.opencode/architecture/architecture.md`, `architecture/index.md`, `docs-catalog.md`, `project-structure.md`, and the `INDEX.md` Architecture section. This step is skipped when the set is already complete.

The `workflow-router` skill then classifies the incoming request and selects the entry point:

| Route | When |
|-------|------|
| **Solution Designer** | No codebase exists; requirements unclear; design exploration needed |
| **Planning Analyst** | Codebase exists; medium/large changes; feasibility/impact analysis needed |
| **Tech Lead** | Requirements defined; scope small; implementation approach obvious |

### Execution Flow

1. **Solution Designer** (when no codebase or design needed) — researches approaches, discusses trade-offs with the user, and only after approval generates design documents.
2. **Planning Analyst** — validates the design against an existing codebase (or analyzes independently), evaluates impact/risks, and only after approval generates planning documents. After Solution Designer, projects without a codebase go directly to Tech Lead.
3. **Tech Lead** — transforms approved design/planning or a small, well-defined request into implementation tasks with ownership, dependencies, file-overlap constraints, and bounded session batch proposals. The orchestrator persists all unfinished tasks exactly once across current and future batches before implementation.
4. **Development (parallel)** — `Senior Frontend` and `Senior Backend` implement tasks within the current batch when dependencies, file ownership, and context budget allow. Independent tasks can use multiple instances of the same role, normally one task per agent.
5. **QA Review (parallel)** — one `QA Reviewer` per layer (frontend/backend) validates implementation against tasks, acceptance criteria, and git diff.
6. **Corrections and Task Commits** — QA findings return to the responsible implementation agent, subject to retry and budget gates. Authorized per-task implementation/docs commits are reconciled at QA handoff and their SHAs recorded. Code/config QA approval or a commit does not satisfy deferred runtime acceptance.
7. **Batch Closeout** — persist evidence and carryover, refresh documentation, write a durable handoff, run the STOP chain, then finalize and verify state and authorized remaining commits. Continue a later batch in a fresh OpenCode session.
8. **Conclusion** — mark `pipeline.yaml` as `completed` only after all tasks have full QA and runtime acceptance and final closeout is verified. Blocked or partial work remains unfinished.

> 💡 **Want to know more? See: [.docs/02-how-it-works.md](.docs/02-how-it-works.md)** — detailed phase walkthrough, state propagation, and failure handling.

---

## Pipeline State

Execution generates local state in the consuming project. `pipeline.yaml`, `plan/`, and `sessions/` are ignored by this repository's `.gitignore`; they are not distributed with the harness. The versioned skills and templates define how these artifacts are created and used.

| Artifact | Location | Purpose |
|---|---|---|
| **`pipeline.yaml`** | `.opencode/pipeline.yaml` | Single source of truth for steps, task implementation/QA/validation evidence, commit SHAs, `batches`, `current_batch`, blockers, closeout checkpoints, history, handoff, and exact resume action. Updated after every agent execution. **Only the orchestrator writes it.** |
| **Planning folder** | `.opencode/plan/<context>/` | Generated artifacts per context: `design-docs/` (Solution Designer), `planning/` + `index.md`/`feasibility.md`/`impact-analysis.md`/`risks.md` (Planning Analyst), and `tasks/` (Tech Lead). |
| **Durable handoffs** | `.opencode/plan/<context>/references/` | Locally generated recovery records for batch or planning sessions. Preserve evidence and the next action across sessions, independently of the two-file session retention policy. |

Template variables are resolved from `.opencode/template/variables.md` (e.g. `[PLAN_FOLDER_LOCATION]`, `[PIPELINE_FILE]`).

---

## Bounded Delivery Sessions

Delivery runs in **bounded session batches**, sized by estimated principal-agent context cost rather than a fixed task count or duration. Parallel execution waves happen within a batch; every known unfinished task is assigned exactly once, including future `pending` batches. Oversized tasks are split into independently reviewable scopes before dispatch.

- **Context gate:** the principal session uses `read-context-budget` with model-specific effective limits derived from `.opencode/context-budget.json`. Each implementation/QA dispatch requires measured current intake + estimated additional principal cost + protected closeout reserve to remain strictly below `limits.stopNewWork`. A `continue` result alone is insufficient; unknown or stale measurements stop new work.
- **Early stop:** preserve completed evidence and move unfinished work exactly once to an eligible pending carryover batch. Blocked batches retain their assignments unless explicitly replanned after checking partial output. If planning fills the safe context room, save a planning-only handoff and close the session before implementation.
- **Resumable closeout:** batch states are `pending`, `running`, `closing`, `completed`, or `blocked`. STOP skills and handoff/commit verification have persisted checkpoints; interruptions resume only missing stages. A closed batch is a session-scope checkpoint, not proof of product completion or runtime acceptance.
- **Session boundary:** after verified closeout, start a **new OpenCode session** and run `/delivery-pipeline:resume`. The command alone does not prove a fresh session; if session identity is unavailable, explicit user confirmation is required before advancing. An interrupted `running` batch may resume in the same session after reconciliation and budget checks; a `closing` batch resumes closeout only.

> 💡 **Current execution contract:** [Bounded Delivery Sessions](skills/delivery-pipeline/references/session-batches.md) and [Execution Phases](skills/delivery-pipeline/references/execution-phases.md).

---

## Self-Learning Flow

At each applicable **batch or planning-session closeout**, the **STOP chain** runs once in sequence, using protected context capacity:

```text
learning-improvement → continuous-learning → session-save
```

Learnings are session evaluations (DONE / WRONG / IMPROV / LEARN / NEXT) that update the docs in `.opencode/` — **always after explicit human approval** — so the pipeline needs less and less human intervention each cycle.

Approval waits leave closeout pending; a declined improvement proposal is recorded and the chain proceeds to `session-save`. Only the two most recent session files are retained in `.opencode/sessions/`; detailed recovery evidence stays in the durable handoff. After the chain, the orchestrator finalizes handoff/state and verifies authorized remaining commits before reporting readiness for a new session.

> 💡 **Want to know more? See: [.docs/03-self-learning.md](.docs/03-self-learning.md)** — the learning cycle, qualification, validation, storage, and knowledge separation.

---

## Human-in-the-Loop

Human approval or confirmation is required at these decision points:

1. **Requirements and Approach Approval** — Architecture Initializer, Solution Designer, and Planning Analyst clarify requirements and obtain approval before generating their documents.
2. **Improvement Plan Review** (Continuous Learning) — proposed learning-driven doc changes require approval; a decline is recorded without applying them.
3. **Pipeline Replacement and Commit Authorization** — replacing existing state requires explicit approval and verified preservation; commits require authorization, with missing authorization persisted as a pending checkpoint.
4. **Fresh-Session Confirmation** — when session identity cannot be determined, the user confirms a new principal session before the pipeline advances to another batch.

The QA ↔ developer correction loop is autonomous within retry and budget gates. The Autoloader Plugin supplies prior session context; durable handoffs and persisted evidence drive recovery.

> 💡 **Want to know more? See: [.docs/04-human-in-the-loop.md](.docs/04-human-in-the-loop.md)** — detailed diagrams of each intervention point.

---

## Skills Description

All skills live in `.opencode/skills/<name>/SKILL.md` and are loaded through the skill tool.

| Skill | Group | Purpose |
|-------|-------|---------|
| `delivery-pipeline` | Orchestration | Main orchestrator — architecture prerequisite, bounded batches, context gates, durable handoffs, and resumable closeout |
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
| **Architecture Initializer** | primary | Bootstraps or backfills missing architecture and supporting documentation after approval |
| **Solution Designer** | primary | Design analysis, trade-offs, web research, design documents |
| **Planning Analyst** | primary | Feasibility, impact, risk analysis, planning documents |
| **Tech Lead** | subagent | Task decomposition, ownership, dependencies, file-overlap strategy, and bounded session batch proposals |
| **Senior Frontend** | subagent | Frontend implementation + validation (build/lint/test) |
| **Senior Backend** | subagent | Backend implementation + validation (build/lint/test) |
| **QA Reviewer** | subagent | Reviews one implementation area (frontend/backend) against tasks and acceptance criteria |

QA reviews are dispatched as `subagent_type: "QA Reviewer"` — one instance per review area. The area is passed in the prompt, not as an agent type.

---

## Commands

Custom opencode commands live in `.opencode/commands/`.

| Command | Purpose |
|---------|---------|
| `delivery-pipeline:new` | Inspects existing state, safely initializes a new workflow, and plans bounded batches; defaults to resume for incomplete work |
| `delivery-pipeline:resume` | Reconciles persisted state, durable handoff, Git/QA evidence, and session identity; resumes eligible work or missing closeout checkpoints |
| `doc-audit` | Runs the doc-audit skill |
| `learning-improvement` | Runs the learning-improvement skill (STOP hook) |

---

## Templates

Templates live in `.opencode/template/`.

| File | Purpose |
|------|---------|
| `architecture-template.md` | Architecture base template (Architecture Initializer) |
| `design-template.md` | Design document template (Solution Designer) |
| `planning-template.md` | Planning document template (Planning Analyst) |
| `task-template.md` | Task artifact template (Tech Lead) |
| `pipeline-template.yaml` | Neutral initial state with task, batch, blocker, closeout, and recovery schema guidance |
| `batch-handoff-template.md` | Durable batch handoff with evidence, carryover, closeout checkpoints, and exact resume action |
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
