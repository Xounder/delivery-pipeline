# How does the `delivery-pipeline` work now (last-version)?

## Operation Modes

The pipeline runs in **two operation modes**:

| Mode | Trigger | Flow |
|------|---------|------|
| **New Execution** | `/delivery-pipeline:new` command or "start the pipeline", "begin development" | Workflow Router → Solution Designer / Planning Analyst → Tech Lead → Dev (FE + BE parallel) → QA (FE + BE parallel) → corrections loop → conclusion |
| **Resume** | `/delivery-pipeline:resume` command | Loads `pipeline.yaml` → identifies incomplete steps → continues from the last persisted state. Completed work is never repeated |

Both modes are orchestrated from the `delivery-pipeline` skill, which is the single entry point for project execution.

---

## Pipeline State — How Context is Propagated

The execution state lives in three artifacts:

| Artifact | Location | Purpose |
|---|---|---|
| **`pipeline.yaml`** | `.opencode/pipeline.yaml` | Single source of truth for progress. Tracks `current_step`, `current_agent`, and per step: `status`, `notes`, `updated_at`, `errors`, `concerns`. Also tracks task status, execution history, and resume metadata. **Only the orchestrator writes it.** |
| **Planning folder** | `.opencode/plan/<context>/` | Generated artifacts per context. Created via the `create-folder-structure` tool with `design-docs/`, `planning/`, and `tasks/` subfolders. Unused subfolders remain empty — this is expected. |

Planning files are written to the context root (`.opencode/plan/<context>/`):

```text
index.md
feasibility.md
impact-analysis.md
risks.md
```

The orchestrator reads `pipeline.yaml.name` at startup and passes the resolved context to all agents via the `Task` tool prompt using template variables from `.opencode/template/variables.md`.

---

## Step 0: Route the Request

Before any execution, the orchestrator invokes the `workflow-router` skill. The router classifies the incoming request and selects the entry point:

| Route | When |
|-------|------|
| **Solution Designer** | No codebase exists; requirements unclear; design exploration needed |
| **Planning Analyst** | Codebase exists; medium/large changes; feasibility/impact analysis needed |
| **Tech Lead** | Requirements already defined; scope small; implementation obvious |

The router never creates files and never modifies source code — it only classifies and routes.

---

## Step 1: Solution Designer (design flow)

| | |
|---|---|
| Input | User request / undefined requirements |
| Agent | `Solution Designer` (primary) |
| Rules | Research approaches (docs + web), present ≥ 2 options with trade-offs, **wait for user approval**. Design documents are only generated after approval. |
| Output | `.opencode/plan/<context>/design-docs/` with `index.md` + one `design-NN-*.md` per design decision |

If a codebase exists, the pipeline automatically chains the **Planning Analyst** after the Solution Designer to validate the design against the actual code. If no codebase exists, proceeds directly to the Tech Lead.

---

## Step 2: Planning Analyst (planning flow)

| | |
|---|---|
| Input | Approved design documents (chained) or the request itself (independent flow) |
| Agent | `Planning Analyst` (primary) |
| Rules | Evaluates feasibility, impact, and risks against the codebase. Presents findings and **waits for user approval**. Planning documents are only generated after approval. |
| Output | `.opencode/plan/<context>/` with `index.md`, `feasibility.md`, `impact-analysis.md`, `risks.md` |

The orchestrator MUST verify planning files were physically created on disk before proceeding.

---

## Step 3: Tech Lead

| | |
|---|---|
| Input | Approved design/planning documents or a direct user request |
| Agent | `Tech Lead` (subagent) |
| Output | `.opencode/plan/<context>/tasks/` with `index.md` + one `TASK-NN-<context-task>.md` per task (dependencies, execution order, ownership, file overlap warnings, parallelization plan) |

---

## Step 4: Development (parallel)

| | |
|---|---|
| Input | Tech Lead tasks |
| Agents | `Senior Frontend` + `Senior Backend` — dispatched **simultaneously** via `Task` tool, one agent per task |
| Output | Implementation of the assigned tasks + layer validation (build, lint, test) |

The orchestrator maximizes parallel execution whenever task dependencies allow. When multiple tasks modify the same file, they are sequenced or use a merge strategy.

---

