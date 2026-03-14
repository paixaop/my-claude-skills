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

    ## Your Job

    1. Implement exactly what the task specifies
    2. Write unit/integration tests following TDD (red-green cycle). E2E tests are handled separately — do NOT write E2E tests.
    3. Verify: build passes, lint clean, tests pass (use concise output flags)
    4. Commit with conventional commit message: feat(<scope>): <feature>
    5. Self-review: completeness, quality, YAGNI, test coverage, security
    6. Fix any self-review issues before reporting
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
    FILES: [comma-separated list of changed file paths]
    TESTS: [N passed, M failed]
    COMMIT: [short SHA]
    ISSUES: none | [1-sentence description per issue, max 3]
    BLOCKED: no | [1-sentence reason]
    ```

    Do NOT include: full test output, full file contents, lengthy explanations,
    or the task description repeated back. The controller needs a summary, not a novel.
```
