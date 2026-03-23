---
name: plan-review
description: Use when the user has an existing plan file and says 'review plan', 'review this plan', 'check my plan', or 'is this plan good' — the artifact is a plan file, not specs or architecture docs
---

# PMP: Review

**Announce at start:** "Using pmp:plan-review to review this plan."

Skeptical senior-engineer review of implementation plans. Scores against a checklist covering architecture, security, testing, config, and completeness.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) ONLY for parallel file reading when the corpus is large. All analysis runs in the main controller context — do not spawn agents for analysis phases (they would re-read all files). Track progress with TodoWrite throughout.

## Agent Readability Gate

Every plan will be executed by a coding agent (Claude Code or similar). Review the plan as if YOU are the agent that must implement it. A plan that a human understands but an agent misinterprets is a failed plan.

**Check each feature for:**

| Check | What to look for | Fail if |
|-------|-----------------|---------|
| **Unambiguous instructions** | Every step has exactly one interpretation | Steps use "should", "might", "consider", "as needed", or "appropriate" without defining what that means |
| **Concrete file paths** | Exact paths for every file to create, modify, or read | Paths use "relevant file", "the config", or "appropriate location" |
| **Testable acceptance criteria** | Each criterion maps to a specific assertion | Criteria say "works correctly", "handles errors properly", or "is performant" |
| **No implicit knowledge** | Plan doesn't assume the agent knows project conventions | References to "the usual pattern", "standard approach", or "as we do elsewhere" without showing the pattern |
| **Explicit scope boundaries** | Each feature says what's in AND out of scope | Agent could reasonably gold-plate or under-build because boundaries are fuzzy |
| **Sequential ordering** | Steps within a feature have a clear execution order | Steps could be done in multiple orders with different results |
| **Complete code context** | When plan says "modify function X", it specifies what to change | Agent must read surrounding code and guess the intent |
| **Error behavior specified** | Every operation that can fail says what to do on failure | "Handle errors" without specifying: which errors, what response, what log level |
| **No hand-waving** | No "etc.", "and so on", "similar to above", "repeat for other cases" | Agent must infer what the elided content should be |

**For each failure:** rewrite the section to be agent-executable. Don't just flag it — fix it.

## Common Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| Rubber-stamping plans that "look reasonable" | Missing gaps that cause execution failures | Score every checklist item independently |
| Skipping security gate for non-auth features | Injection, SSRF, and data leaks hide in "boring" features | Always run security gate if plan touches endpoints, data, or config |
| Approving plans with vague acceptance criteria | Coding agent can't implement what isn't specified | Flag every "should handle errors" that lacks specific error types |
| Auto-advancing to execution after approval | User may want to discuss, modify, or publish issues first | Always ask which next step the user wants |

## Red Flags — STOP

- About to say "plan looks good" without scoring every checklist item
- Skipping security gate because "this feature doesn't touch auth"
- Approving a plan with acceptance criteria you couldn't write a test for
- About to auto-advance to execution or GitHub Issues

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **REQUIRED:** Read [review.md](references/review.md) and follow it completely — it contains the review checklist, spec alignment gate, security gate, and verdict loop
3. The review loops until the user picks "proceed", "update", or "discuss"
4. On approval, ask: "Plan approved. Want to publish as GitHub Issues before implementation?"
   - If yes → tell the user to invoke `/pmp:github`
   - If no → tell the user to invoke `/pmp:execute`

## References

- Review output template: [review-output.md](assets/review-output.md)
- Spec alignment: [spec-alignment.md](references/spec-alignment.md)
- Security analysis: [security-analysis.md](references/security-analysis.md)
- Security report template: [security-analysis-output.md](assets/security-analysis-output.md)

## Shared Resources

- **BACKGROUND:** [overview.md](../pmp/references/overview.md) for lifecycle context
