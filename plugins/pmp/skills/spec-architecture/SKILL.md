---
name: spec-architecture
description: Use when the user says 'review architecture', 'simplicity review', 'consistency check', 'check invariants', or wants focused analysis of architectural quality without the full spec-review suite
---

# PMP: Spec Architecture Review

**Announce at start:** "Using pmp:spec-architecture for architecture quality analysis."

Architecture quality analysis: simplicity & boundaries, consistency & determinism, formal invariants, and state machine validation.

Part of the spec-review suite — can run standalone or be dispatched by `/pmp:spec-review`.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md)
3. **If running standalone:** Read [discovery.md](../spec-review/references/discovery.md) and execute Phase 0 (Discovery) + Phase 1 (System Reconstruction)
4. **If dispatched by orchestrator:** Skip discovery — system model already in context
5. Read [spec-architecture.md](references/spec-architecture.md) and follow it completely
6. Produce report using [spec-architecture-output.md](assets/spec-architecture-output.md)

## Key References

- Analysis methodology: [spec-architecture.md](references/spec-architecture.md)
- Report template: [spec-architecture-output.md](assets/spec-architecture-output.md)
- Shared discovery: [discovery.md](../spec-review/references/discovery.md)
