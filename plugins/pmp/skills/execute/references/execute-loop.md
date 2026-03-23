# Execute Loop

Implements features using single-file tasks with parallel agent dispatch. Each feature gets an orchestrator agent that manages its task agents, reviews spec compliance, and enforces Red-Green-Refactor.

## Execution Architecture

```
Controller (session, max 3 features per batch)
  └── Feature Orchestrator Agent (one per feature)
      ├── Read feature task list + dependency graph from plan
      ├── Dispatch independent impl tasks as parallel agents (one file per agent)
      ├── Wait for dependency resolution, dispatch next wave
      ├── After all impl tasks: run spec review (does code match spec requirements quoted in ACs?)
      ├── Dispatch test task agents (parallel where independent)
      ├── Verify Red-Green-Refactor: each test MUST fail (RED) before passing (GREEN)
      ├── After all tests: run code quality review
      └── Report structured summary to controller
```

**Key principles:**
- Orchestrator does NOT implement code — only dispatches, reviews, coordinates
- Each task agent creates/modifies exactly ONE file
- Tasks on the same dependency level run as parallel agents
- Features with no mutual dependencies (per feature dependency matrix) run as parallel orchestrators
- Red-Green-Refactor: test agents must demonstrate RED (failure) before GREEN (pass)

**Master plan execution:** Sub-plans execute in phase order. Each phase is a separate controller session. Phase N+1 starts only after Phase N exit criteria are met.

## Legacy Two-Task Model (fallback)

For plans that use the older two-task format (Task A: all implementation, Task B: all E2E), the following execution model applies:

```mermaid
stateDiagram-v2
    [*] --> Setup

    state Setup {
        [*] --> ReadPlan
        ReadPlan --> CheckoutBranch
        CheckoutBranch --> CheckGHIssues
        CheckGHIssues --> SetupInfra
        SetupInfra --> VerifyRunner
        VerifyRunner --> InfraFailed : runner fails
        VerifyRunner --> Ready : runner works
        InfraFailed --> [*] : STOP — ask user
        Ready --> [*]
    }

    Setup --> FeatureN : for each feature in dependency order

    state FeatureN {
        [*] --> TaskA

        state TaskA {
            [*] --> Implement
            Implement --> RunUnitTests
            RunUnitTests --> UnitFixLoop : unit tests fail
            RunUnitTests --> CommitImpl : all pass

            state UnitFixLoop {
                [*] --> ParseUnitFailure
                ParseUnitFailure --> ClassifyUnit
                ClassifyUnit --> FixUnitImpl
                FixUnitImpl --> RerunUnit
                RerunUnit --> UnitCounter
                UnitCounter --> [*] : tests pass
                UnitCounter --> ParseUnitFailure : < ceiling
                UnitCounter --> UnitBlocked : ceiling reached
            }

            UnitFixLoop --> CommitImpl : tests pass
            UnitFixLoop --> AskUser : blocked

            CommitImpl --> RunCI_A
            RunCI_A --> TaskADone : CI pass
            RunCI_A --> FixCI_A : CI fail
            FixCI_A --> RunCI_A
            TaskADone --> [*]
        }

        TaskA --> TaskB

        state TaskB {
            [*] --> WriteE2E
            WriteE2E --> RunAllE2E

            RunAllE2E --> E2EFixLoop : any test fails
            RunAllE2E --> CommitE2E : all pass

            state E2EFixLoop {
                [*] --> ParseE2EFailure
                ParseE2EFailure --> ClassifyE2E
                ClassifyE2E --> FixE2EImpl : DETERMINISTIC BUG
                ClassifyE2E --> FixE2ETest : FLAKY
                ClassifyE2E --> FixE2EBoth : SPEC MISMATCH
                ClassifyE2E --> RestartInfra : INFRASTRUCTURE

                FixE2EImpl --> RerunE2E
                FixE2ETest --> RerunE2E
                FixE2EBoth --> RerunE2E
                RestartInfra --> RerunE2E

                RerunE2E --> E2ECounter
                E2ECounter --> [*] : tests pass
                E2ECounter --> ParseE2EFailure : < ceiling
                E2ECounter --> E2EBlocked : ceiling reached
            }

            E2EFixLoop --> CommitE2E : tests pass
            E2EFixLoop --> AskUser : blocked

            CommitE2E --> RunCI_B
            RunCI_B --> Review : CI pass
            RunCI_B --> FixCI_B : CI fail
            FixCI_B --> RunCI_B
            Review --> TaskBDone
            TaskBDone --> [*]
        }

        TaskB --> FeatureDone
        FeatureDone --> [*]
    }

    FeatureN --> FeatureN : next feature
    FeatureN --> Completion : all features done

    state Completion {
        [*] --> FinalE2ERun
        FinalE2ERun --> FinalCI
        FinalCI --> PrintResults
        PrintResults --> AskPR
        AskPR --> CreatePR : user says yes
        AskPR --> UpdateFrontmatter : user says no
        CreatePR --> PRClosesIssues : GH issues exist (Closes #N in body)
        CreatePR --> UpdateFrontmatter : no GH issues
        PRClosesIssues --> UpdateFrontmatter
        UpdateFrontmatter --> ArchivePlan : move to docs/plans/implemented/
        ArchivePlan --> CommitPlanArchive
        CommitPlanArchive --> UpdatePRWithPlan : PR was created
        CommitPlanArchive --> UpdateReview : no PR
        UpdatePRWithPlan --> UpdateReview : if source_review exists
        UpdatePRWithPlan --> OfferRelease : no source_review
        UpdateReview --> OfferRelease
        OfferRelease --> [*]
    }

    Completion --> [*] : done
```

