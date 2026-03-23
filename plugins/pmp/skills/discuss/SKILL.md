---
name: discuss
description: Use when the user has a review output file and says 'discuss review', 'walk through findings', 'discuss findings', or 'go through the review'
---

# PMP: Discuss

**Announce at start:** "Using pmp:discuss to walk through findings."

Structured walkthrough of review findings. Reads a review file (plan review or spec-review), extracts all findings sorted by severity, walks through every one interactively — explaining the issue, proposing solutions (biased toward simplification), and collecting decisions into a plan for execution.

**Always ask before transitioning.** Use AskQuestion. Never auto-advance.

Use agent teams (Task tool) and track progress with TodoWrite throughout.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **REQUIRED:** Read [discuss.md](references/discuss.md) and follow it completely — it contains the walkthrough algorithm, finding extraction, and plan generation
3. Loops through ALL findings (Critical, Important, and Minor) until all are addressed or user says "done"
4. If findings were marked for fixing, generates a plan and asks to execute

## Key References

- Plan template for findings: [discuss-plan.md](assets/discuss-plan.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
