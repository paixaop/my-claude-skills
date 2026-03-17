# Decompose Spec Review into Orchestrator + Sub-Commands

## Context

The spec-review skill is a 750-line monolith with 15 phases covering architecture, security, performance, operability, and remediation. Adding an implementability assessment phase would push it past 1000 lines. The skill needs to be decomposed into focused sub-commands that can run independently or be orchestrated together.

## Architecture

`/pmp:spec-review` becomes a thin orchestrator. Four focused sub-commands handle the analysis. Each can be invoked standalone or dispatched by the orchestrator.

```
/pmp:spec-review (orchestrator)
  ├── Discovery + System Reconstruction (Phase 0-1) — runs once
  ├── dispatches to:
  │   ├── /pmp:spec-architecture    (Phases 2-5)
  │   ├── /pmp:spec-security        (Phases 6-7, 13)
  │   ├── /pmp:spec-operations      (Phases 8-12)
  │   └── /pmp:spec-implementability (new — 11 criteria)
  ├── Remediation (consolidates all findings)
  └── Merged report → docs/reviews/
```

### Key Design Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Cache | Shared `docs/.cache/spec-review/` | Single read pass, all sub-commands benefit |
| Discovery | Centralized in orchestrator | Avoids redundant file reads; standalone sub-commands run their own discovery |
| Report | Single merged file | Discuss skill unchanged, user sees one report |
| Severity inference | Each sub-command assigns severity explicitly | Discuss doesn't need to infer |
| Prior annotations | Loaded by discovery, passed to sub-commands | No re-scanning |

### Standalone Mode

When a sub-command runs directly (e.g., `/pmp:spec-architecture`), it:
1. Runs Discovery + System Reconstruction itself (referencing shared discovery logic)
2. Runs only its analysis phases
3. Produces a partial report (just its sections)

When dispatched by the orchestrator, it skips discovery (already done) and contributes its sections to the merged report.

---

## File Structure

### New directories and files

```
plugins/pmp/skills/
  spec-review/                          # EXISTING — becomes orchestrator
    SKILL.md                            # UPDATE — router description
    references/
      spec-review.md                    # REWRITE — orchestrator logic only
      discovery.md                      # NEW — shared Phase 0-1 + context mgmt
    assets/
      spec-review-output.md            # UPDATE — consolidated template (keeps all sections)

  spec-architecture/                    # NEW
    SKILL.md
    references/spec-architecture.md    # Phases 2-5
    assets/spec-architecture-output.md

  spec-security/                        # NEW
    SKILL.md
    references/spec-security.md        # Phases 6-7, 13
    assets/spec-security-output.md

  spec-operations/                      # NEW
    SKILL.md
    references/spec-operations.md      # Phases 8-12
    assets/spec-operations-output.md

  spec-implementability/                # NEW
    SKILL.md
    references/spec-implementability.md # 11 criteria
    assets/spec-implementability-output.md
```

---

## Detailed Changes

### 1. `spec-review/references/discovery.md` (NEW)

Shared reference for Discovery + System Reconstruction. Extracted from current spec-review.md:

- **Phase 0: Discovery** — corpus mapping, document classification, prior review annotations, analysis cache check, file reading, corpus health assessment
- **Phase 1: System Reconstruction** — components, request lifecycle, trust boundaries, state model
- **Context Management** — rules about agent usage, batched phases, large corpus fallback
- **Using Prior Review Annotations** — the matching logic for acknowledged/skipped findings

All sub-commands reference this file when running standalone. The orchestrator executes it once and passes results.

### 2. `spec-review/references/spec-review.md` (REWRITE)

Becomes the orchestrator. Contains:

- **Mindset** section (preserved from current)
- **Inputs** section (preserved)
- **Corpus Size** section (preserved)
- **Review Process** — updated state diagram:
  ```
  Discovery → SystemReconstruction → SubCommandDispatch → Remediation → Report
  ```
- **Orchestration logic**:
  1. Read [discovery.md](discovery.md), execute Phase 0-1
  2. Dispatch sub-commands sequentially (each reads its own reference):
     - spec-architecture → Phases 2-5
     - spec-security → Phases 6-7 (+13 if AI detected)
     - spec-operations → Phases 8-12
     - spec-implementability → 11 criteria
  3. Consolidate all findings
  4. Run **Remediation** (current Phase 14) — cross-references all findings
  5. Assemble merged report from [spec-review-output.md](../assets/spec-review-output.md)
- **After Report** loop (preserved — discuss/done)
- **Constraints** (preserved)