## E2E Principles

These principles apply across all execution models:

- **Specs, not code:** Plans describe WHAT to build/test, never HOW (no code snippets)
- **Structural traceability:** Every AC has its E2E test case directly beneath it — no cross-references needed
- **Two-stage commits:** Implementation commits after unit/integration tests pass. E2E test commits after E2E tests pass. Each feature produces two commits.
- **Fix loop ceiling:** see [config.md](../../pmp/config.md) Thresholds for max attempts per feature
- **Test isolation:** Each feature's tests handle their own setup/teardown, runnable independently
- **No hardcoded conventions:** Integration branch, CI command, test directory all come from project detection (recorded in the plan header)

---

## Context Management

Read [config.md](../../pmp/config.md) Context Management before starting. The rules below are critical for preventing context window exhaustion during execution.

### Batch Controllers

Do NOT execute the entire plan in a single controller session. Split features into batches of **3** (see config.md Max features per controller session).

**With single-file task model:** The controller dispatches up to 3 feature orchestrators per batch. Each orchestrator manages its own task agents. Independent features (per the feature dependency matrix) run as parallel orchestrators.

**With legacy two-task model:** Each feature has two tasks (Task A: implementation, Task B: E2E tests), so each batch contains up to 6 tasks.

**Phase-aware batching:** If the plan has a `## Phases` section, use phase boundaries as batch boundaries instead of the arbitrary 3-feature cutoff. Each phase becomes a batch (or multiple batches if a phase has more than 3 features). Never split a phase across batches — complete all features in a phase before moving to the next. Phase exit criteria must be met before starting the next phase.

**Per-batch workflow:**
1. Controller reads the plan once and identifies the next batch of features
2. Controller dispatches subagents for each feature's tasks in the batch. Task A and Task B are sequential within a feature, but Task B for Feature N can overlap with Task A for Feature N+1 if they don't write to the same files (see Parallel Agent Dispatching rules).
3. After the batch completes, controller writes a **checkpoint summary**:
   ```
   BATCH: [N] of [total]
   COMPLETED: Feature 1 (commit abc), Feature 2 (commit def), Feature 3 (commit ghi)
   BRANCH: feat/<name> at commit ghi
   REMAINING: Feature 4, Feature 5, ...
   ISSUES: none | [brief list]
   ```
4. A fresh controller session picks up from the checkpoint to execute the next batch
5. The fresh controller reads only: the plan file, the checkpoint summary, and the current branch state — NOT the full history of previous batches

### Concise Test Output

Always use concise output flags for test runners. Verbose test output is the #1 context consumer in fix loops.

| Runner | Concise Flag | Example |
|--------|-------------|---------|
| pytest | `--tb=short -q` | `pytest --tb=short -q` |
| Jest | `--silent --verbose=false` | `npx jest --silent` |
| Go test | pipe through `tail -50` | `go test ./... 2>&1 \| tail -50` |
| Playwright | `--reporter=dot` | `npx playwright test --reporter=dot` |
| Vitest | `--reporter=dot` | `npx vitest --reporter=dot` |

