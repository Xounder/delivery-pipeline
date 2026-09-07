# Git Submodule — Branches and Promotion

## Using as a Git Submodule

The `delivery-pipeline` should be used by codebases through **Git Submodule**:

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
          │ Submodule  │  │ Submodule  │ │ Submodule  │
          └────────────┘  └────────────┘ └────────────┘
```

---

## Project-Specific Branches

Each project that needs to modify the `delivery-pipeline` should first create its own dedicated branch.

### Branch Naming Convention

All project-specific branches must follow this naming convention:

```text
dlvr-ppln/<project-name>
```

Examples:

```text
dlvr-ppln/market-app
dlvr-ppln/project-alpha
dlvr-ppln/my-backend
dlvr-ppln/codebase-a
```

The prefix `dlvr-ppln/` is mandatory and identifies branches created specifically for project-level work.

---

## Why Use Project-Specific Branches?

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

---

## ⚠️ WARNING — Pay Attention When Cloning the Submodule

```text
╔══════════════════════════════════════════════════════════════════════╗
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
║   DO NOT ACCIDENTALLY MAKE PROJECT-SPECIFIC CHANGES DIRECTLY ON:     ║
║                                                                      ║
║              main                                                   ║
║                                                                      ║
║   OR ON ANOTHER PROJECT'S BRANCH.                                   ║
║                                                                      ║
╚══════════════════════════════════════════════════════════════════════╝
```

Always verify the current branch:

```bash
cd delivery-pipeline
git branch --show-current
```

If the submodule is not using the correct project branch, switch to it:

```bash
git checkout dlvr-ppln/<project-name>
```

---

## Promoting Changes to the Shared Harness

A change created for a specific project may eventually become useful for other codebases.

```text
                    Codebase A
                        │
                        ▼
              dlvr-ppln/project-a
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
```

Only changes that are sufficiently agnostic and reusable should be promoted to `main`.

---

## When to Create a Pull Request

A Pull Request must **not** be created immediately after implementing a new flow. It must first be tested and used within the project-specific branch.

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

### Requirements Before Creating a Pull Request

A PR should only be created when the new flow has been demonstrated to be:

```text
✓ Stable
✓ Useful
✓ Tested in real usage
✓ Validated through actual workflow execution
✓ Mature enough to be reused
✓ Not dependent on project-specific assumptions
```

### Merge Auxiliary Branch

Before opening the PR to `main`, a **merge auxiliary branch** must be created from the current project branch to adjust and validate merge conflicts:

```text
                dlvr-ppln/<project-name>
                        │
                        ▼
        dlvr-ppln/<project-name>-merge_aux
                        │
                        ▼
              Conflict resolution
                        │
                        ▼
              Re-test battery (≥ 5x)
                        │
                        ▼
                 Pull Request
                        │
                        ▼
                      main
```

The auxiliary branch name must be exactly the current branch name with `-merge_aux` suffix:

```text
dlvr-ppln/<project-name>-merge_aux
```

### Re-test Battery Requirement

```text
✓ At least 5 consecutive successful runs (5x) — mandatory minimum
✓ More runs may be required depending on the complexity
✓ Validates that the merged addition suffered NO degradation
```

---

## Final Decision Flow

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

---

## Branch Responsibilities

### `main`

```text
✓ Shared Harness
✓ Generic Workflow
✓ Reusable Skills
✓ Generic Agents
✓ Shared Contracts
✓ Cross-Codebase Improvements
```

### `dlvr-ppln/<project-name>`

```text
✓ Project Experiments
✓ Project Adaptations
✓ Project-Specific Improvements
✓ Temporary Divergences
✓ Features Being Validated
✓ Changes Not Yet Generic Enough for main
```
