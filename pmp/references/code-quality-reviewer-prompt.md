# Code Quality Reviewer Prompt Template

Use this template when dispatching a code quality reviewer subagent.

**Purpose:** Verify the implementation is well-built — clean, tested, maintainable.

**Only dispatch AFTER spec compliance review passes.**

**Model:** Use `fast` model — this is a focused review task.

```
Task tool (code-reviewer subagent_type, model: fast):
  description: "Code quality review for Task N: [task name]"
  prompt: |
    Review the code changes for Task N: [task name]

    ## What Was Implemented

    [From implementer's structured return — just FILES and COMMIT lines]

    ## Requirements Summary

    [2-3 sentence summary of what the task required — NOT the full task text]

    ## Changes to Review

    Base SHA: [commit before task]
    Head SHA: [current commit]

    Run `git diff <base>..<head>` to see the changes.

    ## Project Standards

    Read CLAUDE.md for full conventions. Key points:
    - Language/framework: [language]
    - Test coverage target: 60%
    - Complexity limits: [tool and threshold if applicable]

    ## Review Focus

    - Code clarity and maintainability
    - Error handling completeness
    - Test quality (behavior verification, not mock verification)
    - Adherence to existing patterns
    - Security (input validation, auth, injection risks)
    - Performance implications

    ## Report Format (MANDATORY — use this exact structure)

    ```
    VERDICT: ✅ approved | ❌ needs changes
    CRITICAL: none | [bullet per issue with file:line — must fix]
    IMPORTANT: none | [bullet per issue with file:line — should fix]
    MINOR: none | [bullet per issue with file:line — nice to have]
    ```

    Keep it to the structured fields above. Do NOT write lengthy analysis —
    the controller has limited context budget.
```
