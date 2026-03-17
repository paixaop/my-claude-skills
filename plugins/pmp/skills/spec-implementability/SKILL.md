---
name: spec-implementability
description: "Implementability assessment — 11-criteria production-readiness gate. Determines whether a spec is complete enough for a coding agent to generate production-grade code without clarification. Part of the spec-review suite; can run standalone. Use when the user says 'implementability review', 'is this implementable', 'ready to code', 'spec completeness', 'can we implement this', 'coding readiness', or wants to verify a spec is implementation-ready."
---

# PMP: Spec Implementability Review

11-criteria production-readiness gate. Determines whether a specification is complete enough for a coding agent (AI or human) to implement complete, production-grade, observable, secure, and maintainable code without any further clarification, assumptions, design work, or rework.

Part of the spec-review suite — can run standalone or be dispatched by `/pmp:spec-review`.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md)
3. **If running standalone:** Read [discovery.md](../spec-review/references/discovery.md) and execute Phase 0 (Discovery) + Phase 1 (System Reconstruction)
4. **If dispatched by orchestrator:** Skip discovery — system model already in context
5. Read [spec-implementability.md](references/spec-implementability.md) and follow it completely
6. Produce report using [spec-implementability-output.md](assets/spec-implementability-output.md)

## Key References

- Analysis methodology: [spec-implementability.md](references/spec-implementability.md)
- Report template: [spec-implementability-output.md](assets/spec-implementability-output.md)
- Shared discovery: [discovery.md](../spec-review/references/discovery.md)
