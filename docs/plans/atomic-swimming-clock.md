# Plan: Add "What's Next" Lifecycle Navigation to PMP Output Files

## Context

PMP output/asset files (design docs, plans, reviews, reports) don't tell users or agentic workers what comes next in the lifecycle. Users must consult the overview.md flow diagram to know the next step. Adding a "What's Next" section to each output file with the exact `pmp:` subcommand eliminates this friction and enables agentic handoff.

## Approach

Append a `## What's Next` section to the bottom of 15 output template files. Skip 7 embedded sub-templates that aren't standalone outputs. Bump plugin version `1.8.4` -> `1.8.5`.

## Format

Every file gets this pattern at the end:

```markdown
---

## What's Next

> **Next in the PMP lifecycle:** Description.
>
> | Action | Command | When to use |
> |--------|---------|-------------|
> | Do X | `pmp:subcommand` | When... |
```

Exception: `spec-review-output.md` (493 lines) gets a compact single-line version to stay under 500.

## Files to Modify (15 + 1 version bump)

### Stage: Brainstorm
1. **`plugins/pmp/skills/brainstorm/assets/design-doc.md`** (32 lines)
   - Next: `pmp:plan` (generate plan) or `pmp:discuss` (review findings)

### Stage: Plan
2. **`plugins/pmp/skills/plan/assets/plan.md`** (117 lines)
   - Next: `pmp:review` (validate plan) or `pmp:decompose` (phase large plans)

### Stage: Review
3. **`plugins/pmp/skills/review/assets/review-output.md`** (107 lines)
   - Verdict-conditional: APPROVED -> `pmp:github` or `pmp:execute`; NEEDS REVISION -> `pmp:discuss` or update + `pmp:review`
4. **`plugins/pmp/skills/review/assets/security-analysis-output.md`** (36 lines)
   - Points back to review output for next steps

### Stage: Spec Review (Orchestrator)
5. **`plugins/pmp/skills/spec-review/assets/spec-review-output.md`** (493 lines - COMPACT FORMAT)
   - Next: `pmp:discuss` or fix and re-run `pmp:spec-review`

### Stage: Spec Sub-commands
6. **`plugins/pmp/skills/spec-architecture/assets/spec-architecture-output.md`** (122 lines)
7. **`plugins/pmp/skills/spec-security/assets/spec-security-output.md`** (124 lines)
8. **`plugins/pmp/skills/spec-operations/assets/spec-operations-output.md`** (71 lines)
   - All three: return to `pmp:spec-review` or run sibling sub-commands
9. **`plugins/pmp/skills/spec-implementability/assets/spec-implementability-output.md`** (206 lines)
   - Verdict-conditional: READY -> `pmp:plan`; NOT READY -> fix + re-run; Any -> `pmp:spec-review`

### Stage: GitHub
10. **`plugins/pmp/skills/github/assets/issue-simple.md`** (44 lines)
11. **`plugins/pmp/skills/github/assets/issue-epic.md`** (47 lines)
    - Both: `pmp:execute` to start coding

### Stage: Execute
12. **`plugins/pmp/skills/execute/assets/pr-body.md`** (37 lines)
    - Next: `pmp:changelog` (optional) or merge PR

### Stage: Discuss
13. **`plugins/pmp/skills/discuss/assets/discuss-plan.md`** (56 lines)
    - Next: `pmp:execute` (apply fixes) or `pmp:plan` (regenerate plan)

### Stage: Terminal
14. **`plugins/pmp/skills/changelog/assets/changelog-output.md`** (27 lines)
    - Terminal: merge PR, no further commands
15. **`plugins/pmp/skills/arc42/assets/reorganization-report.md`** (106 lines)
    - Terminal with optional: `pmp:spec-review` to validate reorganized specs

### Version Bump
16. **`plugins/pmp/.claude-plugin/plugin.json`** — `1.8.4` -> `1.8.5`

## Files to Skip (7 embedded sub-templates)

- `plan/assets/feature.md` — embedded inside plan.md
- `pmp/assets/github-issues-table.md` — embedded mapping table
- `pmp/assets/phase-exit-criteria.md` — embedded checklist
- `pmp/assets/task.md` — embedded TDD task structure
- `execute/assets/e2e-test-spec.md` — format spec, not output
- `github/assets/issue-sub-issue.md` — embedded in epic flow
- `github/assets/issue-task.md` — embedded in epic flow

## Risk: 500-Line Limit

`spec-review-output.md` is at 493 lines. Use compact format (5 lines total) to stay at 498.

## Verification

1. All 15 files have a `## What's Next` section at the end
2. All `pmp:` subcommands referenced are valid skill names
3. `spec-review-output.md` stays at or below 500 lines
4. No other file exceeds 500 lines after addition
5. Plugin version bumped to `1.8.5`
