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
