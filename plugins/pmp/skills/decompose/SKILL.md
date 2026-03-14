---
name: decompose
description: "Break large plans into dependency-ordered phases with entry/exit criteria. Use when the user says 'decompose plan', 'break this into phases', 'phase this plan', or has an existing plan with 5+ features that lacks a Phases section. Takes a plan file path and restructures it into phases."
---

# PMP: Decompose

Break large implementation plans into dependency-ordered phases with entry/exit criteria.

**Always ask before modifying.** Use AskQuestion. Never auto-apply phases.

Track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. Read [decompose.md](references/decompose.md) and follow it completely — it contains the phasing algorithm
3. Present proposed phases to the user for confirmation/adjustment
4. Insert the `## Phases` section into the plan file

## Key References

- Plan template (Phases section): [plan.md](../plan/assets/plan.md)
- Phasing algorithm: [decompose.md](references/decompose.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
