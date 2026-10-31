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

The parent repository records the exact submodule commit to use. The branch configured in `.gitmodules` selects which branch remote updates follow; it does **not** automatically check out that branch when cloning or initializing the submodule.

Use the project's dedicated `dlvr-ppln/<project-name>` branch throughout the steps below. See [Project-Specific Branches](06-submodule-branches.md#project-specific-branches) for the naming convention. The branch must already exist on `origin`.

### 1. Check the Branch Configuration in the Parent Repository

The consuming project's `.gitmodules` should include the project-specific branch:

```ini
[submodule ".opencode"]
    path = .opencode
    url = <DELIVERY_PIPELINE_REPOSITORY_URL>
    branch = dlvr-ppln/<project-name>
```

If the branch entry is missing or incorrect, configure it from the **parent repository root**:

```bash
git submodule set-branch --branch dlvr-ppln/<project-name> .opencode
```

Commit the resulting `.gitmodules` change in the parent repository so the whole team shares this configuration.

### 2. Clone or Initialize the Submodule

For a new checkout:

```bash
git clone --recurse-submodules <CODEBASE_URL>
cd <CODEBASE_DIRECTORY>
```

If the repository has already been cloned, run from the **parent repository root**:

```bash
git submodule update --init --recursive
```

Both commands check out the submodule commit recorded by the parent repository, normally in **detached HEAD** state. This is expected: it reproduces the project's recorded version rather than downloading the latest project-branch commit.

### 3. Switch to the Project Branch and Configure Tracking

Before editing the harness or using branch-based updates, enter the submodule and fetch its remote branches:

```bash
cd .opencode
git fetch origin
```

If the project branch already exists locally:

```bash
git checkout dlvr-ppln/<project-name>
git branch --set-upstream-to=origin/dlvr-ppln/<project-name> dlvr-ppln/<project-name>
```

If it exists only on `origin`, create the local tracking branch instead:

```bash
git checkout --track -b dlvr-ppln/<project-name> origin/dlvr-ppln/<project-name>
```

Verify the active branch and its upstream:

```bash
git branch --show-current
git branch -vv
```

The active branch should be `dlvr-ppln/<project-name>` and track `origin/dlvr-ppln/<project-name>`. Switching branches can change the submodule commit relative to the version recorded by the parent repository.

### 4. Pull Updates During Daily Work

To fetch and merge updates from the branch configured in `.gitmodules`, run from the **parent repository root**, with the project branch checked out in the submodule:

```bash
git submodule update --remote --merge .opencode
```

Alternatively, from **inside `.opencode/`**, pull from the upstream configured in step 3:

```bash
git pull
```

The first command uses the branch selected in `.gitmodules`; the second uses the local branch's upstream. Keep both aligned with the same project branch. Review local changes before updating and resolve any merge conflicts before continuing.

When the submodule commit changes, review and commit the updated pointer from the **parent repository root** so other contributors receive the same version:

```bash
git diff --submodule=log -- .opencode
git add .opencode
git commit -m "chore: update delivery-pipeline submodule"
```

Any local submodule changes must first be committed and pushed to its project branch before the parent repository publishes a pointer to that commit.

For a checkout that only needs the version already recorded by the parent repository, use `git submodule update --init --recursive` without `--remote`. Recheck the branch before subsequent harness edits, since this can return the submodule to detached HEAD.

After cloning or updating, follow steps below.

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