On failure, read only the **last 50 lines** of output to identify the failing test and error message. Do NOT capture the full multi-page output into context.

### Subagent Returns

All subagents MUST use the structured return format defined in [config.md](../../pmp/config.md) Context Management. The controller should extract only the structured fields and discard verbose prose.

### Fix Loop Context

Each fix loop iteration consumes context. When dispatching a fix agent (or resuming an implementer to fix issues):
- Provide ONLY: the failing test name, the error message (1-3 lines), and the classification
- Do NOT include: full test output, previous fix attempts, or the full task description again
- The agent already has the code — it only needs to know what broke

---

## Multi-Session Resume

When returning to an in-progress execution after a session break (hours or days later), follow this protocol before continuing:

### 1. Verify State

```bash
# What branch are we on?
git branch --show-current

# What was the last commit?
git log --oneline -5

# Is there uncommitted work?
git status

# Has the integration branch advanced?
git fetch origin
git log HEAD..<integration-branch> --oneline
```

### 2. Check for Conflicts

If the integration branch has new commits since execution started:

```bash
# Test merge without committing
git merge --no-commit --no-ff origin/<integration-branch>
# If conflicts: abort and ask the user
git merge --abort
```

Use AskQuestion:
1. **Rebase onto latest** — `git rebase origin/<integration-branch>` then re-run all E2E tests
2. **Merge latest in** — `git merge origin/<integration-branch>` then re-run all E2E tests
3. **Continue without merging** — proceed from current state (may cause merge conflicts later)

After merging/rebasing, re-run the full E2E test suite before continuing with new features. If tests fail, enter the fix loop.

### 3. Find the Checkpoint

- Read the plan file (check `status` in frontmatter — should be `executing`)
- Look for the most recent checkpoint summary in the conversation or plan file
- If no checkpoint: check TodoWrite for which features are marked complete, verify with `git log`

### 4. Verify Completed Work

Run the E2E test suite to confirm previously-implemented features still pass:

```bash
# Run with concise output
<test-runner-command> <concise-flags>
```

If any previously-passing tests now fail (due to environment changes, dependency updates, etc.), fix them before continuing.

### 5. Resume

Continue from the next uncompleted feature in the plan. Re-read the plan file and the current batch boundaries. Proceed with the normal per-feature loop.

---

## Before Starting

1. **Read the plan completely**
2. **Update frontmatter:** Set `status: executing` and `execution_started_at` to the current UTC timestamp in the plan file (see [config.md](../../pmp/config.md) Plan Frontmatter)
3. **Read integration branch** from the plan header:
   ```
   git checkout <branch> && git pull && git checkout -b feat/<name>
   ```
4. **Read CI command** from the plan header
5. **Detect docs-only plan:** Check if ALL features in the plan only modify markdown, documentation, or architectural files (no code changes). If so, mark the plan as `docs_only: true` and **skip all CI gates** throughout execution (Task A CI, Task B CI, Final CI). This applies to plans generated by `/pmp:discuss` that only fix specs/docs, or any plan where every feature is a spec/doc change.
6. **Check for GitHub Issues:** Look for a `## GitHub Issues` section in the plan. If present, note the 3-level mapping (epic → feature issues → task issues). You'll comment on task issues as tasks are committed and include `Closes #N` for all issues in the PR body.
7. **Create TodoWrite** with all features from the plan
8. **Determine batch boundaries** — split features into batches of 3 (see Context Management above). Each feature has 2 tasks (implementation + E2E), so each batch has up to 6 tasks. Only execute the first batch in this session.
9. **Set up E2E test infrastructure** (ALWAYS the first task):
   - Follow the plan's "E2E Test Infrastructure" section
   - Install framework, configure test runner, set up environment
10. **Verify the test runner works**:
   - Run the test runner command with concise output flags (see Context Management above) -- should exit cleanly even with no tests
   - For agent-driven model: verify Playwright MCP is available and browser launches
11. **If infrastructure setup fails**: STOP immediately. Report the failure with full error output. Do NOT proceed to feature implementation. E2E tests are the verification backbone -- coding without them defeats the purpose of this skill.

