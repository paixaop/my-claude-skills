# Architecture & Spec Review: [Project/Component Name]
<!-- Template for Architecture & Spec Review output — used by references/spec-review.md -->

## Executive Summary

**Architecture quality score:** N/10

**Top 5 critical risks:**

1. [Risk]
2. [Risk]
3. [Risk]
4. [Risk]
5. [Risk]

**Top 5 recommended fixes:**

1. [Fix]
2. [Fix]
3. [Fix]
4. [Fix]
5. [Fix]

---

## System Model

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

## Threat Model Findings

| Threat ID | Category | Threat | Attack Path | Impact | Severity | Mitigation |
|-----------|----------|--------|-------------|--------|----------|------------|
| T-NNN | Spoofing / Tampering / Repudiation / Info Disclosure / DoS / Elevation / AI-specific | [threat] | [attack path] | [impact] | Critical / High / Medium / Low | [mitigation] |

---

## Attack Simulation Results

### [Scenario Name]

- **Attack:** [description]
- **System behavior:** [what happens]
- **Weaknesses found:** [weaknesses]
- **Severity:** Critical / High / Medium / Low

---

## Performance Bottlenecks

| Bottleneck | Location | Impact | Latency Tier |
|------------|----------|--------|--------------|
| [bottleneck] | [component/path] | [description] | P50 / P95 / P99 / throughput |

---

## Resource Utilization Risks

| Risk | Resource | Bounded? | Backpressure? | Impact |
|------|----------|----------|---------------|--------|
| [risk] | CPU / memory / threads / connections / FDs / queues | yes / no | yes / no | [impact] |

---

## Failure Mode Weaknesses

| Failure Scenario | System Behavior | Fail Open/Closed? | Cascading? | Severity |
|------------------|-----------------|--------------------|------------|----------|
| [scenario] | [behavior] | open / closed / undefined | yes / no | Critical / High / Medium / Low |

---

## Observability Gaps

| Gap | What's Missing | Impact on Operations |
|-----|---------------|---------------------|
| [gap] | [missing capability] | [operational impact] |

---

## Scalability Risks

| Risk | Bottleneck Type | Scale Trigger | Impact |
|------|-----------------|---------------|--------|
| [risk] | centralized state / sync calls / shared cache / global lock | [at what scale] | [impact] |

---

## Maintainability Problems

| Problem | Area | Description | Recommendation |
|---------|------|-------------|----------------|
| [problem] | config / rollout / compatibility / schema evolution / upgrades | [description] | [recommendation] |

---

## Recommended Architectural Improvements

| Priority | Improvement | Addresses | Effort |
|----------|-------------|-----------|--------|
| 1 | [improvement] | [which findings] | Low / Medium / High |
| 2 | [improvement] | [which findings] | Low / Medium / High |
| ... | | | |
