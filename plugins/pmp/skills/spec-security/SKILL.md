---
name: spec-security
description: Use when the user says 'threat model', 'security review', 'red team', 'attack simulation', 'STRIDE analysis', or wants focused security analysis without the full spec-review suite
---

# PMP: Spec Security Review

**Announce at start:** "Using pmp:spec-security for security analysis."

Security analysis: STRIDE threat modeling, adversarial attack simulation, and AI red team analysis (conditional).

Part of the spec-review suite — can run standalone or be dispatched by `/pmp:spec-review`.

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants
2. **Check the analysis cache FIRST** — see [analysis-cache.md](../pmp/references/analysis-cache.md)
3. **If running standalone:** Read [discovery.md](../spec-review/references/discovery.md) and execute Phase 0 (Discovery) + Phase 1 (System Reconstruction)
4. **If dispatched by orchestrator:** Skip discovery — system model already in context
5. Read [spec-security.md](references/spec-security.md) and follow it completely
6. Produce report using [spec-security-output.md](assets/spec-security-output.md)

## Key References

- Analysis methodology: [spec-security.md](references/spec-security.md)
- Report template: [spec-security-output.md](assets/spec-security-output.md)
- Shared discovery: [discovery.md](../spec-review/references/discovery.md)