### 3. `spec-architecture/references/spec-architecture.md` (NEW)

Extracted from current spec-review.md Phases 2-5:

- **Phase 2: Architectural Simplicity & Boundaries** — simplicity, component boundaries, design philosophy
- **Phase 3: Internal Consistency & Determinism** — consistency, determinism, configuration model
- **Phase 4: Formal Invariants** — critical invariants identification
- **Phase 5: State Machine Validation** — state machine modeling

Standalone preamble: "When invoked directly, first read [discovery.md](../../spec-review/references/discovery.md) and run Phases 0-1. When dispatched by orchestrator, skip discovery."

### 4. `spec-security/references/spec-security.md` (NEW)

Extracted from current spec-review.md Phases 6-7 and 13:

- **Phase 6: Threat Modeling** — STRIDE analysis
- **Phase 7: Attack Simulation** — infrastructure + AI/agentic attack scenarios
- **Phase 13: AI Red Team Attack Analysis** (conditional) — all 9 sub-sections (attack surface mapping through advanced attacks)

### 5. `spec-operations/references/spec-operations.md` (NEW)

Extracted from current spec-review.md Phases 8-12:

- **Phase 8: Performance & Architectural Optimization**
- **Phase 9: Resource Utilization**
- **Phase 10: Failure Mode Analysis**
- **Phase 11: Scalability Analysis**
- **Phase 12: Operability**

### 6. `spec-implementability/references/spec-implementability.md` (NEW)

The new 11-criteria implementability assessment. Evaluates whether the spec is complete enough for a coding agent to produce production-grade code:

1. **End-to-End Wiring & Connectivity** — every component, service, module, external system, trigger, handoff, and dependency explicitly defined with sequence, conditions, and failure modes. Produce connectivity matrix.
2. **Workflow & Pipeline Completeness** — all steps, branches, state transitions, error paths, retry logic, compensation flows, timeouts, concurrency requirements. Cross-references Phase 5 state machines. List every gap.
3. **Complete Interface & Contract Coverage** — every API, function, service call, DB interaction, queue, event, external integration has: precise input/output schemas (types, validation, defaults, required/optional), error codes/messages/recovery, authz, rate limits, idempotency, versioning, backward compatibility.
4. **Data Models & State Management** — all entities, schemas, relationships, constraints, transformations, serialization, state machines completely defined, including migration strategy.
5. **Non-Functional Requirements** — performance (latency, throughput, concurrency), scalability, reliability (SLOs, HA, DR), cost, resource constraints. Must be explicitly stated and measurable. Cross-references Phases 8-11: are findings specified precisely enough to implement and test?
6. **Security, Compliance & Privacy** — encryption, secret management, data classification, OWASP controls, audit logging, compliance requirements, threat mitigations. Cross-references Phases 6-7/13: are mitigations specified as implementable requirements, not just identified risks?
7. **Observability, Logging & Monitoring** — logging standards, metrics, distributed tracing, alerting rules, dashboards, runbooks. Cross-references Phase 12: can an engineer implement the observability layer from the spec alone?
8. **Testing, Quality & Acceptance** — unit, integration, contract, E2E, performance test requirements, testability hooks, acceptance criteria.
9. **Deployment, Operations & Configuration** — deployment method, CI/CD pipeline, feature flags, configuration management, rollback strategy, operational runbooks.
10. **Ambiguities, Assumptions & Contradictions** — ambiguous language, implicit assumptions, contradictions between sections, missing cross-references. Cross-references Phase 3 consistency findings.
11. **Coding-Agent Readiness** — holistic component-by-component verdict: could a coding agent generate complete, production-grade, testable, observable, and secure code without asking questions or inventing behavior?

### 7. Output Templates

**spec-review-output.md** (UPDATE) — add the Implementability Assessment section before Recommended Architectural Improvements. Keep all existing sections. The implementability section uses this exact structure:

```markdown
## Implementability Assessment

### Overall Verdict
**Verdict:** READY / READY WITH MINOR TWEAKS / NOT READY
[one-sentence explanation]

### Risk Summary
- **Risk Level:** LOW / MEDIUM / HIGH / BLOCKER
- **Confidence:** XX%
- **Estimated fix effort:** LOW / MEDIUM / HIGH

### Per-Criterion Findings

#### 1. End-to-End Wiring & Connectivity
[detailed reasoning, quotes from spec, or "Not specified in the spec"]

[... criteria 2-11 ...]

### Gap List

| Severity | Category | Gap Description | Suggested Fix | Effort |
|----------|----------|-----------------|---------------|--------|
| BLOCKER  | ...      | ...             | ...           | ...    |

### Prioritized Action Items
1. [what must be added/clarified before implementation]
2. ...

### Recommended Wording
[For top 3 critical gaps: exact spec language to add]
```

