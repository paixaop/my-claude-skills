---
name: spec-operations
description: "Operations analysis — performance, resource utilization, failure modes, scalability, operability. Part of the spec-review suite; can run standalone. Use when the user says 'operations review', 'performance review', 'scalability review', 'failure mode analysis', 'operability review', or wants focused operational analysis without the full spec-review suite."
---

# PMP: Spec Operations Review

Operations analysis: performance & optimization, resource utilization, failure modes, scalability, and operability.

Part of the spec-review suite — can run standalone or be dispatched by `/pmp:spec-review`.

## Workflow

1. Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md)
3. **If running standalone:** Read [discovery.md](../spec-review/references/discovery.md) and execute Phase 0 (Discovery) + Phase 1 (System Reconstruction)
4. **If dispatched by orchestrator:** Skip discovery — system model already in context
5. Read [spec-operations.md](references/spec-operations.md) and follow it completely
6. Produce report using [spec-operations-output.md](assets/spec-operations-output.md)

## Key References

- Analysis methodology: [spec-operations.md](references/spec-operations.md)
- Report template: [spec-operations-output.md](assets/spec-operations-output.md)
- Shared discovery: [discovery.md](../spec-review/references/discovery.md)