## Step 5: QA Review (parallel)

| | |
|---|---|
| Input | Implemented code + Tech Lead tasks + git diff |
| Agent | `QA Reviewer` — **one instance per layer** (frontend and backend in parallel) |
| Output | Structured summary with a verdict: **Approved** or **Corrections needed** |

QA reviews are dispatched as `subagent_type: "QA Reviewer"` — the review area (frontend/backend) is passed in the prompt, not as the agent type.

---

## Step 6: Corrections Loop (isolated per layer)

For each layer independently:

1. If QA approved → mark the layer `completed`.
2. If QA requested corrections → reopen the implementation step as `in_progress`.
3. The orchestrator re-invokes the implementation agent via `Task` tool with the QA issues as input — **never fixes code directly**.
4. The agent implements the corrections and QA revalidates.
5. Repeat until approval. After 3 consecutive failures of the same action, the agent must stop and escalate to the orchestrator.

---

## Step 7: Conclusion

1. Confirm both layers are approved and no pending work remains.
2. Update `pipeline.yaml` to `status: completed`, record final errors/concerns, set `updated_at`.
3. Load the mandatory STOP chain in sequence: `learning-improvement` → `continuous-learning` → `session-save`.
4. The STOP chain is part of the pipeline — the session does not end before it completes.

---

## How Agents Communicate

- Primary agents (`Solution Designer`, `Planning Analyst`) are dispatched using their proper agent type name via `Task` with `subagent_type` matching the agent name — never as `general`.
- Subagents (`Tech Lead`, `Senior Frontend`, `Senior Backend`, `QA Reviewer`) are dispatched with their task instructions inline.
- Every agent must return a **non-empty structured summary** using the [standard agent response format](../skills/delivery-pipeline/references/agent-response-format.md).
- `pipeline.yaml` is the single source of truth for state transitions — only the orchestrator updates it, after every agent execution and before dispatching the next agent.

---

## How Validations Are Performed

- QA review per layer, comparing the code against Tech Lead tasks (functional requirements, acceptance criteria) and the relevant git diff.
- Layer-specific checks run via `run-package-command`: build, lint, typecheck, test.
- Architectural checks (e.g., responsibilities properly separated, no obvious design violations).
- For `apps/web` tests, QA checks Playwright E2E pass/fail counts, not just the exit code.

---

## How Failures Are Handled

- QA corrections reopen the implementation step, followed by re-invocation of the implementation agent and QA revalidation (per layer).
- Non-code failures (tests, spawn issues, port conflicts, build problems) are reported in the agent summaries and recorded in the `errors` and `concerns` arrays of `pipeline.yaml`.
- A named agent dispatch failure is retried up to 3 consecutive attempts; after that the step is aborted and reported.
- Execution can always be resumed from `.opencode/pipeline.yaml` via `delivery-pipeline:resume`; completed work is never repeated.

---

## Workflow Diagram (Actual)

```text
                     ┌───────────────────┐
                     │      Input        │
                     │  Feature / Bug    │
                     │  Requirement      │
                     └─────────┬─────────┘
                               │
                 ┌─────────────┴─────────────┐
                 │       Workflow Router     │
                 └──┬────────┬──────────┬────┘
                    ▼        ▼          ▼
            ┌─────────┐ ┌─────────┐ ┌──────────┐
            │Solution │ │Planning │ │  Tech    │
            │Designer │ │Analyst  │ │  Lead    │
            └────┬────┘ └────┬────┘ └────┬─────┘
                 │           │           │
                 ▼           ▼           ▼
          design-docs/   planning/    tasks/
                 │           │           │
                 └───────────┼───────────┘
                             ▼
                 ┌──────────────────────┐
                 │  Development          │
                 │  (FE + BE parallel)   │
                 └─────────┬────────────┘
                           ▼
                 ┌──────────────────────┐
                 │  QA Review            │
                 │  (FE + BE parallel)   │
                 └─────────┬────────────┘
                           │
                 Corrections loop (per layer,
                 until approval)
                           │
                           ▼
                 ┌──────────────────────┐
                 │  Conclusion           │
                 │  (pipeline completed, │
                 │   STOP chain)         │
                 └─────────┬────────────┘
                           │
                           ▼
                        Output
```