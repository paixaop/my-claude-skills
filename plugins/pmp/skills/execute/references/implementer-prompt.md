# Implementer Subagent Prompt Template

Use this template when dispatching an implementer subagent via the Task tool.

**Context budget:** Implementer agents do the most work and are most at risk of context exhaustion. Keep the prompt lean — only include what the agent needs to start working. Point to `CLAUDE.md` for project conventions instead of inlining them.

```
Task tool (generalPurpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N: [task name]

    ## Project Context

    Read CLAUDE.md for project conventions. Key commands:
    - Build: [command]
    - Test: [command] (use concise output: --tb=short / --reporter=dot / equivalent)
    - Lint: [command]
    - CI: [command]

    ## Task Description

    [FULL TEXT of this specific task from the plan — paste it here, don't make
    subagent read the plan file. Do NOT include other tasks or the full plan.]

    ## Context

    [2-3 sentences max: where this task fits, what previous tasks produced,
    key files to look at. NOT a full architectural overview.]

    ## Before You Begin

    If you have questions about requirements, approach, or dependencies — ask now.

    ## Scope

    You are implementing exactly ONE file: [file path]
    Action: [Create | Modify]

    ## Your Job

    1. Implement exactly what the task specifies for THIS FILE ONLY — do not touch other files
    2. **Red-Green-Refactor for test files:**
       - Write the test FIRST
       - Run it and verify it FAILS (RED) — this proves the test catches the missing feature
       - Write minimal implementation to make the test pass (GREEN)
       - Run it and verify it PASSES
       - Refactor if needed, verify tests STILL PASS (REFACTOR)
       - If the test passes immediately in RED, the test is wrong — fix it before proceeding
    3. **For implementation files:** Write the code, then verify the corresponding test (if already written by a prior task agent) passes
    4. Verify: build passes, lint clean, tests pass (use concise output flags)
    5. Commit with conventional commit message: feat(<scope>): <feature>
    6. Self-review: does this match the spec requirements quoted in the AC?
    7. Report back using the EXACT format below

    While working: if you encounter something unexpected, ask — don't guess.

    ## Context Budget Rules

    Your context window is finite. Protect it:
    - Do NOT re-read files you just wrote (unless checking something specific)
    - Use concise test output flags (--tb=short, --reporter=dot, etc.)
    - If build/test output exceeds ~50 lines, read only the last 50 lines
    - Avoid reading the entire plan file — everything you need is above

    ## Report Format (MANDATORY — use this exact structure)

    ```
    FILE: [the single file path this task targeted]
    ACTION: [Created | Modified]
    TESTS: [N passed, M failed]
    RGR: [RED: test failed as expected | SKIP: not a test task] → [GREEN: test passes] → [REFACTOR: still passes | SKIP: no refactor needed]
    COMMIT: [short SHA]
    ISSUES: none | [1-sentence description per issue, max 3]
    BLOCKED: no | [1-sentence reason]
    ```

    Do NOT include: full test output, full file contents, lengthy explanations,
    or the task description repeated back. The controller needs a summary, not a novel.
```
