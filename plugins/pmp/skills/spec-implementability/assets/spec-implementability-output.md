# Spec Implementability Review: [Project/Component Name]
<!-- Template for Implementability Assessment output -->

## Overall Verdict

**Verdict:** READY / READY WITH MINOR TWEAKS / NOT READY
[one-sentence explanation]

## Risk Summary

- **Risk Level:** LOW / MEDIUM / HIGH / BLOCKER
- **Confidence in verdict:** XX%
- **Estimated total fix effort:** LOW / MEDIUM / HIGH

---

## Per-Criterion Findings

### 1. End-to-End Wiring & Connectivity

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

#### Connectivity Matrix

| Source | Target | Connection | Status |
|--------|--------|------------|--------|
| [component] | [component] | [what flows between them] | Specified / Partial / Missing |

**Dangling outputs:** [list or "none"]
**Unconnected inputs:** [list or "none"]

---

### 2. Workflow & Pipeline Completeness

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Gap | Location | Type | Severity |
|-----|----------|------|----------|
| [gap] | [spec section] | missing branch / missing error path / missing retry / missing timeout / missing concurrency rule | BLOCKER / HIGH / MEDIUM / LOW |

---

### 3. Complete Interface & Contract Coverage

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Interface | Type | Inputs Typed? | Outputs Typed? | Errors Specified? | Auth? | Idempotent? | Status |
|-----------|------|:---:|:---:|:---:|:---:|:---:|--------|
| [interface] | API / function / queue / DB / external | yes/partial/no | yes/partial/no | yes/partial/no | yes/no/N/A | yes/no/N/A | Complete / Gaps |

---

### 4. Data Models & State Management

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Entity | Schema Complete? | Relationships? | Constraints? | Migrations? | Status |
|--------|:---:|:---:|:---:|:---:|--------|
| [entity] | yes/partial/no | yes/partial/no | yes/partial/no | yes/no/N/A | Complete / Gaps |

---

### 5. Non-Functional Requirements

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Requirement | Specified? | Measurable? | Value | Status |
|-------------|:---:|:---:|-------|--------|
| P99 latency | yes/no | yes/no | [value or "not specified"] | Complete / Gap |
| Throughput | yes/no | yes/no | [value or "not specified"] | Complete / Gap |
| Availability SLO | yes/no | yes/no | [value or "not specified"] | Complete / Gap |
| [etc.] | | | | |

---

### 6. Security, Compliance & Privacy

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Control | Specified? | Implementable? | Details |
|---------|:---:|:---:|---------|
| Encryption at rest | yes/no | yes/no | [details or "not specified"] |
| Encryption in transit | yes/no | yes/no | [details or "not specified"] |
| Secret management | yes/no | yes/no | [details or "not specified"] |
| Audit logging | yes/no | yes/no | [details or "not specified"] |
| [etc.] | | | |

---

### 7. Observability, Logging & Monitoring

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Capability | Specified? | Implementable? | Details |
|------------|:---:|:---:|---------|
| Structured logging | yes/no | yes/no | [details or "not specified"] |
| Metrics | yes/no | yes/no | [details or "not specified"] |
| Distributed tracing | yes/no | yes/no | [details or "not specified"] |
| Alerting rules | yes/no | yes/no | [details or "not specified"] |
| Dashboards | yes/no | yes/no | [details or "not specified"] |
| Runbooks | yes/no | yes/no | [details or "not specified"] |

---

### 8. Testing, Quality & Acceptance

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Test Type | Requirements Defined? | Coverage Target? | Acceptance Criteria? | Status |
|-----------|:---:|:---:|:---:|--------|
| Unit tests | yes/no | yes/no | yes/no | Complete / Gap |
| Integration tests | yes/no | yes/no | yes/no | Complete / Gap |
| E2E tests | yes/no | yes/no | yes/no | Complete / Gap |
| Performance tests | yes/no | yes/no | yes/no | Complete / Gap |

---

### 9. Deployment, Operations & Configuration

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Aspect | Specified? | Details |
|--------|:---:|---------|
| Deployment method | yes/no | [details or "not specified"] |
| CI/CD pipeline | yes/no | [details or "not specified"] |
| Feature flags | yes/no | [details or "not specified"] |
| Configuration management | yes/no | [details or "not specified"] |
| Rollback strategy | yes/no | [details or "not specified"] |

---

### 10. Ambiguities, Assumptions & Contradictions

[detailed reasoning, quotes from spec]

| ID | Type | Description | Location | Severity |
|----|------|-------------|----------|----------|
| AA-NNN | ambiguity / assumption / contradiction / missing reference / undefined term / deferred decision | [description with spec quote] | [file/section] | BLOCKER / HIGH / MEDIUM / LOW |

---

### 11. Coding-Agent Readiness

[detailed component-by-component assessment]

| Component | Implementable? | Judgment Calls Required | Missing Info |
|-----------|:---:|----------------------|--------------|
| [component] | yes / mostly / no | [what a coding agent would have to guess] | [what's missing] |

---

## Gap List

| Severity | Category | Gap Description | Suggested Fix | Effort |
|----------|----------|-----------------|---------------|--------|
| BLOCKER | [criterion #] | [description] | [specific fix] | Low / Medium / High |
| HIGH | [criterion #] | [description] | [specific fix] | Low / Medium / High |
| MEDIUM | [criterion #] | [description] | [specific fix] | Low / Medium / High |
| LOW | [criterion #] | [description] | [specific fix] | Low / Medium / High |

---

## Prioritized Action Items

1. [what must be added/clarified before implementation — most critical first]
2. ...
3. ...

---

## Recommended Wording

For the top 3 critical gaps, provide exact spec language to add:

### Gap: [gap title]

**Add to:** [spec file / section]

**Suggested wording:**
> [exact text to add to the spec]

### Gap: [gap title]

**Add to:** [spec file / section]

**Suggested wording:**
> [exact text to add to the spec]

### Gap: [gap title]

**Add to:** [spec file / section]

**Suggested wording:**
> [exact text to add to the spec]
