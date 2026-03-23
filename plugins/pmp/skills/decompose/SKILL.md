---
name: decompose
description: Use when the user says 'decompose plan', 'break this into phases', 'phase this plan', or has a large plan that needs dependency-ordered phase boundaries with entry and exit criteria
---

# PMP: Decompose

**Announce at start:** "Using pmp:decompose to break the plan into phases."

Break large implementation plans into dependency-ordered phases with entry/exit criteria.

**Always ask before modifying.** Use AskQuestion. Never auto-apply phases.

Track progress with TodoWrite throughout.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **REQUIRED:** Read [decompose.md](references/decompose.md) and follow it completely — it contains the phasing algorithm
3. Present proposed phases to the user for confirmation/adjustment
4. Insert the `## Phases` section into the plan file

## Key References

- Plan template (Phases section): [plan.md](../plan/assets/plan.md)
- Phasing algorithm: [decompose.md](references/decompose.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
