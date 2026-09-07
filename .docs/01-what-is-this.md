# What is `delivery-pipeline`?

This repository contains the **harness for the `delivery-pipeline` AI agentic pipeline**.

The `delivery-pipeline` was designed as a reusable and codebase-agnostic workflow that can be integrated as a **Git Submodule** into different projects.

## Goals

The goal is to provide a common structure that allows multiple codebases to use the same AI agent development pipeline while maintaining:

- A centralized workflow;
- Independent harness evolution;
- Reusability across different codebases;
- Standardization of agentic processes;
- Separation between the workflow and project-specific rules;
- The ability for the system to evolve and self-learn.

---

## Approach: SRDD

The approach is inspired by **SRDD — Spec-Roundtrip Driven Development**, where development happens through structured cycles of:

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

The goal is to create a structure capable of:

1. Receiving a requirement;
2. Transforming it into structured context;
3. Planning its implementation;
4. Executing tasks through specialized agents;
5. Validating the results;
6. Capturing learnings;
7. Using those learnings in future executions.

---

## Influences

The `delivery-pipeline` has strong influences from AI-assisted development workflows, especially:

- BMAD;
- Superpowers.

These references influenced concepts such as:

- Structured workflows;
- Separation of responsibilities;
- Agent specialization;
- Context-driven execution;
- Planning before implementation;
- Validation after execution;
- Process reuse across projects.

However, the `delivery-pipeline` has its own structure and objectives.

---

## Workflow: `delivery-pipeline`

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

## Conceptual Structure

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

## Principles

| Principle | Description |
|---|---|
| **Agnosticism** | The harness should not depend on a single codebase. |
| **Reusability** | Processes should be reusable across multiple projects. |
| **Context Separation** | Project-specific context belongs to the codebase; generic workflow logic belongs to the harness. |
| **Independent Evolution** | The `delivery-pipeline` should evolve without requiring immediate changes across all codebases. |
| **Controlled Versioning** | Each codebase should control which version of the harness it is using. |
| **Learning** | The system should capture and reuse knowledge from previous executions. |
| **Observability** | Workflow stages should be understandable and traceable. |
