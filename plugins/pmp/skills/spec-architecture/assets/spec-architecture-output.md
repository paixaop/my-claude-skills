# Spec Architecture Review: [Project/Component Name]
<!-- Template for Architecture Quality Review output -->

## Executive Summary

**Architecture quality score:** N/10

**Top 5 critical findings:**

1. [Finding]
2. [Finding]
3. [Finding]
4. [Finding]
5. [Finding]

---

## System Model
<!-- Included for standalone mode. When dispatched by orchestrator, this is already in the main report. -->

### Components

| Component | Classification | Responsibility | Failure Modes |
|-----------|---------------|----------------|---------------|
| [name] | data plane / control plane / policy / observability / external | [responsibility] | [failure modes] |

### Request Lifecycle

[Step-by-step request flow]

### Trust Boundaries

| Boundary | From → To | Trust Level |
|----------|-----------|-------------|
| [boundary] | [source] → [destination] | trusted / untrusted |

### State Model

| State | Type | Scope | Consistency |
|-------|------|-------|-------------|
| [state] | cache / queue / session / buffer | local / shared / replicated | strong / eventual |

---

## Simplification Opportunities

| Component/Abstraction | Problem | Type | Recommendation | Effort |
|-----------------------|---------|------|----------------|--------|
| [name] | unnecessary / redundant / over-engineered / premature extensibility | remove / merge / flatten / simplify | [concrete recommendation] | Low / Medium / High |

---

## Design Philosophy Assessment

### First Principles & Best Practices

| Aspect | Assessment | Evidence |
|--------|-----------|----------|
| First principles grounding | [grounded / partially grounded / cargo-culted] | [which decisions reflect first principles vs copied patterns] |
| Best practices alignment | [aligned / partially aligned / divergent] | [which practices are followed or violated] |

### Optimization & Priority Analysis

| Question | Current State | Recommendation | Rationale |
|----------|--------------|----------------|-----------|
| Optimizing for what we shouldn't? | [what the design over-optimizes for] | [what to optimize for instead] | [why] |
| Prioritizing what we shouldn't? | [what the design over-prioritizes] | [what to prioritize instead] | [why] |

---

## Component Boundary Issues

| Issue | Components Involved | Problem Type | Recommendation |
|-------|-------------------|--------------|----------------|
| [issue] | [components] | circular dependency / unclear ownership / cross-cutting / leaky abstraction | [recommendation] |

---

## Consistency Problems

| Inconsistency | Locations | Type | Proposed Resolution |
|---------------|-----------|------|---------------------|
| [inconsistency] | [where it appears] | conflicting statements / duplicate concepts / inconsistent terminology / mismatched responsibilities | [canonical term or resolution] |

### Unified Terminology Model (if needed)

| Canonical Term | Also Called | Definition |
|---------------|------------|------------|
| [term] | [aliases found in spec] | [single definition] |

---

## Determinism Issues

| Issue | Location | Type | Risk | Recommendation |
|-------|----------|------|------|----------------|
| [issue] | [component/pipeline] | non-deterministic / order-dependent / race-condition / hidden state | Critical / High / Medium / Low | [specific fix] |

---

## Configuration Model Issues

| Issue | Setting(s) | Type | Risk | Recommendation |
|-------|-----------|------|------|----------------|
| [issue] | [config field(s)] | ambiguous / overlapping / hidden default / undefined state / precedence conflict | Critical / High / Medium / Low | [specific fix] |

---

## Invariant Violations

| Invariant | Enforced? | Violation Path | Consequence |
|-----------|-----------|----------------|-------------|
| [invariant] | yes / partially / no | [how it could be violated] | [impact] |

---

## State Machine Issues

| Issue | Type | Description | Risk |
|-------|------|-------------|------|
| [issue] | missing transition / undefined state / race condition / cancellation gap / retry loop | [description] | Critical / High / Medium / Low |

---

## What's Next

> **Next in the PMP lifecycle:** Return results to the spec review orchestrator or run sibling analyses.
>
> | Action | Command | When to use |
> |--------|---------|-------------|
> | Consolidated spec review | `pmp:spec-review` | Combine with other sub-command results into a full report |
> | Run security analysis | `pmp:spec-security` | Assess threats, STRIDE, attack simulation |
> | Run operations analysis | `pmp:spec-operations` | Assess performance, failure modes, scalability |
> | Run implementability check | `pmp:spec-implementability` | Assess coding readiness of specs |