**spec-architecture-output.md** (NEW) — extracted sections: System Model, Simplification Opportunities, Design Philosophy Assessment, Component Boundary Issues, Consistency Problems, Determinism Issues, Configuration Model Issues, Invariant Violations, State Machine Issues.

**spec-security-output.md** (NEW) — extracted sections: Threat Model Findings, Attack Simulation Results, AI Red Team Attack Analysis (conditional).

**spec-operations-output.md** (NEW) — extracted sections: Performance & Optimization, Resource Utilization Risks, Failure Mode Weaknesses, Scalability Risks, Observability Gaps, Maintainability Problems.

**spec-implementability-output.md** (NEW) — the implementability assessment section above (used when running standalone).

### 8. SKILL.md Files

**spec-review/SKILL.md** (UPDATE):
- Description: "Orchestrator for deep architecture & spec analysis — runs discovery once, then dispatches to spec-architecture, spec-security, spec-operations, and spec-implementability sub-commands. Produces a consolidated report. For focused analysis, invoke sub-commands directly."
- Trigger words: same as current ("review specs", "spec review", "architecture review", etc.)

**spec-architecture/SKILL.md** (NEW):
- Description: "Architecture quality analysis — simplicity, consistency, determinism, invariants, state machines. Part of spec-review suite; can run standalone."
- Trigger: "review architecture", "simplicity review", "consistency check"

**spec-security/SKILL.md** (NEW):
- Description: "Security analysis — STRIDE threat modeling, attack simulation, AI red team. Part of spec-review suite; can run standalone."
- Trigger: "threat model", "security review", "red team", "attack simulation"

**spec-operations/SKILL.md** (NEW):
- Description: "Operations analysis — performance, resource utilization, failure modes, scalability, operability. Part of spec-review suite; can run standalone."
- Trigger: "operations review", "performance review", "scalability review"

**spec-implementability/SKILL.md** (NEW):
- Description: "Implementability assessment — 11-criteria production-readiness gate. Determines whether a spec is complete enough for a coding agent to generate production-grade code. Part of spec-review suite; can run standalone."
- Trigger: "implementability review", "is this implementable", "ready to code", "spec completeness"

### 9. Router Updates

**pmp/SKILL.md** — add new sub-commands to the table:

| Sub-Skill | When to Use |
|-----------|-------------|
| `/pmp:spec-review` | Full architecture & spec analysis (orchestrates all sub-reviews) |
| `/pmp:spec-architecture` | Architecture quality: simplicity, consistency, invariants, state machines |
| `/pmp:spec-security` | Security: STRIDE threat modeling, attack simulation, AI red team |
| `/pmp:spec-operations` | Operations: performance, resources, failure modes, scalability, operability |
| `/pmp:spec-implementability` | Implementability: 11-criteria production-readiness gate |

**pmp/SKILL.md** routing table — add rows:

| Signal | Stage | Reference |
|--------|-------|-----------|
| "review specs", "architecture review", "full spec review" | Spec Review (full) | spec-review.md |
| "review architecture", "simplicity", "consistency" | Spec Architecture | spec-architecture.md |
| "threat model", "security review", "red team" | Spec Security | spec-security.md |
| "operations review", "performance", "scalability" | Spec Operations | spec-operations.md |
| "implementability", "ready to code", "spec completeness" | Spec Implementability | spec-implementability.md |

**pmp/references/overview.md** — update the file tree and workflow 5 description.

### 10. Discuss Skill Compatibility

**No changes needed to discuss.md.** The orchestrator produces a single merged report in the same format as today. The discuss skill reads one report file and extracts findings from all tables — this stays the same. The only addition is new tables from the implementability section, which discuss handles automatically (it scans all tables with severity columns).

---

## Verification

1. `grep -r "15-phase" plugins/pmp/` returns zero matches
2. Invoke `/pmp:spec-review` on an existing spec → runs discovery once, dispatches all 4 sub-commands, produces merged report
3. Invoke `/pmp:spec-implementability` standalone on an existing spec → runs its own discovery, produces partial report with just the implementability section
4. Invoke `/pmp:discuss` on a merged report → extracts findings from all sections including implementability
5. Each sub-command's standalone output is valid and self-contained
