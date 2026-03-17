# Spec Operations Review: [Project/Component Name]
<!-- Template for Operations Analysis output -->

## Executive Summary

**Operations readiness score:** N/10

**Top 5 operational risks:**

1. [Risk]
2. [Risk]
3. [Risk]
4. [Risk]
5. [Risk]

---

## Performance & Optimization

### Bottlenecks

| Bottleneck | Location | Impact | Latency Tier |
|------------|----------|--------|--------------|
| [bottleneck] | [component/path] | [description] | P50 / P95 / P99 / throughput |

### Optimization Recommendations

| Optimization | Target | Expected Impact | Type | Effort |
|-------------|--------|-----------------|------|--------|
| [optimization] | [component/path] | [latency/throughput/complexity reduction] | eliminate hop / remove abstraction / simplify data flow / reduce dependencies | Low / Medium / High |

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
