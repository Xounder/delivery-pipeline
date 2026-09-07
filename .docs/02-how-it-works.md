# How does the `delivery-pipeline` work now (last-version)?

## Operation Modes

The pipeline runs in **two operation modes**:

| Mode | Trigger | Flow |
|------|---------|------|
| **Full Pipeline Mode** | `/start` command or "start the pipeline", "begin development" | PM → Tech Lead → Dev (FE + BE parallel) → QA (FE + BE parallel) → corrections loop → conclusion |
| **Direct Task Mode** | Direct/specific request ("add an endpoint", "fix bug in X") | Single layer (or both) → Dev → QA → report. Skips PM and Tech Lead entirely; QA is skipped only for trivial tasks (e.g., change a color) |

---

## Pipeline State — How Context is Propagated

The execution state lives in three artifacts:

| Artifact | Location | Purpose |
|---|---|---|
| **`pipeline.yaml`** | Project root | Single source of truth for progress. Tracks `current_step` and, per step: `status`, `notes`, `updated_at`, and `problems`. **Only the orchestrator writes it.** |
| **`.opencode/plan/active.txt`** | `.opencode/plan/` | Points to the active planning context folder. Created by PM, read by all phases, **deleted at pipeline conclusion**. |
| **`.opencode/plan/<context>/`** | `.opencode/plan/` | Planning folder: `epics/` (PM output) and `tasks/` (Tech Lead output). |

The orchestrator reads `active.txt` at startup and passes the active context folder to all subagents via the `Task` tool prompt.

---

## Phase 0: Context Check / Resume

Before any phase, the orchestrator checks for existing context (`glob(".opencode/plan/active.txt")`, analysis files, epics, `pipeline.yaml`). If the pipeline was already started, it resumes from the last incomplete step instead of restarting (see the `jobfindr-pipeline-next` skill).

---

## Phase 1: Product Manager (conditional — Full Pipeline only)

| | |
|---|---|
| Input | Requirement / existing analysis document |
| Agent | `Product Manager` (serial) |
| Rules | Asks questions **only if no plan exists**. If `active.txt` + analysis/epics already exist, the phase is **skipped** (`status: "skipped"`). |
| Output | `.opencode/plan/<context>/epics/` with `index.md` + one `.md` per epic (Objective, Deliverables, Tasks, Acceptance Criteria); writes `active.txt`; updates `pipeline.yaml`. |

---

## Phase 2: Tech Lead (Full Pipeline only)

| | |
|---|---|
| Input | Epics from `.opencode/plan/<context>/epics/` |
| Agent | `Tech Lead` (serial) |
| Output | `.opencode/plan/<context>/tasks/` with `index.md` + one `TASK-NN-<context-task>.md` per task (dependencies, execution order, mermaid dependency graph, and frontend/backend allocation) |

---

## Phase 3: Development (parallel)

| | |
|---|---|
| Input | Tech Lead tasks |
| Agents | `Senior Frontend` + `Senior Backend` — triggered **simultaneously** via `Task` tool |
| Output | Implementation of the assigned tasks + layer validation (lint, build, start/stop the app with HTTP checks) |

---

## Phase 4: QA Review (parallel)

| | |
|---|---|
| Input | Implemented code + Tech Lead tasks (+ epics/recommendations if they exist) |
| Agent | `QA Reviewer` — **one instance per layer** (frontend and backend in parallel) |
| Output | Structured summary with a verdict: **Approved** or **Corrections needed** |

---

## Phase 5: Corrections Loop (isolated per layer)

For each layer independently:

1. If QA approved → mark the layer `completed`.
2. If QA requested corrections → reopen the implementation step as `in_progress`.
3. The orchestrator re-invokes the implementation agent via `Task` tool with the QA issues as input — **never fixes code directly**.
4. The agent implements the corrections and QA revalidates.
5. Repeat until approval. After 3 consecutive failures of the same action, the agent must stop and escalate to the orchestrator.

---

## Phase 6: Conclusion

1. Confirm both layers are approved.
2. Update `pipeline.yaml` to `current_step: "completed"`.
3. Summarize and **commit** the changes following the project conventions.
4. **Remove `.opencode/plan/active.txt`**.
5. Load the learning skills in sequence: `learning-improvement` → `continuous-learning` → `session-save`.
6. Update `AGENTS.md` if necessary (tests, commands, scripts).

---

## How Agents Communicate

- Agents are invoked through the `Task` tool by `subagent_type`:
  `Product Manager` → `Tech Lead` → `Senior Frontend` / `Senior Backend` → `QA Reviewer`.
- Every agent must return a **non-empty structured summary** (what was implemented, validation results, errors) back to the orchestrator.
- `pipeline.yaml` is the single source of truth for state transitions — only the orchestrator updates it.

---

## How Validations Are Performed

- QA review per layer, comparing the code against Tech Lead tasks (functional requirements, acceptance criteria) and, when present, PM epics and planning recommendations.
- Layer-specific checks: lint, build, tests, and app start/stop with HTTP verification.
- Architectural checks (e.g., no business logic in the frontend, provider isolation and statelessness in the backend).

---

## How Failures Are Handled

- QA corrections reopen the implementation step, followed by re-invocation of the implementation agent and QA revalidation (per layer).
- Non-code failures (tests, spawn issues, port conflicts, build problems) are reported in the agent summaries and recorded in the `problems` array of `pipeline.yaml`.
- Apps started during validation are stopped (cleanup) after the checks.

---

## Workflow Diagram (Actual)

```text
                     ┌───────────────────┐
                     │      Input        │
                     │  Feature / Bug    │
                     │  Requirement      │
                     └─────────┬─────────┘
                               │
                ┌──────────────┴──────────────┐
                │        Operation Mode       │
                ▼                             ▼
     Full Pipeline Mode              Direct Task Mode
     (/start, epics, resume)         (specific request)
                │                             │
                ▼                             │
  Phase 0 — Context Check / Resume            │
  (pipeline.yaml, active.txt)                 │
                │                             │
                ▼                             │
  Phase 1 — Product Manager                   │
  (conditional — skips if plan        ┌───────┴──────┐
   already exists)                     ▼              ▼
                │                Senior FE      Senior BE
                ▼                (+ QA FE)      (+ QA BE)
  Phase 2 — Tech Lead                 │              │
  (epics → tasks)                     └──────┬───────┘
                │                            │
                ▼                     Corrections loop
  Phase 3 — Development               (per layer,
  FE + BE parallel                    until approval)
                │                            │
                ▼                            │
  Phase 4 — QA Review                        │
  FE + BE parallel                           │
                │                            │
                └─────────┬──────────────────┘
                          │
                          ▼
             Phase 6 — Conclusion
             (commit, remove active.txt,
              learning skills)
                          │
                          ▼
                       Output
```
