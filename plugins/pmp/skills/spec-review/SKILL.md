---
name: spec-review
description: "Deep architecture and specification review — a principal architect analyzing system design. Use when the user says 'review specs', 'spec review', 'architecture review', 'design review', 'review documentation', 'check specs', 'threat model', 'red team', 'find inconsistencies', 'simplify architecture', or wants a thorough analysis of an existing system's specifications. Runs 15-phase analysis: architectural simplicity, consistency, determinism, configuration validation, invariants, state machines, threat modeling, AI red team (for AI systems), performance, resource utilization, failure modes, scalability, operability, and remediation. Standalone workflow — produces a report, does not feed into plan generation or execution."
---

# PMP: Spec Review

Principal architect review of technical specifications. Reconstructs the system model, then runs a 15-phase deep analysis producing a comprehensive report.

Standalone workflow — does not feed into plan generation or execution.

Use agent teams (Task tool) ONLY for parallel file reading when the corpus is large. All analysis runs in the main controller context — do not spawn agents for analysis phases (they would re-read all files). Track progress with TodoWrite throughout.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md). Check `docs/.cache/spec-review/manifest.json`. Load cached summaries for unchanged files, only read changed/new files in full. This is mandatory, not optional.
3. Read [spec-review.md](references/spec-review.md) and follow it completely — it contains the full 15-phase analysis methodology
4. Loops on discussion until user says "done"

## Key References

- Report template: [spec-review-output.md](assets/spec-review-output.md)

## Shared Resources

- Full lifecycle overview: [overview.md](../pmp/references/overview.md)