## Agent Coordination Protocol

When executing features with agent teams (Task tool), follow these rules strictly.

### Task Dependencies

When creating tasks with `TaskCreate`, always set up dependency chains with `TaskUpdate` using `addBlocks`/`addBlockedBy` for tasks that have ordering requirements.

**Agents MUST follow this protocol:**
1. Before claiming a task, call `TaskGet` and verify `blockedBy` is empty or all blocking tasks are `completed`
2. Never start a blocked task — if all available tasks are blocked, notify the team lead and wait
3. After completing a task, call `TaskList` to check if any tasks were unblocked
4. When a task produces artifacts (files, configs, schemas) that downstream tasks depend on, include the artifact paths in the task description so dependent agents know what to read

**Team lead responsibilities:**
- Create ALL tasks with explicit dependency edges before spawning agents
- Stagger agent spawning: launch agents for unblocked tasks first, then spawn more as tasks complete
- When assigning tasks via `TaskUpdate`, double-check the task isn't blocked
- Include in each task's description: what it depends on, what files it reads/writes, and what it produces

### Parallel Agent Dispatching

1. **Only parallelize truly independent work** — if task B reads files that task A writes, they are NOT independent. Run A first, then B.
2. **Map file ownership before dispatching** — list which files each agent will read/write. If two agents write the same file, they MUST be serialized.
3. **Split into waves** — group independent tasks into waves. Launch wave N+1 only after wave N completes. Never launch dependent work speculatively.
4. **Include full context in each agent's prompt** — parallel agents share nothing. Each prompt must contain all file paths, decisions, and constraints it needs. Never assume an agent can see another agent's output.
5. **Consolidate after parallel waves** — after a parallel wave completes, review all outputs for conflicts before launching the next wave.
6. **Lean prompts** — include only the file paths, task-specific decisions, and constraints each agent needs. Point to `CLAUDE.md` for project conventions instead of inlining them. Never dump the full plan into every agent prompt.
7. **Demand structured returns** — every agent MUST return in the structured format defined in [config.md](../../pmp/config.md) Context Management. Extract only the structured fields from agent returns — discard prose.
8. **Use fast model for reviewers** — spec compliance and code quality reviewers are focused tasks that benefit from `model: fast`.

Never launch parallel agents that write to the same files or where one agent's output is another's input. When in doubt, serialize.

### Practical Wins

1. **Explicit file ownership in task descriptions** — "This task writes `streaming.go`, no other task should modify it" prevents overwrite conflicts
2. **Staggered spawning** — don't launch 5 agents if only 2 tasks are unblocked
3. **Artifact paths in descriptions** — telling an agent "read the schema from `internal/db/schema.go` (produced by task 1)" is more reliable than hoping it checks

---

## Unit TDD Within Features

This loop does NOT replace unit-level TDD. Within step 1 (IMPLEMENT) of each feature, if the project uses TDD (check for existing test patterns, TDD skill, or project docs), follow Red-Green-Refactor for unit tests. The E2E test in step 2 is an additional integration verification layer, not a replacement for unit tests.

---

## Per-Feature Loop (Code-File Model)

Each feature has two tasks. Task A (implementation + unit tests) and Task B (E2E tests) each get their own commit.

For each feature in dependency order:

### Task A: IMPLEMENT

#### A1: Implement

- Read the feature spec from the plan
- Implement the feature (apply unit TDD if project uses it)
- Run existing unit tests to verify no regressions
- Do NOT commit yet

#### A2: Run Unit/Integration Tests

- Execute the project's unit test command with concise output flags
- On success: note pass count only
- On failure: read only the last 50 lines to identify failing test and error

#### A3: Fix Loop (Unit Tests)

**Hard ceiling: see [config.md](../../pmp/config.md) Thresholds for fix loop ceiling.**

Same classification and fix protocol as the E2E fix loop (see Task B below), applied to unit/integration test failures.

#### A4: Commit Implementation

Only after all unit/integration tests pass:

- Stage implementation code + unit/integration test files
- Commit: `feat(<scope>): <feature>` (see [config.md](../../pmp/config.md) Commit Conventions)
- Run the project CI command (from plan header) — **skip if docs-only plan**
- If CI fails, fix and amend the commit
- **Comment on Task A Issue** (if GitHub Issues exist for this plan): `gh issue comment <task-A-number> --body "Implemented in <commit-sha>"`

