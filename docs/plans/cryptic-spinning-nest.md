# PMP: Decouple E2E Tests from Feature Implementation

## Context

The current PMP execute loop bundles implementation + E2E tests as a single atomic unit per feature: IMPLEMENT → WRITE E2E → RUN ALL E2E → FIX LOOP → COMMIT. This doubles the cognitive load and context consumption per task. The implementing agent must context-switch between production code and test code, and running ALL E2E tests after each feature gets progressively more expensive.

The fix: split each feature into two separate tasks — an implementation task (with unit/integration tests) and an E2E test task. Each gets its own commit. This halves the per-task workload while preserving full coverage.

## Changes

### 1. `references/execute-loop.md` — Split per-feature loop

**Current flow (per feature):**
```
IMPLEMENT → WRITE E2E → RUN ALL E2E → FIX LOOP → COMMIT
```

**New flow (per feature = 2 tasks):**
```
Task A: IMPLEMENT (+ unit/integration tests) → RUN UNIT TESTS → FIX LOOP → COMMIT
Task B: WRITE E2E TESTS → RUN ALL E2E → FIX LOOP → COMMIT
```

Specific edits:

- **Rename "Per-Feature Loop" sections** to reflect the two-task model
- **Step 1 (IMPLEMENT)**: Stays the same — implement the feature, apply unit TDD if applicable
- **Step 2 (WRITE E2E TEST)**: Becomes a separate task (Task B) that runs after Step 1's commit
- **Step 3 (RUN ALL E2E)**: Moves into Task B
- **Step 4 (FIX LOOP)**: Applies to both tasks — unit test fix loop in Task A, E2E fix loop in Task B. Same ceiling (3 attempts) and classification system for both.
- **Step 5 (COMMIT)**: Two commits per feature now:
  - Task A commit: `feat(<scope>): <feature>` (implementation + unit tests)
  - Task B commit: `test(<scope>): e2e tests for <feature>` (E2E tests only)
- **Step 5.5 (TWO-STAGE REVIEW)**: Runs after Task B commits (reviews implementation + E2E together)
- **Step 6 (NEXT FEATURE)**: Task B for Feature N can run in parallel with Task A for Feature N+1 if they don't share files

Update the mermaid state diagram to show the two-task structure.

Update the **Batch Controllers** section: batch size stays at 3 features, but each feature now has 2 tasks (6 tasks per batch). Adjust wording accordingly.

Update **Agent-Driven Model** and **Hybrid Model** sections with the same two-task split.

Update the **Completion** section: "Run full E2E test suite one final time" stays unchanged.

### 2. `references/generate-plans.md` — Update plan rules

Edit the **Plan Rules** section:

- Change: "Each feature is an atomic commit boundary (implementation + E2E tests committed together)"
- To: "Each feature produces two commits: implementation (with unit/integration tests) and E2E tests. The implementation commit is the atomic boundary for the feature's behavior. The E2E commit verifies it."

No other changes to plan generation — the feature template, AC format, and E2E spec format all stay the same.

### 3. `SKILL.md` — Update project rules and E2E principles

**Project Rules section:**
- Update "Tests with every change": clarify that implementation commits include unit/integration tests; E2E tests follow in a separate commit
- Update "Commits": reference the two commit conventions

**E2E-Specific Principles section:**
- Change "Deferred commits: Do NOT commit until all E2E tests pass for a feature" to: "Two-stage commits: Implementation commits after unit tests pass. E2E commits after E2E tests pass."
- Keep "Structural traceability" unchanged (ACs still have E2E specs beneath them in the plan)

### 4. `config.md` — Add E2E commit convention

**Commit Conventions table** — add:
```
| E2E test commit | `test(<scope>): e2e tests for <feature>` |
```

### 5. `references/implementer-prompt.md` — Scope down

Update the implementer prompt template:
- "Your Job" step 2: change from "Write tests following TDD (red-green cycle)" to "Write unit/integration tests following TDD (red-green cycle). E2E tests are handled separately."
- Remove any mention of E2E tests from the implementer's responsibilities
- The report format stays the same (FILES, TESTS, COMMIT, ISSUES, BLOCKED)

### 6. New: E2E test writer prompt concept

Add guidance in `execute-loop.md` for dispatching the E2E task. When using agent teams, the E2E task gets a focused prompt:
- Receives: the feature spec (ACs + E2E test specs from the plan), the implementation commit SHA, and the test file path
- Job: write E2E tests matching the specs, run them, fix loop, commit
- Same structured return format as implementer

No new file needed — the guidance lives in the execute-loop's Task B description.

## Files to Modify

| File | Change |
|------|--------|
| [execute-loop.md](../../plugins/pmp/skills/pmp/references/execute-loop.md) | Split per-feature loop into two tasks (main change) |
| [generate-plans.md](../../plugins/pmp/skills/pmp/references/generate-plans.md) | Update atomic boundary rule in Plan Rules |
| [SKILL.md](../../plugins/pmp/skills/pmp/SKILL.md) | Update project rules and E2E principles |
| [config.md](../../plugins/pmp/skills/pmp/config.md) | Add E2E test commit convention |
| [implementer-prompt.md](../../plugins/pmp/skills/pmp/references/implementer-prompt.md) | Remove E2E responsibility from implementer |

## What Stays the Same

- Feature template (`assets/feature.md`) — no format changes
- Plan template (`assets/plan.md`) — no structure changes
- E2E spec format (6 fields per AC) — kept as-is
- All ACs get E2E tests — no tier system
- Batch size (3 features per controller session)
- Fix loop ceiling (3 attempts)
- E2E opt-out mechanism for projects where E2E doesn't apply
- Two-stage review (spec + code quality)
- GitHub Issues integration

## Parallelism Opportunity

The two-task model unlocks a parallelism option: Task B (E2E for Feature N) can overlap with Task A (implementation of Feature N+1) **if they don't write to the same files**. This is optional — the execute-loop should document it as a possibility but not require it. The existing file-ownership rules from the parallel agent protocol apply.

## Verification

1. Read through the updated execute-loop and confirm the two-task flow is clear
2. Verify the mermaid diagram accurately reflects the new structure
3. Check that batch controller math works (3 features × 2 tasks = up to 6 tasks per session)
4. Confirm the implementer prompt no longer mentions E2E
5. Verify generate-plans.md and SKILL.md are consistent with the new commit model
