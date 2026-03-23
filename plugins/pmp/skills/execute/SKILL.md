---
name: execute
description: "Code-test-fix execution loop that implements plans with agent teams. Use when the user says 'execute plan', 'implement plan', 'start coding', 'build this', 'run the plan', or has an approved plan ready for implementation. Also handles 'run tests', 're-test', 'check E2E' for test-only mode. Uses a two-task model per feature: Task A (implementation + unit tests) → Task B (E2E tests). Manages batching, parallel agents, fix loops, CI gates, PR creation, and GitHub Issue updates. Not for plan creation (use pmp:plan) or review (use pmp:plan-review)."
---

# PMP: Execute

Implements plans using a two-task model: Task A (implementation + unit tests) and Task B (E2E tests) per feature. Manages agent teams, fix loops, CI gates, and PR creation.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants — especially the Context Management section
2. Read [execute-loop.md](references/execute-loop.md) and follow it completely — it contains setup, per-feature loops, batch controllers, fix loops, review, completion, Test Only Mode, and multi-session resume

## Verification Gate

Before claiming execution is complete: identify what command proves the claim, run it, read full output, check exit code, verify it confirms the claim. No shortcuts. Evidence before assertions.

## Key References

- Implementer prompt: [implementer-prompt.md](references/implementer-prompt.md)
- Spec reviewer prompt: [spec-reviewer-prompt.md](../pmp/references/spec-reviewer-prompt.md)
- Code quality reviewer: [code-quality-reviewer-prompt.md](references/code-quality-reviewer-prompt.md)
- PR body template: [pr-body.md](assets/pr-body.md)
- E2E test spec template: [e2e-test-spec.md](assets/e2e-test-spec.md)

## Shared Resources

- Testing approaches: [testing-approaches.md](../pmp/references/testing-approaches.md)
- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
