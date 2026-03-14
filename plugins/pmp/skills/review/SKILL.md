---
name: review
description: "Skeptical senior-engineer review of implementation plans. Use when the user has an existing plan file and says 'review plan', 'review this', 'check my plan', 'is this plan good', or presents a plan for feedback. Evaluates architecture, security, testing coverage, complexity, and completeness. Includes a security gate (STRIDE + attack trees) for plans touching auth, data, or endpoints. Produces a structured verdict with actionable findings. Not for architecture/spec review of existing systems (use pmp:spec-review)."
---

# PMP: Review

Skeptical senior-engineer review of implementation plans. Scores against a checklist covering architecture, security, testing, config, and completeness.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) ONLY for parallel file reading when the corpus is large. All analysis runs in the main controller context — do not spawn agents for analysis phases (they would re-read all files). Track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. Read [review.md](references/review.md) and follow it completely — it contains the review checklist, security gate, and verdict loop
3. The review loops until the user picks "proceed", "update", or "discuss"
4. On approval, ask: "Plan approved. Want to publish as GitHub Issues before implementation?"
   - If yes → tell the user to invoke `/pmp:github`
   - If no → tell the user to invoke `/pmp:execute`

## Key References

- Review output template: [review-output.md](assets/review-output.md)
- Security analysis: [security-analysis.md](references/security-analysis.md)
- Security report template: [security-analysis-output.md](assets/security-analysis-output.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