### Task B: E2E TESTS

Runs after Task A commits successfully.

#### B1: Write E2E Tests

- Read the E2E test cases under the current feature's acceptance criteria
- Write the E2E test file to the test directory following the specs exactly
- Do NOT commit yet

#### B2: Run All E2E Tests

- Execute the test runner command with concise output flags (runs ALL E2E suites, not just the new one)
- On success: note pass count only (do NOT capture full output into context)
- On failure: read only the last 50 lines to identify failing test and error

#### B3: Fix Loop (E2E Tests)

**Hard ceiling: see [config.md](../../pmp/config.md) Thresholds for fix loop ceiling.**

If any test fails:

**a. Parse the failure** -- which test, what assertion, actual vs expected

**b. Classify the failure:**

| Category | Signal | Action |
|---|---|---|
| **DETERMINISTIC BUG** | Same test fails the same way every run | Fix the implementation code |
| **FLAKY** | Test passes on immediate retry | Add retry/wait logic to the test |
| **SPEC MISMATCH** | Test expectation is wrong (e.g., expected 200 but 201 is correct) | Fix test code AND update the plan (see Plan Correction below) |
| **INFRASTRUCTURE** | Server crashed, DB down, port conflict | Restart infrastructure and re-run |

**c. Apply the fix**

**d. Re-run ALL E2E tests**

**e. Increment the attempt counter** (regardless of which category)

**f. If ceiling reached (see [config.md](../../pmp/config.md) Thresholds):** STOP. Report concisely:
- Which test(s) are failing (name + 1-line error)
- Failure classification per attempt (1 line each)
- What was tried (1 line per attempt)

Ask the user for help before proceeding.

#### B4: Commit E2E Tests

Only after ALL E2E tests pass:

- Stage E2E test files only
- Commit: `test(<scope>): e2e tests for <feature>` (see [config.md](../../pmp/config.md) Commit Conventions)
- Run the project CI command (from plan header) — **skip if docs-only plan**
- If CI fails, fix and amend the commit
- **Comment on Task B Issue** (if GitHub Issues exist for this plan): `gh issue comment <task-B-number> --body "E2E tests in <commit-sha>"` — do NOT close; the PR will close it on merge

#### B5: Two-Stage Review (Subagent Mode)

When using agent teams (Task tool) for implementation, run a two-stage review after Task B commits. Skip this step if implementing directly (no subagents). The review covers both the implementation (Task A) and E2E tests (Task B) together.

**Stage 1: Spec Compliance Review**

Dispatch a spec reviewer using [spec-reviewer-prompt.md](../../pmp/references/spec-reviewer-prompt.md):
- Reviewer reads actual code — does NOT trust the implementer's report
- Verifies: nothing missing, nothing extra, nothing misunderstood
- If issues found: implementer fixes, spec reviewer re-reviews. Loop until approved.

**Stage 2: Code Quality Review**

Only after spec compliance passes. Dispatch a code quality reviewer using [code-quality-reviewer-prompt.md](code-quality-reviewer-prompt.md):
- Reviews code clarity, error handling, test quality, security, patterns
- If issues found: implementer fixes, quality reviewer re-reviews. Loop until approved.

Both reviewers use `model: fast` (focused read-and-compare tasks). See the prompt templates for the exact dispatch format and mandatory structured return format.

- Mark feature complete in TodoWrite

### Parallelism Opportunity

Task B for Feature N can overlap with Task A for Feature N+1 **if they don't write to the same files**. Apply the existing file-ownership rules from the Parallel Agent Dispatching section. When in doubt, serialize.

### Next Feature or Batch Handoff

If more features remain in the current batch: go to Task A for the next feature.

If the current batch is complete but more features remain in the plan:
1. Write a checkpoint summary (see Context Management above)
2. Report to the user: "Batch N complete. N features remaining. Start a fresh session to continue."
3. The fresh controller session reads: plan file + checkpoint summary + branch state

If all features are done: proceed to Completion.

---

> **Continued in [execute-loop-models.md](execute-loop-models.md)** — covers Agent-Driven Model, Hybrid Model, Plan Correction Protocol, Completion, Test Only Mode, and When to Stop and Ask.
