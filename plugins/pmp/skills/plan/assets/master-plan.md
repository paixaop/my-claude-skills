---
status: draft
created_at: "YYYY-MM-DDTHH:MM:SSZ"
reviewed_at:
issues_created_at:
execution_started_at:
completed_at:
pr:
epic:
---

# [System] Master Implementation Plan

**Goal:** [One sentence]
**Source:** [Spec directory path]
**Spec Corpus:** [N files across M sections]
**Decomposition:** [P phases, each with its own sub-plan]
**Integration Branch:** [Detected branch name]
**CI Command:** [Detected CI command]

> **To execute this plan:** execute sub-plans in phase order using `/pmp execute`.

---

## Spec Coverage Summary

<!-- Maps spec sections to phases and sub-plan files -->

| Spec Section | Files | Phase | Sub-Plan |
|---|---|---|---|
| [01-introduction/](path/) | N files | Phase 1 | [phase-1-name](phase-1-name-plan.md) |
| [05-building-blocks/](path/) | N files | Phase 2 | [phase-2-name](phase-2-name-plan.md) |
| [08-crosscutting/](path/) | N files | Phase 3 | [phase-3-name](phase-3-name-plan.md) |

---

## Phase Dependency Graph

1. **Phase 1: [Name]** (no dependencies) → [sub-plan](phase-1-plan.md)
   - Features: N features, M tasks
   - Spec files: [list]
   - Testing: [which layers this phase needs]
2. **Phase 2: [Name]** (depends on Phase 1) → [sub-plan](phase-2-plan.md)
   - Features: N features, M tasks
   - Spec files: [list]
   - Testing: [which layers]
3. **Phase 3: [Name]** (depends on Phase 1, Phase 2) → [sub-plan](phase-3-plan.md)
   - Features: N features, M tasks
   - Spec files: [list]
   - Testing: [which layers]

---

## Spec Traceability (cross-phase)

<!-- Every spec section maps to a phase, feature, and AC. Validates complete coverage. -->

| Spec File | Section | Phase | Feature | AC | Status |
|-----------|---------|-------|---------|----|----|
| [file.md](file.md) | #section | Phase 1 | Feature 1 | AC-1.1 | Covered |
| [file2.md](file2.md) | #section | Phase 2 | Feature 4 | AC-4.2 | Covered |
| [file3.md](file3.md) | #section | — | — | — | Not covered — intentional (reason) |

---

## Test Harness Summary

**Harness location:** `specs/test-specs/`
**Master spec:** `master_spec.json`
**Coverage by layer:**

| Layer | Test Files | Total Tests | Phases |
|-------|-----------|------------|--------|
| Unit | N files | M tests | All |
| Module | N files | M tests | Phase 1, 2 |
| Integration | N files | M tests | Phase 2, 3 |
| E2E | N files | M tests | All |
| Performance | N files | M tests | Phase 3 |
| Resilience | N files | M tests | Phase 2, 3 |
| Fuzz | N files | M tests | Phase 1 |

---

## Execution Strategy

- **Parallel features per phase:** Controller dispatches independent feature orchestrators concurrently (max 3 per batch)
- **Phase boundaries:** Phase N+1 starts only after Phase N exit criteria met
- **Context management:** Each phase is a separate controller session (fresh context)
- **Sub-plan execution:** Each sub-plan follows the standard execute-loop with orchestrator agents

---

## What's Next

> | Action | Command | When to use |
> |--------|---------|-------------|
> | Review this master plan | `pmp:plan-review` | Validate coverage, phasing, dependencies |
> | Review a sub-plan | `pmp:plan-review` | Point at specific sub-plan file |
> | Execute Phase 1 | `pmp:execute` | Point at Phase 1 sub-plan file |
> | Generate test harness | `pmp:test-harness` | If not yet generated |

<!-- Include only when generated as master-plan-only (no sub-plans yet) -->

> **Reminder:** This master plan was generated without detailed sub-plans.
> To generate sub-plans and begin implementation, run `pmp:execute` pointed at this file.
> The execute loop will generate each phase's sub-plan before implementing it.
