---
name: spec-implementability
description: Use when the user says 'is this implementable', 'ready to code', 'spec completeness', 'can we implement this', or wants to verify a spec is complete enough for a coding agent to implement without clarification
---

# PMP: Spec Implementability Review

**Announce at start:** "Using pmp:spec-implementability for implementation readiness check."

13-criteria production-readiness gate. Determines whether a specification is complete enough for a coding agent (AI or human) to implement complete, production-grade, observable, secure, and maintainable code without any further clarification, assumptions, design work, or rework. Criterion 13 specifically evaluates whether an agent will follow the spec to the letter without deviation or gold-plating.

Part of the spec-review suite — can run standalone or be dispatched by `/pmp:spec-review`.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md)
3. **If running standalone:** Read [discovery.md](../spec-review/references/discovery.md) and execute Phase 0 (Discovery) + Phase 1 (System Reconstruction)
4. **If dispatched by orchestrator:** Skip discovery — system model already in context
5. Read [spec-implementability.md](references/spec-implementability.md) and follow it completely
6. Produce report using [spec-implementability-output.md](assets/spec-implementability-output.md)

## Key References

- Analysis methodology: [spec-implementability.md](references/spec-implementability.md)
- Report template: [spec-implementability-output.md](assets/spec-implementability-output.md)
- Shared discovery: [discovery.md](../spec-review/references/discovery.md)
