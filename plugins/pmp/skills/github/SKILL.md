---
name: github
description: "Publish implementation plans as GitHub Issues, Epics, Projects, and milestones — or sync plan changes to existing issues. Use when the user says 'create issues', 'publish to GitHub', 'make an epic', 'create GitHub issues for this plan', 'update issues', 'sync issues', 'push plan changes', or wants to turn a plan into trackable GitHub artifacts. Handles 3-level hierarchy (Epic → Feature Issues → Task Sub-issues), complexity tiers (SIMPLE/STANDARD/COMPLEX), Projects v2 boards, label taxonomy, and bidirectional linking. Not for plan creation (use pmp:plan) or execution (use pmp:execute)."
---

# PMP: GitHub

Publish plans as GitHub Issues or sync plan changes to existing issues. Handles the full GitHub integration: epics, sub-issues, Projects v2, milestones, and label taxonomy.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

### Create Issues (from plan)

1. Read [config.md](../pmp/config.md) for current constants — especially Complexity Tiers and Label Taxonomy
2. Read [github-planning.md](../pmp/references/github-planning.md) and follow it completely — it contains pre-flight checks, tier-based creation, Projects v2 setup, plan annotation, and the mandatory checklist
3. After issues are created and the plan is annotated, ask: "Issues created. Ready to start implementation?"
4. If yes → tell the user to invoke `/pmp:execute` or read [execute-loop.md](../pmp/references/execute-loop.md)

### Sync Issues (update existing)

1. Read [config.md](../pmp/config.md) for current constants
2. Read [sync-issues.md](../pmp/references/sync-issues.md) and follow it completely — it diffs plan changes against live issues

## Key References

- Epic template: [issue-epic.md](../pmp/assets/issue-epic.md)
- Feature issue template: [issue-sub-issue.md](../pmp/assets/issue-sub-issue.md)
- Task issue template: [issue-task.md](../pmp/assets/issue-task.md)
- Simple issue template: [issue-simple.md](../pmp/assets/issue-simple.md)
- GitHub Issues table: [github-issues-table.md](../pmp/assets/github-issues-table.md)
- YAML forms: [yaml-feature-form.yml](../pmp/assets/yaml-feature-form.yml), [yaml-bug-form.yml](../pmp/assets/yaml-bug-form.yml), [yaml-epic-form.yml](../pmp/assets/yaml-epic-form.yml)
- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
