# Self-Learning Flow

The self-learning flow was already implemented and runs at the end of each completed implementation, triggered by the **STOP hook**. It is a chain of 3 skills executed in sequence:

```text
learning-improvement → continuous-learning → session-save
```

---

## What Qualifies as a Learning

A learning is the **session evaluation** extracted after a complete implementation: what was done, what went wrong, what can be improved, discoveries, and suggested next steps. It is structured in one line per topic (English, 5–7 lines):

```text
DONE:  What was accomplished in the session
WRONG: What went wrong (bugs, bad decisions, rework)
IMPROV: Improvements to apply in future sessions
LEARN:  Discoveries and lessons learned
NEXT:   Suggested next steps
```

---

## Where Learnings Are Extracted From

At the end of a completed pipeline (or a complex Direct Task), the `learning-improvement` skill reviews the **session history** (tool calls, errors, results) and produces the evaluation.

---

## How Learnings Are Validated

The `continuous-learning` skill analyzes the evaluation against `.opencode/docs-catalog.md`, builds a **change plan** (which file, why, what to modify) and **always requires explicit human approval** before editing any document. An incorrect or unsupported learning is never written to disk on its own — it only persists after the user accepts the proposed change.

---

## Where They Are Stored

The `session-save` skill persists the raw evaluation via the `save-session` custom tool (`.opencode/tools/save-session.ts`) into:

```text
.opencode/sessions/YYYYMMDD-HH-MM-<description>-session.tmp
```

Only the **2 most recent** session files are retained (newly created + previous one); older files are deleted automatically.

---

## How Learnings Are Used

Approved learnings update the documentation that drives future sessions, via `continuous-learning`:

| File | When |
|---|---|
| `project-structure.md` | Important folders/files changed |
| `docs-catalog.md` | New `.md` files created or removed in `.opencode/` |
| `AGENTS.md` | Commands, scripts, conventions |
| `INDEX.md` | Central guide and references |
| `skills/**/SKILL.md` | Skill adjustments |
| `commands/*.md` | Chat commands |
| `architecture/*.md` | Architecture docs |

---

## How Knowledge Is Separated (Global vs Local)

- **Local / codebase-specific knowledge** lives in each consuming project's `.opencode/` folder and is versioned with that codebase.
- **Global / generic knowledge** belonging to the shared harness is promoted to the `delivery-pipeline` repository through the branch + Pull Request flow (`dlvr-ppln/<project>` → `main`), following the same rules as any other generic improvement.

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

## Chain Enforcement

- The STOP hook expects **all 3 skills** to run — producing only the evaluation without chaining is considered a failure.
- `learning-improvement` hands off directly to `continuous-learning`, which must then call `session-save`.
- `session-save` is the **last** skill in the chain; after saving, the pipeline reports to the user and stops. No further skills are loaded.

---

## Autolearning Cycle

```
    ┌─────────────────────────────────────────────────┐
    │              SESSION N                           │
    │                                                  │
    │  Pipeline runs → errors/successes happen         │
    │          │                                       │
    │          ▼                                       │
    │  learning-improvement:                           │
    │    DONE ✓  WRONG ✗  IMPROV ↑  LEARN 📚  NEXT → │
    │          │                                       │
    │          ▼                                       │
    │  continuous-learning:                            │
    │    Analyzes docs → proposes changes → HUMAN      │
    │    approves                                       │
    │          │                                       │
    │          ▼                                       │
    │  session-save:                                   │
    │    Saves evaluation in .opencode/sessions/       │
    └──────────────────┬──────────────────────────────┘
                       │
                       ▼
    ┌─────────────────────────────────────────────────┐
    │              SESSION N+1                         │
    │                                                  │
    │  Plugin injects context from session N           │
    │          │                                       │
    │          ▼                                       │
    │  Agent reads: "WRONG: wrong barrel import"       │
    │  Agent reads: "LEARN: project uses barrel        │
    │  exports"                                        │
    │          │                                       │
    │          ▼                                       │
    │  Agent does NOT repeat the mistake               │
    │  (the updated doc with the rule is already in    │
    │  .opencode/)                                     │
    │          │                                       │
    │          ▼                                       │
    │  Pipeline improves with every cycle              │
    └─────────────────────────────────────────────────┘
```

---

## Conclusion

This SRDD is a living system that evolves every session. The agents run autonomously, but the **human keeps control** at two critical points:

1. **Approval of document changes** — continuous-learning never edits without a "yes"
2. **Requirements/approach clarification** — the primary agents (Solution Designer / Planning Analyst) ask before assuming

The result is a pipeline that **needs less and less human intervention**, because mistakes become instructions and the instructions evolve the docs in `.opencode/`.
