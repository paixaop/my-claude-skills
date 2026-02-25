# Write

Create comprehensive implementation plans with bite-sized, TDD-driven tasks.

**Announce at start** with message from [config.md](../config.md) Stage Announcements.

Write plans assuming the implementing agent has zero context and questionable taste. Document everything: which files to touch, what to test, verification commands, expected output. DRY. YAGNI. TDD. Atomic commits.

CRITICAL: No code in plans. All commands and conventions come from project detection (see SKILL.md E2E Project Detection), not hardcoded values.

## Context Gathering

Before writing the plan:

1. **Read all input files COMPLETELY** — design docs, tickets, research
2. **Spawn parallel research tasks:**
   - **codebase-locator**: Find all related files for the feature
   - **codebase-analyzer**: Understand current implementation patterns
   - **docs-locator**: Find existing plans, research, decisions
3. **Read ALL files identified** into main context
4. **Cross-reference** requirements with actual code
5. **Present findings and questions** — use AskQuestion tool for anything not found in research

```
Based on research:

I found:
- [Implementation detail with file:line reference]
- [Pattern or constraint discovered]

Questions:
- [Technical question requiring judgment]
```

DO NOT write plans with unresolved questions. If questions arise, STOP and ask.

## Plan Structure

**Save to:** plans directory using filename pattern from [config.md](../config.md) File Paths.

### Header (required)

Use the header from [assets/plan.md](../assets/plan.md).

### Task Granularity

Each step is one action (2-5 minutes):
- "Write the failing test" — step
- "Run it to make sure it fails" — step
- "Implement the minimal code to make the test pass" — step
- "Run tests and verify they pass" — step
- "Run CI" — step
- "Commit" — step

### Task Template

Use [assets/task.md](../assets/task.md) for each task.

### Phase Exit Criteria

Every phase MUST include [assets/phase-exit-criteria.md](../assets/phase-exit-criteria.md).

**Never proceed with:** compilation errors, failing tests, linting errors, complexity violations.

### Security Section

Every plan MUST include a Security Considerations section (see the bottom of [assets/plan.md](../assets/plan.md)).

## Common Patterns

### New Features
1. Data model/types + unit tests
2. Business logic + unit tests
3. CLI commands or API endpoints + unit tests
4. Integration/E2E tests (final phase)

### Database Changes
1. Schema migration
2. Store methods + unit tests
3. Business logic + unit tests
4. CLI/API endpoints + unit tests
5. Integration tests

### Refactoring
1. Document current behavior with tests
2. Incremental changes maintaining backward compatibility
3. Each change verified by existing + new tests

## Execution Handoff

After saving the plan:

**"Plan complete and saved to `docs/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (this session)** — Fresh subagent per task, two-stage review after each, fast iteration

**2. Batch Execution (this session)** — Execute in batches of 3 tasks, report for review between batches

**Which approach?"**

Then read [execute.md](execute.md) and follow the chosen execution mode.

## Remember

- Exact file paths always
- Exact commands with expected output
- DRY, YAGNI, TDD, atomic commits
- Security in every plan
- No unresolved questions
- Plans describe **what**, not **how** (unless user requests low-level detail)
