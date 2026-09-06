# Delivery Pipeline

> A harness for running an AI agentic pipeline focused on structured development, continuous validation, and self-learning (SRDD as base).

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

## TODO

This section should document in detail:

* How the `delivery-pipeline` workflow is started;
* Which stages exist;
* Which agents participate in each stage;
* What the inputs and outputs of each stage are;
* How agents communicate;
* How context is propagated;
* How validations are performed;
* How failures are handled;
* When a stage should be repeated;
* How self-learning works;
* How learnings are persisted;
* How previous learnings are used;
* How the workflow is completed.

### TODO: Add a complete workflow diagram

This section should also contain a detailed diagram similar to:

```text
                     ┌───────────────┐
                     │     Input     │
                     └───────┬───────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │Contextualization│
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Planning     │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │    Execution    │
                    └────────┬────────┘
                             │
                             ▼
                    ┌─────────────────┐
                    │   Validation    │
                    └────────┬────────┘
                             │
                    ┌────────┴────────┐
                    │                 │
                    ▼                 ▼
                 Success            Failure
                    │                 │
                    ▼                 │
              Learning ◄─────────────┘
                    │
                    ▼
                  Output
```

> **TODO:** The diagram above is only illustrative. The final diagram must represent the actual stages of the `delivery-pipeline`.

---

# Self-Learning Flow

## TODO

Document:

* What qualifies as a learning;
* Where learnings are extracted from;
* How learnings are validated;
* Where they are stored;
* How incorrect learnings are prevented;
* How agents access previous learnings;
* How knowledge is updated;
* How codebase-specific learnings are separated from global learnings.

It must also be clearly defined:

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

---

# Skills Description

## TODO

This section should document all `skills` used by the `delivery-pipeline`.

For each skill, the following should be described:

* Name;
* Purpose;
* Responsibility;
* Inputs;
* Outputs;
* Preconditions;
* Postconditions;
* Dependencies;
* When it should be used;
* When it should not be used;
* Relationship with other skills;
* Agents allowed to use it.

Example:

```text
Skill
│
├── Responsibility
│
├── Input
│
├── Processing
│
├── Output
│
└── Next Stage
```

### TODO: Add Skill diagrams

The documentation for each skill must include diagrams showing:

* Execution flow;
* Inputs and outputs;
* Relationship with other skills;
* Dependencies;
* Position within the `delivery-pipeline`.

Conceptual example:

```text
                ┌─────────────────┐
                │      Input      │
                └────────┬────────┘
                         │
                         ▼
                  ┌─────────────┐
                  │    Skill    │
                  └──────┬──────┘
                         │
              ┌──────────┴──────────┐
              │                     │
              ▼                     ▼
        Updated Context          Result
              │                     │
              └──────────┬──────────┘
                         │
                         ▼
                    Next Stage
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

# How does the Submodule flow work?

Each codebase maintains a reference to a specific commit of the `delivery-pipeline`.

Example:

```text
Codebase A
    │
    └── delivery-pipeline
            │
            └── Commit: abc123
```

Another codebase may use a different version:

```text
Codebase B
    │
    └── delivery-pipeline
            │
            └── Commit: def456
```

While another uses a newer version:

```text
Codebase C
    │
    └── delivery-pipeline
            │
            └── Commit: ghi789
```

Visually:

```text
                     delivery-pipeline

        ───────────────────────────────────────►

        Commit A
           │
           ▼
        Commit B
           │
           ▼
        Commit C
           │
           ▼
        Commit D


Codebase A ───────────────────────► Commit B

Codebase B ──────────────────────────────► Commit C

Codebase C ─────────────────────────────────────► Commit D
```

This allows each project to control exactly which version of the harness it is using.

---

# Updating the `delivery-pipeline`

When a change is made to the central repository:

```text
delivery-pipeline
```

A new commit will be created:

```text
Commit A
    │
    ▼
Commit B
    │
    ▼
Commit C
    │
    ▼
Commit D ← New Change
```

The codebases are not automatically changed.

Each codebase needs to update its own reference.

Example:

```bash
cd delivery-pipeline
git pull
```

Then, at the root of the codebase:

```bash
git add delivery-pipeline
git commit -m "chore: update delivery-pipeline"
git push
```

The complete flow is:

```text
                ┌─────────────────────┐
                │                     │
                │ delivery-pipeline   │
                │                     │
                └──────────┬──────────┘
                           │
                           │ New Change
                           ▼
                    ┌──────────────┐
                    │  New Commit  │
                    └──────┬───────┘
                           │
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
        ▼                  ▼                  ▼

   Codebase A         Codebase B         Codebase C

        │                  │                  │
        ▼                  ▼                  ▼

 Update ref.        Update ref.        Update ref.

        │                  │                  │
        ▼                  ▼                  ▼

 New commit         New commit         New commit
 in Codebase        in Codebase        in Codebase
```

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

# Roadmap

## TODO

### Workflow

* [ ] Define the actual `delivery-pipeline` stages
* [ ] Define inputs and outputs
* [ ] Define contracts between stages
* [ ] Document the complete flow
* [ ] Create official diagrams

### Agents

* [ ] Define agents
* [ ] Define responsibilities
* [ ] Define handoffs
* [ ] Define contracts

### Skills

* [ ] Map existing skills
* [ ] Create documentation
* [ ] Define inputs and outputs
* [ ] Add diagrams
* [ ] Define dependencies

### Learning

* [ ] Define the self-learning mechanism
* [ ] Define storage
* [ ] Define validation
* [ ] Separate global and local knowledge

### Git Submodule

* [ ] Define the versioning strategy
* [ ] Define the update process
* [ ] Evaluate automated updates
* [ ] Create automations for multiple codebases

---

# Status

> 🚧 Under Development

The structure presented in this repository represents the conceptual foundation of the `delivery-pipeline`.

Several sections still need to be specified as the actual harness components are defined.

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
