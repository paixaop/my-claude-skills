# Execute Loop — Additional Models & Completion

> Part 2 of the execute loop. See [execute-loop.md](execute-loop.md) for the core loop, context management, and code-file model.

---

## Per-Feature Loop (Agent-Driven Model)

For web apps using Playwright MCP instead of code-file tests. Same two-task structure as the code-file model.

### Task A: IMPLEMENT

Same as code-file model Task A: implement feature with unit TDD if applicable, run unit tests, fix loop, commit implementation.

### Task B: E2E TESTS (Agent-Driven)

Runs after Task A commits successfully.

#### B1: Write Test Spec

- Read the E2E test cases under the current feature's acceptance criteria
- Write the test spec file to the PROJECT's test directory: `<test-dir>/<feature>-tests.md`
  (e.g., `e2e/providers-tests.md` -- inside the project, NOT inside the skill directory)
- Do NOT commit yet

#### B2: Execute Tests via Playwright MCP

Agent-driven tests are expensive (each action = MCP call + snapshot). Use a tiered strategy:

**a. Run the NEW suite in full:** Execute all test cases for the current feature. For each test case:
- Follow the steps in the spec exactly
- After every browser action (click, fill, navigate), take a snapshot
- Verify assertions against the snapshot
- Record PASS/FAIL per test case

**b. Run SMOKE CHECK of previous suites:** Execute only the FIRST test case of each prior suite to verify no regressions. Full re-run of all prior suites is too expensive per feature.

**c. Record results** for all executed test cases.

#### B3: Fix Loop

Same hard ceiling (see [config.md](../../pmp/config.md) Thresholds) and classification as code-file model, with adjustments:

- **FLAKY**: Add explicit `browser_wait_for` calls before assertions in the test spec
- **SPEC MISMATCH**: Update both the test spec file and the plan
- **Re-runs**: Only re-execute the failing suite(s), not the full smoke check again

#### B4: Commit E2E Tests

Only after the new suite passes AND the smoke check passes:

- Stage test spec files only
- Commit: `test(<scope>): e2e test specs for <feature>` (see [config.md](../../pmp/config.md) Commit Conventions)
- Run project CI command if applicable
- **Comment on GitHub Issue** (if issues exist for this plan) using format from [config.md](../../pmp/config.md) GitHub Conventions — do NOT close; the PR will close it on merge
- **Two-stage review:** Same as code-file model Task B, Step B5 (spec compliance then code quality) when using agent teams
- Mark feature complete in TodoWrite

### Parallelism and Batch Handoff

Same as code-file model: Task B for Feature N can overlap with Task A for Feature N+1 if no file conflicts. Continue within batch, or write checkpoint and hand off to fresh session.

---

## Hybrid Model (Fullstack)

When the plan uses hybrid execution (code-file for API, agent-driven for UI):

- Task A: implement feature with unit tests, commit implementation
- Task B: write code-file E2E tests for API suites AND agent-driven spec files for UI suites
- In B2: run code-file test runner for API suites, then execute agent-driven specs for UI suites
- In B3: apply fixes and re-run only the failing model's tests
- In B4: commit all E2E artifacts (test files + test specs) together

---

## Plan Correction Protocol

When the agent discovers a spec mismatch (the plan is wrong about expected behavior):

1. **Fix the test code/spec** to match actual correct behavior
2. **Add a correction note** to the affected E2E test case in the plan file:
   ```markdown
   **Correction (YYYY-MM-DD):** Changed expected status from 200 to 201.
   Reason: POST endpoint correctly returns 201 Created per REST conventions.
   ```
3. Do NOT silently deviate -- every plan correction must be visible in the plan file

---

## Completion

After all features are implemented and pass:

1. **Run full E2E test suite one final time** -- ALL suites, ALL test cases — **skip if docs-only plan**
   - For agent-driven: this is the full regression run (only time all suites run completely)
   - For code-file: same as any other run (test runner runs everything)
2. **Run project CI command** (from plan header) — **skip if docs-only plan**
3. **Print final results table:**

```
Feature         | E2E Tests | Status
────────────────┼───────────┼───────
Feature 1       | 5/5       | PASS
Feature 2       | 3/3       | PASS
Feature 3       | 4/4       | PASS
────────────────┼───────────┼───────
Total           | 12/12     | ALL PASS
```

