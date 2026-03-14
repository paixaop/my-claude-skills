---
name: plan
description: "Generate detailed implementation plans from specs, roadmaps, requirements, or GitHub Issues. Use when the user provides a spec, roadmap, requirements doc, user stories, or says 'create a plan', 'plan this', 'plan from roadmap', 'plan from issues', 'plan from epic #N', 'generate plan', 'write a plan', or 'extend plan'. Also handles GitHub Issues Mode — fetching epics and sub-issues to build plans from existing issue hierarchies. Includes E2E project detection, monorepo support, and execution model selection. Not for vague ideas (use pmp:brainstorm) or reviewing existing plans (use pmp:review)."
---

# PMP: Plan

Generate implementation plans from specs, roadmaps, or GitHub Issues. Each plan includes feature specs with acceptance criteria, E2E test cases, and a dependency graph.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. Read [generate-plans.md](../pmp/references/generate-plans.md) and follow it completely — it contains E2E project detection, plan generation, GitHub Issues Mode, and Extend Mode
3. After saving the plan, ask: "Plan saved. Ready for plan review?"
4. If yes → tell the user to invoke `/pmp:review` or read [review.md](../pmp/references/review.md) and proceed

## Key References

- Plan template: [plan.md](../pmp/assets/plan.md)
- Feature template: [feature.md](../pmp/assets/feature.md)
- GitHub Issues table: [github-issues-table.md](../pmp/assets/github-issues-table.md)
- Testing approaches by project type: [testing-approaches.md](../pmp/references/testing-approaches.md)
- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
