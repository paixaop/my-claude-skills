---
name: spec-review
description: "Orchestrator for deep architecture & spec analysis — runs discovery once, then dispatches to spec-architecture, spec-security, spec-operations, and spec-implementability sub-commands. Produces a consolidated report. Use when the user says 'review specs', 'spec review', 'architecture review', 'design review', 'review documentation', 'check specs', 'full spec review', or wants a thorough analysis of an existing system's specifications. For focused analysis, invoke sub-commands directly: pmp:spec-architecture (simplicity, consistency, invariants), pmp:spec-security (threat modeling, attack simulation), pmp:spec-operations (performance, scalability, operability), pmp:spec-implementability (coding-readiness gate). Standalone workflow — produces a report, does not feed into plan generation or execution."
---

# PMP: Spec Review — Orchestrator

Full architecture & specification review. Runs discovery once, dispatches four focused sub-commands, consolidates findings into remediation, and produces a unified report.

Standalone workflow — does not feed into plan generation or execution.

## Sub-Commands

For focused analysis, invoke sub-commands directly:

| Sub-Command | Focus |
|-------------|-------|
| `/pmp:spec-architecture` | Simplicity, consistency, determinism, invariants, state machines |
| `/pmp:spec-security` | STRIDE threat modeling, attack simulation, AI red team |
| `/pmp:spec-operations` | Performance, resources, failure modes, scalability, operability |
| `/pmp:spec-implementability` | 11-criteria production-readiness gate |

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md)
3. Read [spec-review.md](references/spec-review.md) and follow it — orchestrates discovery, sub-commands, remediation, and report
4. Loops on discussion until user says "done"

## Key References

- Orchestration logic: [spec-review.md](references/spec-review.md)
- Shared discovery: [discovery.md](references/discovery.md)
- Report template: [spec-review-output.md](assets/spec-review-output.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
