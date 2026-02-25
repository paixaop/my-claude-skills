# PMP Configuration

Central configuration for all PMP constants. Reference files and templates use these values. Change them here to update the entire skill.

---

## File Paths

| Constant | Value | Used By |
|----------|-------|---------|
| Plans directory | `docs/plans/` | generate-plans, write, brainstorm, execute-loop, SKILL |
| Completed plans directory | `docs/plans/implemented/` | execute-loop, execute |
| Plan filename pattern | `YYYY-MM-DD-<name>-plan.md` | generate-plans, write |
| Design doc filename pattern | `YYYY-MM-DD-<topic>-design.md` | brainstorm |

---

## Plan Frontmatter

Every plan file MUST have YAML frontmatter. The frontmatter tracks lifecycle status and is updated at each workflow stage.

### Schema

```yaml
---
status: draft | reviewed | issues_created | executing | implemented
created_at: "YYYY-MM-DDTHH:MM:SSZ"
reviewed_at: "YYYY-MM-DDTHH:MM:SSZ"
issues_created_at: "YYYY-MM-DDTHH:MM:SSZ"
execution_started_at: "YYYY-MM-DDTHH:MM:SSZ"
completed_at: "YYYY-MM-DDTHH:MM:SSZ"
pr: "#<number>"
epic: "#<number>"
---
```

### Status Transitions

| Stage | Sets `status` to | Also sets |
|-------|-----------------|-----------|
| Generate Plan | `draft` | `created_at` |
| Review approved | `reviewed` | `reviewed_at` |
| GitHub Issues created | `issues_created` | `issues_created_at`, `epic` |
| Execute begins | `executing` | `execution_started_at` |
| PR created / execution done | `implemented` | `completed_at`, `pr` |

### Rules

- Frontmatter is the **first thing** in the plan file (before the `#` title)
- Empty fields use no value (blank after the colon) — never use `null` or `~`
- Timestamps are ISO 8601 UTC
- When updating frontmatter, preserve all existing fields — only update the field(s) for the current transition
- After setting `status: implemented`, the plan is moved from `docs/plans/` to `docs/plans/implemented/`

---

## Thresholds

| Constant | Value | Used By |
|----------|-------|---------|
| Fix loop ceiling | **3** attempts per feature | execute-loop |
| Test coverage minimum | **60%** | SKILL, code-quality-reviewer-prompt |
| Batch execution size | **3** tasks per batch | execute, write |

---

## Context Management

Agents have finite context windows. Every file read, tool output, and subagent return consumes budget. These constraints prevent context exhaustion during plan execution.

| Constant | Value | Used By |
|----------|-------|---------|
| Max features per controller session | **3** | execute-loop, subagent-driven-development |
| Subagent return format | Structured summary (see below) | implementer-prompt, reviewer prompts |
| Test output mode | Concise — use `--tb=short`, `--reporter=dot`, or equivalent | execute-loop |
| Max subagent return length | **~300 words** | implementer-prompt, reviewer prompts |
| Reviewer model | `fast` (for spec-reviewer and code-quality-reviewer) | subagent-driven-development |

### Subagent Return Format

All subagents MUST return structured summaries, not prose. The controller's context is precious — every word of a subagent return lives in the controller's window for the rest of the session.

**Implementer return:**
```
FILES: path/a.ts, path/b.ts, tests/a.test.ts
TESTS: 8 passed, 0 failed
COMMIT: abc1234
ISSUES: none | [1-sentence description per issue]
BLOCKED: no | [1-sentence reason]
```

**Spec reviewer return:**
```
VERDICT: ✅ compliant | ❌ issues found
MISSING: none | [bullet list with file:line]
EXTRA: none | [bullet list with file:line]
MISUNDERSTOOD: none | [bullet list with file:line]
```

**Code quality reviewer return:**
```
VERDICT: ✅ approved | ❌ needs changes
CRITICAL: none | [bullet list with file:line]
IMPORTANT: none | [bullet list with file:line]
MINOR: none | [bullet list with file:line]
```

### Controller Context Hygiene

1. **Batch controllers** — start a fresh controller session every 3 features. After a batch, hand off via a checkpoint summary (features completed, current branch state, remaining features).
2. **Compress subagent returns** — when a subagent returns, extract only the structured fields above. Discard verbose explanations.
3. **Don't echo plan text back** — the controller already has the plan. Subagents should NOT repeat the task description in their return.
4. **Truncate test output** — pipe test runners through concise output modes. Never pass raw multi-page test output into the controller's context.
5. **Avoid redundant reads** — subagents should not re-read files they just wrote unless checking something specific.

