# Human-in-the-Loop — Points of Human Intervention

The `delivery-pipeline` is an **SRDD (Self-Refining Development Document)** with **autolearning** and **human-in-the-loop**: the agents run autonomously, but every cycle the system learns from its mistakes and successes, proposes improvements to its own instruction documents, and **only applies changes after explicit human approval**.

There are **4 critical points** where human intervention is mandatory:

---

## 1. Requirements / Approach Clarification (Solution Designer / Planning Analyst)

```
┌──────────┐   "I want to add filters"   ┌──────────────────────┐
│  User     │ ──────────────────────────▶ │  Workflow Router      │
└──────────┘                              └───────────┬──────────┘
                                                      │
                                          ┌───────────┴───────────┐
                                          ▼                       ▼
                                ┌────────────────┐      ┌──────────────────┐
                                │ Solution       │      │ Planning Analyst  │
                                │ Designer       │      │ (codebase exists) │
                                └───────┬────────┘      └────────┬─────────┘
                                        │                        │
                              ┌─────────┴────────┐   ┌───────────┴──────────┐
                              ▼                  ▼   ▼                      ▼
                    ┌──────────────────┐  ┌──────────────┐      ┌──────────────────┐
                    │ Presents ≥2      │  │ Presents     │      │ Presents         │
                    │ options with     │  │ feasibility, │      │ analysis and     │
                    │ trade-offs       │  │ impact, risks│      │ asks for         │
                    │ and asks for the │  │ and asks for │      │ clarification    │
                    │ user's choice    │  │ the user's   │      │ if unclear       │
                    └────────┬─────────┘  │ approval     │      └──────────────────┘
                             │           └──────┬───────┘
                             ▼                  │
                    ┌──────────────────┐        │
                    │ User approves →  │◀───────┘
                    │ documents created│
                    └──────────────────┘

 NOTE: Approval must be explicit (semantic wording).
 Design/planning documents are NEVER created before approval.
```

---

## 2. Improvement Plan Review (STOP chain — Continuous Learning)

```
┌─────────────────────────────────────────────────────────┐
│  Session evaluation:                                     │
│  DONE: Implemented country and modality filters          │
│  WRONG: Wrong import of the TrustBadge component         │
│  IMPROV: Always check imports before build               │
│  LEARN: The project uses barrel exports in packages/     │
│         shared                                           │
│  NEXT: Add unit tests for filters                        │
└──────────────────────────┬──────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────┐
│  continuous-learning:                                    │
│  "Based on the session, I propose:"                      │
│                                                          │
│  `Where`:  .opencode/AGENTS.md                           │
│  `Why`: Import error indicates docs do not document      │
│         barrel exports clearly                           │
│  `Best`: AGENTS.md is the place for project conventions  │
│  `Modification`:                                         │
│    + Add "Import Conventions" section with a rule to     │
│      check barrel exports before importing               │
│                                                          │
│  Waiting for human approval...                            │
│                                                          │
│  "Would you like to apply these changes?"                │
└──────────────────────────┬──────────────────────────────┘
                           │
              ┌────────────┼────────────┐
              ▼            ▼            ▼
        ┌──────────┐ ┌──────────┐ ┌──────────────┐
        │  "yes"   │ │"adjust-" │ │    "no"      │
        │          │ │  ments"  │ │              │
        └────┬─────┘ └────┬─────┘ └──────┬───────┘
             │            │              │
             ▼            ▼              ▼
        Applies      Re-plans       Does not
        changes    and re-presents   apply
```

---

## 3. QA ↔ Developer Correction Loop (corrections loop)

```
┌────────────┐   issues   ┌────────────┐
│ QA Review  │ ─────────▶ │  Senior     │
│ (rejected) │            │  Frontend   │
└────────────┘            └──────┬─────┘
      ▲                          │
      │    fixed code             │
      │◀─────────────────────────┘
      │
      ▼
┌────────────┐
│ QA Review  │   APPROVED → Continues
│ (re-check) │
└────────────┘

 NOTE: This loop is AUTONOMOUS — no human input required.
 The orchestrator continues automatically until approval.
```

---

## 4. Prior Session Context Injection (Autoloader Plugin)

```
┌─────────────────────────────────────────────────────────┐
│  New session starts                                      │
│           │                                              │
│           ▼                                              │
│  ┌─────────────────────────────────────┐                │
│  │ session-load-autoloader.plugin.ts    │                │
│  │                                      │                │
│  │ 1. Reads .opencode/sessions/*.tmp    │                │
│  │ 2. Takes the most recent one         │                │
│  │ 3. Injects it into the user's 1st    │                │
│  │    message as [Previous Session      │                │
│  │    Context]                          │                │
│  └─────────────────────────────────────┘                │
│           │                                              │
│           ▼                                              │
│  ┌─────────────────────────────────────┐                │
│  │ Agent receives context:              │                │
│  │ "DONE: ... WRONG: ... LEARN: ..."   │                │
│  │ → Knows what happened in the        │                │
│  │   previous session and applies      │                │
│  │   the learnings                      │                │
│  └─────────────────────────────────────┘                │
│                                                          │
│  INDIRECT HUMAN-IN-THE-LOOP:                             │
│     The human approved the doc changes in the previous   │
│     session; the next session consumes those changes     │
│     automatically                                         │
└─────────────────────────────────────────────────────────┘
```
