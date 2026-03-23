---
name: github
description: Use when the user says 'create issues', 'publish to GitHub', 'make an epic', 'create GitHub issues for this plan', 'update issues', 'sync issues', or 'push plan changes'
---

# PMP: GitHub

**Announce at start:** "Using pmp:github to publish to GitHub."

Publish plans as GitHub Issues or sync plan changes to existing issues. Handles the full GitHub integration: epics, sub-issues, Projects v2, milestones, and label taxonomy.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

### Create Issues (from plan)

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants — especially Complexity Tiers and Label Taxonomy
2. **REQUIRED:** Read [github-planning.md](references/github-planning.md) and follow it completely — it contains pre-flight checks, tier-based creation, Projects v2 setup, plan annotation, and the mandatory checklist
3. After issues are created and the plan is annotated, ask: "Issues created. Ready to start implementation?"
4. If yes → tell the user to invoke `/pmp:execute`

### Sync Issues (update existing)

1. Read [config.md](../pmp/config.md) for current constants
2. Read [sync-issues.md](references/sync-issues.md) and follow it completely — it diffs plan changes against live issues

## Key References

- Epic template: [issue-epic.md](assets/issue-epic.md)
- Feature issue template: [issue-sub-issue.md](assets/issue-sub-issue.md)
- Task issue template: [issue-task.md](assets/issue-task.md)
- Simple issue template: [issue-simple.md](assets/issue-simple.md)
- YAML forms: [yaml-feature-form.yml](assets/yaml-feature-form.yml), [yaml-bug-form.yml](assets/yaml-bug-form.yml), [yaml-epic-form.yml](assets/yaml-epic-form.yml)

## Shared Resources

- GitHub Issues table: [github-issues-table.md](../pmp/assets/github-issues-table.md)
- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