---

## Complexity Tiers

| Tier | Task Count | Artifacts |
|------|-----------|-----------|
| SIMPLE | 1–3 tasks, no sub-teams | Single issue with checklist |
| STANDARD | 4–10 tasks, clear feature groups | Epic + native sub-issues + milestone + Projects v2 board |
| COMPLEX | 10+ tasks, multiple teams/milestones | Epic + sub-issues + Projects v2 board + automation rules |

---

## Commit Conventions

| Constant | Value |
|----------|-------|
| Format | Conventional Commits — `<type>(<scope>): <description>` |
| Feature commit (E2E loop) | `feat(<scope>): <feature> with e2e tests` |
| Feature commit (agent-driven) | `feat(<scope>): <feature> with e2e test specs` |

---

## GitHub Conventions

| Constant | Value | Used By |
|----------|-------|---------|
| Epic title prefix | `[EPIC]` | github-planning |
| PR closing keyword | `Closes #N` | execute-loop |
| Issue comment on commit | `Implemented in <commit-sha>` | execute-loop |
| Manual parent link text | `Contributes to #N` | issue-sub-issue template |

### Label Taxonomy

#### Type Labels

| Label | Color | Description |
|-------|-------|-------------|
| `type:epic` | `#8B5CF6` | Epic/parent issue |
| `type:feature` | `#10B981` | New feature |
| `type:bug` | `#EF4444` | Bug report |
| `type:task` | `#3B82F6` | Implementation task |
| `type:chore` | `#6B7280` | Maintenance/housekeeping |
| `type:docs` | `#F59E0B` | Documentation |

#### Priority Labels

| Label | Color | Description |
|-------|-------|-------------|
| `priority:critical` | `#DC2626` | P0 — Drop everything |
| `priority:high` | `#F97316` | P1 — This sprint |
| `priority:medium` | `#EAB308` | P2 — Next sprint |
| `priority:low` | `#84CC16` | P3 — Backlog |

#### Status Labels

| Label | Color | Description |
|-------|-------|-------------|
| `status:blocked` | `#FEE2E2` | Blocked by dependency |
| `status:needs-review` | `#DBEAFE` | Ready for review |
| `status:in-progress` | `#FEF3C7` | Currently being worked |

### Issue Form Destinations

| Form | File |
|------|------|
| Feature Request | `.github/ISSUE_TEMPLATE/feature.yml` |
| Bug Report | `.github/ISSUE_TEMPLATE/bug.yml` |
| Epic | `.github/ISSUE_TEMPLATE/epic.yml` |
| PR Template | `.github/pull_request_template.md` |

### Temp File Naming

| File | Purpose |
|------|---------|
| `/tmp/issue-body.md` | SIMPLE tier issue |
| `/tmp/epic-body.md` | Epic issue |
| `/tmp/sub-issue-N-body.md` | Sub-issues (N = 1, 2, ...) |

---

## Stage Announcements

| Stage | Message |
|-------|---------|
| Brainstorm / Generate Plan | `Planning...` |
| Review | `Reviewing this junior dev's plan...` |
| GitHub Planning | `Publishing to GitHub...` |
| Sync Issues | `Syncing plan to GitHub Issues...` |
| Execute (completion) | `I'm done baby - Skippy the Magnificent` |

---

## Projects v2 Behavior by Tier

| Tier | Fields | Views | Automation |
|------|--------|-------|------------|
| STANDARD | Status | Board | None |
| COMPLEX | Status + Priority + Sprint | Board + Table + Roadmap | Auto-move on assign/close |

---

## Projects v2 Fields (STANDARD + COMPLEX)

### Status Field Options

| Option | Color |
|--------|-------|
| Backlog | GRAY |
| Ready | BLUE |
| In Progress | YELLOW |
| In Review | PURPLE |
| Done | GREEN |

### Priority Field Options

| Option | Color |
|--------|-------|
| P0 Critical | RED |
| P1 High | ORANGE |
| P2 Medium | YELLOW |
| P3 Low | GREEN |

### Default Views

| View | Layout |
|------|--------|
| Board | BOARD_LAYOUT |
| Table | TABLE_LAYOUT |
| Roadmap | ROADMAP_LAYOUT |

### Automation Rules

| Trigger | Action |
|---------|--------|
| Issue assigned | Move to "In Progress" |
| Issue closed | Move to "Done" |
