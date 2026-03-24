---
name: plan
description: Use when the user has a spec, roadmap, requirements, or GitHub Issues and says 'create a plan', 'plan from roadmap', 'plan from issues', or 'extend plan' — requires existing input artifacts, not vague ideas
---

# PMP: Plan

**Announce at start:** "Using pmp:plan to generate the implementation plan."

Generate implementation plans from specs, roadmaps, or GitHub Issues. Each plan includes feature specs with acceptance criteria, E2E test cases, and a dependency graph.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **REQUIRED:** Read [generate-plans.md](references/generate-plans.md) and follow it completely — it contains E2E project detection, plan generation, and GitHub Issues Mode. For Extend Mode see [extend-mode.md](references/extend-mode.md)
3. After saving the plan, ask: "Plan saved. Ready for plan review?"
4. If yes → tell the user to invoke `/pmp:plan-review`

## Key References

- Plan template: [plan.md](assets/plan.md)
- Feature template: [feature.md](assets/feature.md)

## Shared Resources

- GitHub Issues table: [github-issues-table.md](../pmp/assets/github-issues-table.md)
- Testing approaches by project type: [testing-approaches.md](../pmp/references/testing-approaches.md)
- **BACKGROUND:** [overview.md](../pmp/references/overview.md) for lifecycle context
