# Codebase Analysis — Documentation

Generated: 2026-09-08T02:23:17.517Z

```

╔══════════════════════════════════════════╗
║     Codebase — Documentation Report      ║
╚══════════════════════════════════════════╝

Generated: 2026-09-08T02:23:17.517Z

── Overview ──
  Total files:       112
  Markdown (.md):    105
  JSON:              5
  YAML:              2
  Text (.txt):       0
  Total headings:    1929
  Total code blocks: 580
  Total list items:  1827

── Code Block Languages ──
  text                 445
  ts                   49
  json                 46
  tsx                  17
  (none)               11
  css                  4
  yaml                 4
  bash                 2
  http                 1
  mermaid              1

── Duplicate Headings (254) ──
  "goals" — 3x
  "self-learning flow" — 2x
  "skills description" — 2x
  "separation of responsibilities" — 2x
  "description" — 8x
  "role" — 6x
  "inputs" — 7x
  "responsibilities" — 12x
  "workflow" — 12x
  "return format" — 2x
  "constraints" — 13x
  "task rules" — 2x
  "dependency rules" — 2x
  "parallelization rules" — 3x
  "output" — 4x
  ... and 239 more

── All Headings (1169) ──
  what is `delivery-pipeline`?                  .opencode/.docs/01-what-is-this.md
  goals                                         .opencode/.docs/01-what-is-this.md
  approach: srdd                                .opencode/.docs/01-what-is-this.md
  influences                                    .opencode/.docs/01-what-is-this.md
  workflow: `delivery-pipeline`                 .opencode/.docs/01-what-is-this.md
  conceptual structure                          .opencode/.docs/01-what-is-this.md
  principles                                    .opencode/.docs/01-what-is-this.md
  how does the `delivery-pipeline` work now (last-version)? .opencode/.docs/02-how-it-works.md
  operation modes                               .opencode/.docs/02-how-it-works.md
  pipeline state — how context is propagated    .opencode/.docs/02-how-it-works.md
  phase 0: context check / resume               .opencode/.docs/02-how-it-works.md
  phase 1: product manager (conditional — full pipeline only) .opencode/.docs/02-how-it-works.md
  phase 2: tech lead (full pipeline only)       .opencode/.docs/02-how-it-works.md
  phase 3: development (parallel)               .opencode/.docs/02-how-it-works.md
  phase 4: qa review (parallel)                 .opencode/.docs/02-how-it-works.md
  phase 5: corrections loop (isolated per layer) .opencode/.docs/02-how-it-works.md
  phase 6: conclusion                           .opencode/.docs/02-how-it-works.md
  how agents communicate                        .opencode/.docs/02-how-it-works.md
  how validations are performed                 .opencode/.docs/02-how-it-works.md
  how failures are handled                      .opencode/.docs/02-how-it-works.md
  workflow diagram (actual)                     .opencode/.docs/02-how-it-works.md
  self-learning flow                            .opencode/.docs/03-self-learning.md
  what qualifies as a learning                  .opencode/.docs/03-self-learning.md
  where learnings are extracted from            .opencode/.docs/03-self-learning.md
  how learnings are validated                   .opencode/.docs/03-self-learning.md
  where they are stored                         .opencode/.docs/03-self-learning.md
  how learnings are used                        .opencode/.docs/03-self-learning.md
  how knowledge is separated (global vs local)  .opencode/.docs/03-self-learning.md
  chain enforcement                             .opencode/.docs/03-self-learning.md
  autolearning cycle                            .opencode/.docs/03-self-learning.md
  ... and 1139 more

── Files Scanned ──
  .opencode/.docs/01-what-is-this.md  [7 headings, 0 code blocks]
  .opencode/.docs/02-how-it-works.md  [14 headings, 0 code blocks]
  .opencode/.docs/03-self-learning.md  [10 headings, 0 code blocks]
  .opencode/.docs/04-human-in-the-loop.md  [5 headings, 0 code blocks]
  .opencode/.docs/05-skills.md  [14 headings, 0 code blocks]
  .opencode/.docs/06-submodule-branches.md  [16 headings, 0 code blocks]
  .opencode/.docs/07-submodule-install.md  [13 headings, 0 code blocks]
  .opencode/.github/pull_request_template.md  [5 headings, 0 code blocks]
  .opencode/INDEX.md  [13 headings, 0 code blocks]
  .opencode/OPENCODE_PLUGINS.md  [21 headings, 0 code blocks]
  .opencode/README.md  [10 headings, 0 code blocks]
  .opencode/agents/00-planning-agent.md  [18 headings, 0 code blocks]
  .opencode/agents/01-solution-designer.md  [18 headings, 4 code blocks]
  .opencode/agents/02-tech-lead.md  [22 headings, 0 code blocks]
  .opencode/agents/03-senior-frontend.md  [20 headings, 0 code blocks]
  .opencode/agents/04-senior-backend.md  [21 headings, 0 code blocks]
  .opencode/agents/05-qa-reviewer.md  [19 headings, 0 code blocks]
  .opencode/architecture/architecture.md  [44 headings, 10 code blocks]
  .opencode/architecture/canonical-data-model.md  [45 headings, 22 code blocks]
  .opencode/architecture/commit-pattern.md  [6 headings, 2 code blocks]
  .opencode/architecture/docs/adrs/ADR-001-google-calendar-source-of-truth.md  [8 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-002-no-database.md  [8 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-003-local-storage.md  [9 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-004-dedicated-calendar.md  [10 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-005-30-minute-slots.md  [8 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-006-stateless-backend.md  [9 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-007-preview-before-save.md  [8 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-008-future-only-recalculation.md  [8 headings, 0 code blocks]
  .opencode/architecture/docs/adrs/ADR-009-google-calendar-wins-conflicts.md  [8 headings, 0 code blocks]
  .opencode/architecture/docs/backend/api-specification.md  [48 headings, 32 code blocks]
  .opencode/architecture/docs/backend/references/api-endpoints.md  [88 headings, 38 code blocks]
  .opencode/architecture/docs/deployment-strategy.md  [16 headings, 7 code blocks]
  .opencode/architecture/docs/design-system.md  [20 headings, 5 code blocks]
  .opencode/architecture/docs/error-handling-strategy.md  [17 headings, 3 code blocks]
  .opencode/architecture/docs/feature/accessibility.md  [12 headings, 1 code blocks]
  .opencode/architecture/docs/feature/allocation-failure-handling.md  [11 headings, 3 code blocks]
  .opencode/architecture/docs/feature/blocked-slot-management.md  [15 headings, 1 code blocks]
  .opencode/architecture/docs/feature/calendar-navigation.md  [11 headings, 2 code blocks]
  .opencode/architecture/docs/feature/calendar-sync-refresh.md  [11 headings, 3 code blocks]
  .opencode/architecture/docs/feature/completion-flow.md  [11 headings, 2 code blocks]
  .opencode/architecture/docs/feature/conflict-handling.md  [10 headings, 1 code blocks]
  .opencode/architecture/docs/feature/dedicated-calendar-setup.md  [12 headings, 1 code blocks]
  .opencode/architecture/docs/feature/design-system-implementation.md  [5 headings, 0 code blocks]
  .opencode/architecture/docs/feature/error-handling-and-retry.md  [5 headings, 0 code blocks]
  .opencode/architecture/docs/feature/future-features-roadmap.md  [15 headings, 0 code blocks]
  .opencode/architecture/docs/feature/google-oauth-authentication.md  [12 headings, 2 code blocks]
  .opencode/architecture/docs/feature/index.md  [5 headings, 0 code blocks]
  .opencode/architecture/docs/feature/manual-calendar-editing.md  [13 headings, 1 code blocks]
  .opencode/architecture/docs/feature/mobile-experience.md  [11 headings, 0 code blocks]
  .opencode/architecture/docs/feature/references/workflow-details.md  [35 headings, 19 code blocks]
  .opencode/architecture/docs/feature/save-to-google-calendar.md  [13 headings, 4 code blocks]
  .opencode/architecture/docs/feature/schedule-generation-workflow.md  [41 headings, 19 code blocks]
  .opencode/architecture/docs/feature/schedule-preview.md  [13 headings, 2 code blocks]
  .opencode/architecture/docs/feature/settings-management.md  [13 headings, 1 code blocks]
  .opencode/architecture/docs/feature/task-management.md  [15 headings, 1 code blocks]
  .opencode/architecture/docs/feature/task-priority-and-weight.md  [41 headings, 26 code blocks]
  .opencode/architecture/docs/feature/unsaved-changes-protection.md  [10 headings, 2 code blocks]
  .opencode/architecture/docs/frontend/component-architecture.md  [56 headings, 19 code blocks]
  .opencode/architecture/docs/frontend/references/component-specs.md  [66 headings, 33 code blocks]
  .opencode/architecture/docs/logging-monitoring-strategy.md  [16 headings, 10 code blocks]
  .opencode/architecture/docs/security-architecture.md  [18 headings, 4 code blocks]
  .opencode/architecture/docs/testing-strategy.md  [14 headings, 1 code blocks]
  .opencode/architecture/domain-model.md  [47 headings, 11 code blocks]
  .opencode/architecture/event-classification.md  [36 headings, 13 code blocks]
  .opencode/architecture/generation-algorithm.md  [39 headings, 12 code blocks]
  .opencode/architecture/google-calendar-integration.md  [31 headings, 7 code blocks]
  .opencode/architecture/index.md  [10 headings, 0 code blocks]
  .opencode/architecture/monorepo-structure.md  [46 headings, 20 code blocks]
  .opencode/architecture/package-contracts.md  [30 headings, 12 code blocks]
  .opencode/architecture/references/algorithm-phases.md  [82 headings, 51 code blocks]
  .opencode/architecture/references/canonical-model-notes.md  [9 headings, 1 code blocks]
  .opencode/architecture/references/contract-enforcement.md  [13 headings, 6 code blocks]
  .opencode/architecture/references/domain-model-references.md  [28 headings, 21 code blocks]
  .opencode/architecture/references/event-details.md  [21 headings, 16 code blocks]
  .opencode/architecture/references/google-operations.md  [70 headings, 38 code blocks]
  .opencode/architecture/references/monorepo-details.md  [25 headings, 15 code blocks]
  .opencode/architecture/references/transformation-pipeline.md  [3 headings, 2 code blocks]
  .opencode/architecture/references/ux-states.md  [65 headings, 40 code blocks]
  .opencode/architecture/ux-flows.md  [55 headings, 15 code blocks]
  .opencode/commands/delivery-pipeline-new.md  [0 headings, 0 code blocks]
  .opencode/commands/delivery-pipeline-resume.md  [0 headings, 0 code blocks]
  .opencode/commands/doc-audit.md  [0 headings, 0 code blocks]
  .opencode/commands/learning-improvement.md  [0 headings, 0 code blocks]
  .opencode/docs-catalog.md  [8 headings, 0 code blocks]
  .opencode/package-lock.json  [keys: name, lockfileVersion, requires, packages]
  .opencode/package.json  [keys: dependencies]
  .opencode/pipeline.yaml  [keys: pipeline]
  .opencode/project-structure.md  [1 headings, 1 code blocks]
  .opencode/skills/codebase-analysis/SKILL.md  [13 headings, 0 code blocks]
  .opencode/skills/codebase-analysis/output/combined-report.json  [keys: generatedAt, mode, implementation, documentation]
  .opencode/skills/codebase-analysis/output/combined-report.md  [2 headings, 1 code blocks]
  .opencode/skills/codebase-analysis/output/docs-report.json  [keys: generatedAt, mode, summary, files, errors, headingIndex, codeBlockLanguages]
  .opencode/skills/codebase-analysis/output/docs-report.md  [1 headings, 1 code blocks]
  .opencode/skills/codebase-analysis/output/implementation-report.json  [keys: generatedAt, mode, parser, summary, files, errors, dependencyCount, perPackage]
  .opencode/skills/codebase-analysis/output/implementation-report.md  [1 headings, 1 code blocks]
  .opencode/skills/continuous-learning/SKILL.md  [4 headings, 0 code blocks]
  .opencode/skills/delivery-pipeline/SKILL.md  [31 headings, 0 code blocks]
  .opencode/skills/delivery-pipeline/references/agent-response-format.md  [2 headings, 2 code blocks]
  .opencode/skills/delivery-pipeline/references/constraints.md  [1 headings, 0 code blocks]
  .opencode/skills/delivery-pipeline/references/failure-recovery.md  [4 headings, 2 code blocks]
  .opencode/skills/delivery-pipeline/references/parallelization-rules.md  [4 headings, 3 code blocks]
  .opencode/skills/delivery-pipeline/references/task-tracking.md  [1 headings, 1 code blocks]
  .opencode/skills/doc-audit/SKILL.md  [11 headings, 0 code blocks]
  .opencode/skills/learning-improvement/SKILL.md  [5 headings, 0 code blocks]
  .opencode/skills/learning-improvement/references/stop-chain.md  [1 headings, 0 code blocks]
  .opencode/skills/session-save/SKILL.md  [7 headings, 2 code blocks]
  .opencode/skills/workflow-router/SKILL.md  [13 headings, 5 code blocks]
  .opencode/template/design-template.md  [29 headings, 0 code blocks]
  .opencode/template/pipeline-template.yaml  [keys: pipeline]
  .opencode/template/planning-template.md  [36 headings, 0 code blocks]
  .opencode/template/task-template.md  [28 headings, 0 code blocks]
  .opencode/template/variables.md  [0 headings, 0 code blocks]

```
