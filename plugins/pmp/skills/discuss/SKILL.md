---
name: discuss
description: "Structured walkthrough of review findings — iterates through issues from most critical to least, explains each one, proposes solutions, and asks the user what to do. Works with both plan review and spec-review output files. Collects 'Fix' decisions into a task list and hands off to execute. Use when the user says 'discuss review', 'walk through findings', 'discuss findings', 'go through the review', or provides a review file to discuss."
---

# PMP: Discuss

Structured walkthrough of review findings. Reads a review file (plan review or spec-review), extracts findings sorted by severity, walks through each one interactively, and collects fix decisions into a plan for execution.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. Read [discuss.md](references/discuss.md) and follow it completely — it contains the walkthrough algorithm, finding extraction, and plan generation
3. Loops through findings until all are addressed or user says "done"
4. If fixes were collected, generates a plan and asks to execute

## Key References

- Plan template for fixes: [discuss-plan.md](assets/discuss-plan.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
