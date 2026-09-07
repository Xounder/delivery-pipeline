# Delivery Pipeline

> A harness for running an AI agentic pipeline focused on structured development, continuous validation, and self-learning (SRDD as base).

> 🔒 **Human-in-the-Loop**: the pipeline runs autonomously, but critical decisions (requirement clarification and approval of self-learning doc updates) require explicit human approval before any change is applied.

---

## ⚠️ IMPORTANT NOTE

**This first implementation of the `delivery-pipeline` was built specifically for the `jobfindr-pipeline` workflow.**

The current artifacts (skills, agents, commands, and plugins) were created with the JobFindr workflow exclusively in mind — a concrete project where the monorepo structure (`apps/frontend`, `apps/backend`, `packages/*`), the stack (`Fastify`, `React`, `pnpm --filter`), the endpoints (`GET /jobs/search`), and even the domain (matchmaking, ranking, trust, branding) were embedded directly into the harness.

**At this stage, there was no concern about improvements, extensions, or the possibility of the `delivery-pipeline` becoming a reusable lib independent of codebases.**

This means the harness today is **NOT agnostic** as the rest of this document describes. It serves JobFindr's immediate needs and reflects decisions specific to that project.

**What will be done:** in future commits, we will remove all project-specific references and leave the lib **completely codebase-agnostic**, following the separation of responsibilities documented below (generic harness vs. project context).

Until then, treat this state as an **evolving implementation** and be aware of the coupling limitations before reusing the harness in another project.

---

## Overview

This repository contains the **harness for the `delivery-pipeline` AI agentic pipeline**, designed as a reusable and codebase-agnostic workflow integrated via **Git Submodule** into different projects.

The goal is a common structure allowing multiple codebases to use the same AI agent development pipeline while maintaining: a centralized workflow, independent harness evolution, reusability, standardization of agentic processes, separation between workflow and project-specific rules, and the ability to evolve and self-learn.

> 💡 **Want to know more? See: [.docs/01-what-is-this.md](.docs/01-what-is-this.md)** — objectives, SRDD approach, influences, and the conceptual structure.

---

## How the Workflow Works

The pipeline is driven by the **orchestrator** (the main agent), which loads the `jobfindr-pipeline` skill and routes work to specialized agents through the `Task` tool.

It runs in **two operation modes**:

| Mode | Trigger | Flow |
|------|---------|------|
| **Full Pipeline Mode** | `/start` command or "start the pipeline" | PM → Tech Lead → Dev (FE + BE parallel) → QA (FE + BE parallel) → corrections loop → conclusion |
| **Direct Task Mode** | Direct/specific request ("add an endpoint") | Single layer (or both) → Dev → QA → report. Skips PM and Tech Lead; QA skipped only for trivial tasks |

The pipeline runs through 7 phases: **Phase 0** Context check/resume, **Phase 1** Product Manager (conditional), **Phase 2** Tech Lead, **Phase 3** Development (parallel), **Phase 4** QA Review (parallel), **Phase 5** Corrections loop (isolated per layer), **Phase 6** Conclusion.

State context is propagated through `pipeline.yaml` (single source of truth), `.opencode/plan/active.txt`, and `.opencode/plan/<context>/`.

> 💡 **Want to know more? See: [.docs/02-how-it-works.md](.docs/02-how-it-works.md)** — operation modes, all phases in detail, state propagation, agent communication, validations, and failures.

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

1. **Requirements Clarification** (Phase 1 — PM): the PM asks before assuming.
2. **Improvement Plan Review** (Phase 6 — Continuous Learning): doc changes require human approval.
3. **QA ↔ Developer Correction Loop** (Phase 5): autonomous, no human input required.
4. **Prior Session Context Injection** (Autoloader Plugin): indirect HITL via session context injection.

> 💡 **Want to know more? See: [.docs/04-human-in-the-loop.md](.docs/04-human-in-the-loop.md)** — detailed diagrams of each intervention point.

---

## Skills Description

All skills live in `.opencode/skills/<name>/SKILL.md` and are organized into three groups: **orchestration**, **learning chain**, and **support**.

| Skill | Group | Purpose |
|-------|-------|---------|
| `jobfindr-pipeline` | Orchestration | Main orchestrator — full pipeline or direct task routing |
| `jobfindr-pipeline-next` | Orchestration | Resumes a stopped pipeline from `pipeline.yaml` |
| `learning-improvement` | Learning chain | Evaluates a completed session (DONE/WRONG/IMPROV/LEARN/NEXT) |
| `continuous-learning` | Learning chain | Proposes `.opencode/` doc updates based on learnings |
| `session-save` | Learning chain | Persists the session file in `.opencode/sessions/` |
| `codebase-analysis` | Support | Scans the codebase/docs and generates structural reports |
| `doc-audit` | Support | Audits `.opencode/` docs for duplicates and similarities |
| `06-branding` | Support | Maintains JobFindr branding guidelines |

> 💡 **Want to know more? See: [.docs/05-skills.md](.docs/05-skills.md)** — full catalog details and relationship diagram.

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

The `delivery-pipeline` is shared across codebases through **Git Submodule**. Each project uses its own **project-specific branch** (`dlvr-ppln/<project-name>`), never `main` directly.

> 💡 **Want to know more? See: [.docs/06-submodule-branches.md](.docs/06-submodule-branches.md)** — branch conventions, warning about cloning, PR/promotion flow, and the merge auxiliary branch.

> 💡 **Want to install it? See: [.docs/07-submodule-install.md](.docs/07-submodule-install.md)** — installation, cloning, plugin dependencies, and commit conventions.

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