4. **Ask the user:** "All features pass and CI is clean. Want me to create a PR?"
5. **If yes — build PR body with closing keywords** (if GitHub Issues exist for this plan):
   - Read the `## GitHub Issues` table from the plan file
   - Collect ALL issue numbers at all levels (task issues, feature issues, AND epic)
   - Create the PR with `Closes #N` for every issue:
   ```bash
   gh pr create --title "feat: <feature-set-name>" --body "$(cat <<'EOF'
   ## Summary
   <summary of all implemented features>

   ## Closes
   Closes #<task-A-1>
   Closes #<task-B-1>
   Closes #<feature-1>
   Closes #<task-A-2>
   Closes #<task-B-2>
   Closes #<feature-2>
   Closes #<epic>

   ## E2E Test Results
   | Feature | Tests | Status |
   |---------|-------|--------|
   | Feature 1 | 5/5 | PASS |
   | Feature 2 | 3/3 | PASS |
   | Total | 8/8 | ALL PASS |
   EOF
   )"
   ```
   - Update the `## GitHub Issues` table in the plan file — set all statuses to `Closes via PR #<number>`
   - **Do NOT** manually run `gh issue close` — GitHub auto-closes all listed issues when the PR merges
6. **If yes — no GitHub Issues:** Create PR normally with `gh pr create` summarizing features and E2E results
7. **Update frontmatter (MANDATORY):** Update the plan file's YAML frontmatter (see [config.md](../../pmp/config.md) Plan Frontmatter):
   - Set `status: implemented`
   - Set `completed_at` to the current UTC timestamp
   - Set `pr: "#<number>"` (the PR number just created, or blank if user declined PR)
   - Preserve all other frontmatter fields from prior stages
8. **Archive the plan:** Move the plan file from plans directory to completed plans directory `docs/plans/implemented/` (see [config.md](../../pmp/config.md) File Paths; create directory if it doesn't exist):
   ```bash
   mkdir -p docs/plans/implemented
   mv docs/plans/<plan-file>.md docs/plans/implemented/<plan-file>.md
   ```
9. **Commit and push plan archive:**
   ```bash
   git add docs/plans/ docs/plans/implemented/
   git commit -m "chore(<scope>): archive plan <plan-name>"
   git push
   ```
10. **Update PR with plan link (if PR was created):**
    Append a "Plan" section to the PR body linking to the archived plan file:
    ```bash
    gh pr edit <pr-number> --body "$(gh pr view <pr-number> --json body -q .body)

    ## Plan

    [<plan-name>](docs/plans/implemented/<plan-file>.md)"
    ```
    Skip this step if the user declined PR creation.
11. **Update source review file (if applicable):** Check if the plan has a `source_review` field in its frontmatter (set by discuss-generated plans). If yes:
   - Read the source review file
   - Append a `## Resolution` section documenting:
     - **Fixed:** Which findings were fixed, with a brief description of what was done
     - **Acknowledged:** Which findings were acknowledged, with the rationale (spec files were annotated with GitHub blockquote alerts during discussion)
     - **Deferred:** Which findings were deferred, with the reason
     - **Decisions:** Key decisions made during discussion and execution, with explanations
     - **Date:** Resolution date and link to the PR (if created)
   - This creates a complete audit trail from review → discussion → resolution
12. **Offer release notes:** Ask the user: "Want me to generate release notes for this implementation?" If yes, read [changelog.md](../../changelog/references/changelog.md) and follow it — the plan file and PR number are already available from this session.
13. **Announce** with completion message from [config.md](../../pmp/config.md) Stage Announcements

---

## Test Only Mode

When the user just wants to re-run E2E tests (no new coding):

1. Read the plan's E2E Test Infrastructure section to find the test runner command (code-file) or test spec files (agent-driven)
2. Execute all E2E test suites
3. Report results in a summary table:

```
Suite           | Tests   | Status
────────────────┼─────────┼───────
Suite 1         | 5/5     | PASS
Suite 2         | 3/3     | PASS
────────────────┼─────────┼───────
Total           | 8/8     | ALL PASS
```

4. If failures found, offer to enter fix mode (fix loop from the per-feature loop above)

For agent-driven tests, the user can specify which suites to run:
- No arguments: run all suites
- Suite names: run only those suites

Project detection details (type, conventions, monorepo) are in the plan header — no need to re-detect.

---

## When to Stop and Ask

STOP immediately and ask the user when:

- Deterministic E2E test failure persists after fix loop ceiling (see [config.md](../../pmp/config.md) Thresholds)
- New feature breaks a previously-passing E2E suite and the regression fix is non-obvious
- Missing infrastructure (DB, service, API key) that can't be mocked or substituted
- Ambiguous requirement that affects test correctness and can't be resolved from the plan
- Flaky test persists after adding retries/waits (may indicate a real race condition)

In all cases: report what's broken with full context (failure output, classification, attempted fixes), then ask the user.
