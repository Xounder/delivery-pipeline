# Git Submodule — Installation and Setup

## Installing the Submodule in a Codebase

Go to the repository where you want to use the `delivery-pipeline`:

```bash
cd my-codebase
```

Add the submodule (mounted at `.opencode`, the opencode config directory):

```bash
git submodule add <DELIVERY_PIPELINE_REPOSITORY_URL> .opencode
```

Example resulting structure:

```text
my-codebase/
│
├── src/
├── tests/
├── docs/
├── .opencode/
│   ├── skills/
│   ├── agents/
│   ├── plugins/
│   ├── plan/
│   └── ...
├── .gitmodules
└── .git/
```

Register the change:

```bash
git add .
git commit -m "chore: add delivery-pipeline submodule"
git push
```

> After adding or updating the submodule (`.opencode/`), install the opencode plugin dependencies (see below).

---

## Cloning a Codebase that Contains the Submodule

When cloning a codebase that uses the `delivery-pipeline`, use:

```bash
git clone --recurse-submodules <CODEBASE_URL>
```

If the repository has already been cloned:

```bash
git submodule update --init --recursive
```

---

## Installing the opencode Plugin Dependencies

The `.opencode/` directory contains opencode plugins (`.opencode/plugins/*.plugin.ts`) that import the `@opencode-ai/plugin` package.

These dependencies are declared in `.opencode/package.json` and installed into `.opencode/node_modules`. Since `node_modules` is git-ignored, the dependencies must be installed after cloning or updating the submodule:

```bash
cd .opencode
npm install
```

Alternatively, with [Bun](https://bun.sh):

```bash
cd .opencode
bun install
```

> `.opencode/package.json` and `.opencode/package-lock.json` are versioned in the repository — only `node_modules`, `bun.lock`, and project-specific files are git-ignored.

---

## Commit Convention

All commits must follow the **Conventional Commits** specification:

| Type | Description |
|---|---|
| `feat` | A new feature or capability |
| `fix` | A bug fix |
| `chore` | Maintenance or tooling changes |
| `docs` | Documentation changes |
| `refactor` | Code or workflow restructuring without changing behavior |
| `test` | Adding or modifying tests |
| `ci` | CI/CD changes |
| `perf` | Performance improvements |
| `build` | Build system or dependency changes |

### Project-Specific Commits

When a change is related to a specific project, the project name must be included as scope:

```text
<type>(<project-name>): <description>
```

Examples:

```text
feat(market-app): add flow validation
fix(market-app): correct skill loading
chore(market-app): update pipeline configuration
```

For generic changes, omit the project scope:

```text
feat: add flow validation
fix: correct skill loading
```

### Commit Rules

- All commits must follow the Conventional Commits format.
- Project-specific changes must include the project name as the scope.
- Generic harness changes must not use a project-specific scope.
- Keep the commit subject concise and descriptive.
- Use the imperative mood when possible.
- Do not omit the project scope when the change is specific to a consuming project.

### Task-Scoped Commit Preparation

Commit only when the user requests it. Before the first commit, confirm the target branch and inspect status, diff, and recent history in each repository involved. The application's branch and the harness's `dlvr-ppln/<project-name>` branch belong to different repositories; one does not select the other. Follow an explicitly requested branch and the consuming project's branch rules rather than silently committing to its default branch.

Keep documentation and implementation separate, and group implementation by task. Honor any documented subdivision inside a task. Where tasks share files, stage the relevant changes rather than assigning the entire final file to whichever task commits first. Prefer task-boundary commits during authorized delivery over reconstructing a long combined diff afterward.

If retrospective splitting is requested, preserve unrelated work, verify intermediate dependencies/typecheck or build states as appropriate, and confirm the final committed content equals the reviewed working content. Do not rewrite existing history or move branch references without the applicable user authorization. A request to commit does not authorize push, force-push, deployment, or a pull request.

### Close Both Repository Levels

Committing files inside `.opencode` and recording that commit in the parent repository are separate operations:

1. Inspect and commit the authorized harness changes on its project branch.
2. Inspect the parent's submodule diff and verify which harness SHA it will record.
3. Stage and commit the `.opencode` reference in the parent only when that update is within the requested scope. Include `.gitmodules` when introducing or intentionally changing the submodule registration.
4. Check status in both repositories before reporting completion. A clean submodule can still leave its parent with a modified reference; a clean parent index does not prove the submodule working tree is clean.

If the submodule advances during the operation, inspect the new commits. Do not silently include unrelated commits or reset them. Clarify their inclusion when needed, and identify any remaining pointer update or unrelated dirty files in the final report. See [the branch and repository distinction](06-submodule-branches.md#application-and-harness-branches).

---

## Recommended Submodule Setup Flow

```text
Start
  │
  ▼
Create Project Repository
  │
  ▼
Create Project Branch (dlvr-ppln/<project-name>)
  │
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

---

## Relationship Between Codebases

The codebases share the same harness repository but remain independent:

```text
                         ┌─────────────────────┐
                         │                     │
                         │ delivery-pipeline   │
                         │ Shared Harness      │
                         │                     │
                         └──────────┬──────────┘
                                    │
                 ┌──────────────────┼──────────────────┐
                 │                  │                  │
                 ▼                  ▼                  ▼

        ┌────────────────┐ ┌────────────────┐ ┌────────────────┐
        │   Codebase A   │ │   Codebase B   │ │   Codebase C   │
        │ Context A      │ │ Context B      │ │ Context C      │
        │ Skills A       │ │ Skills B       │ │ Skills C       │
        │ Rules A        │ │ Rules B        │ │ Rules C        │
        │ Architecture A │ │ Architecture B │ │ Architecture C │
        └────────────────┘ └────────────────┘ └────────────────┘
```

The `delivery-pipeline` provides **HOW** to work. Each codebase provides **WHAT** to work with.

---

## Separation of Responsibilities

### `delivery-pipeline`

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

### Codebases

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

## Harness Evolution

The `delivery-pipeline` can evolve independently from the codebases:

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

Each codebase is free to decide when to update. In the future, this process may be automated through CI/CD, automated PRs, Dependabot, Renovate, or internal scripts.
