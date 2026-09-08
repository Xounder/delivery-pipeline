## Description

- What does this PR do? Keep it focused on generic, reusable harness improvements.

## Promotion Context

- **Source branch:** `dlvr-ppln/<project-name>-merge_aux`
- **Target branch:** `main`
- **Project-specific change promoted:** Yes / No

## What was removed before this PR

<!-- main must receive ONLY generic, codebase-agnostic content. Confirm what was stripped. -->

- [ ] Project-specific documentation removed
- [ ] Project-specific docs files/examples removed
- [ ] Project-specific artifacts/workflows removed
- [ ] References tying the harness to a single codebase removed

## Validation

- [ ] Change is generic and reusable beyond the originating project
- [ ] Re-test battery: **at least 5 consecutive successful runs (≥ 5x)** — mandatory
- [ ] No degradation observed after the merge simulation
- [ ] Merge conflicts resolved and validated on `-merge_aux` branch

## Checklist

- [ ] Branching follows the `dlvr-ppln/<project-name>` + `-merge_aux` convention
- [ ] Commit messages follow Conventional Commits (`type: description`, optional `(project-name)` scope)
- [ ] No project-specific scope left in generic harness content
- [ ] Docs updated if behavior/workflow/commands changed
- [ ] `.opencode/sessions/` or session files NOT included (self-learning stays out of `main`)
- [ ] 
