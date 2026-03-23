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

## Corpus Health Advisory
<!-- OPTIONAL: Include when corpus health signals exceed thresholds (2+ signals). Delete if corpus is well-organized. -->

**Recommendation:** Run `/pmp:arc42` before or after addressing review findings.

| Signal | Value | Threshold | Status |
|--------|-------|-----------|--------|
| Concern scatter | [N] files share concerns | >2 | [WARN/OK] |
| Duplicate density | [N]% | >15% | [WARN/OK] |
| Cross-reference density | [N] avg links/file | >5 | [WARN/OK] |
| Orphan content | [N]% of sections | >20% | [WARN/OK] |
| File:concern ratio | [N]:1 | >2:1 | [WARN/OK] |

**Rationale:** [1-2 sentences explaining why reorganization would improve the corpus]

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

---

## AI Red Team Attack Analysis
<!-- CONDITIONAL: Include this section ONLY when the system involves AI/LLM features. Delete entirely for non-AI systems. -->

### Red Team Executive Summary

**AI security risk score:** N/10

**Top 5 exploit paths:**

1. [Exploit path]
2. [Exploit path]
3. [Exploit path]
4. [Exploit path]
5. [Exploit path]

**Most dangerous vulnerability:** [description]

### Attack Surface Map

| Entry Point | Components Reached | Trust Boundary Crossed | Risk Level |
|-------------|-------------------|----------------------|------------|
| [entry point] | [components] | [boundary] | Critical / High / Medium / Low |

### Prompt Injection Attacks

| Attack ID | Attack Type | Goal | Expected Defense | Bypass Plausibility | Severity |
|-----------|------------|------|-----------------|---------------------|----------|
| PI-NNN | hierarchy / hidden / multi-step / RAG poisoning / tool override / jailbreak | [goal] | [what spec says should happen] | High / Medium / Low | Critical / High / Medium / Low |

**Detailed attack scenarios:**

#### [PI-NNN: Attack Name]

- **Attack Prompt:** [crafted input]
- **Attack Goal:** [objective]
- **Expected System Behavior:** [spec-defined response]
- **Possible Bypass:** [how it might circumvent defenses]
- **Severity:** Critical / High / Medium / Low
- **Likelihood:** High / Medium / Low

### Tool Execution Exploits

| Exploit ID | Vector | Target Tool/Action | Bypass Method | Severity | Likelihood |
|-----------|--------|-------------------|---------------|----------|------------|
| TE-NNN | argument injection / command chaining / SSRF / indirect trigger | [target] | [method] | Critical / High / Medium / Low | High / Medium / Low |

### Data Exfiltration Attacks

| Exfil ID | Target Data | Extraction Method | Spec Defense | Defense Sufficient? | Severity |
|----------|------------|-------------------|--------------|--------------------| ---------|
| DE-NNN | system prompt / policies / secrets / cross-tenant data | [method] | [what spec specifies] | yes / partially / no | Critical / High / Medium / Low |

### Policy Bypass Attacks

| Bypass ID | Technique | Target Policy | Evasion Method | Severity | Likelihood |
|-----------|-----------|--------------|----------------|----------|------------|
| PB-NNN | obfuscation / encoding / multi-step / token splitting / shadowing / indirect | [policy] | [how it evades] | Critical / High / Medium / Low | High / Medium / Low |

### Model Output Exploits

| Output ID | Vector | Downstream Target | Impact | Severity |
|-----------|--------|-------------------|--------|----------|
| MO-NNN | malicious links / unsafe instructions / filter bypass / recursive loops / XSS | [target system] | [impact] | Critical / High / Medium / Low |

### Resource Exhaustion Attacks

| DoS ID | Vector | Resource Targeted | Amplification Factor | Spec Defense | Severity |
|--------|--------|-------------------|---------------------|--------------|----------|
| RE-NNN | token flood / long prompt / streaming abuse / recursive tools / concurrent flood / slow client | [resource] | [N:1 ratio] | [defense or "none"] | Critical / High / Medium / Low |

### Multi-Tenant Attacks

| Tenant ID | Vector | Isolation Boundary | Breach Method | Severity | Likelihood |
|-----------|--------|--------------------|---------------|----------|------------|
| MT-NNN | prompt leakage / cache poisoning / policy bleed / context leak / ID spoofing | [boundary] | [method] | Critical / High / Medium / Low | High / Medium / Low |

### Advanced Attack Vectors

| Adv ID | Technique | Description | Novelty | Severity | Likelihood |
|--------|-----------|-------------|---------|----------|------------|
| AA-NNN | indirect injection / Unicode / CoT manipulation / context poisoning / fingerprinting / timing | [description] | High / Medium / Low | Critical / High / Medium / Low | High / Medium / Low |

### AI-Specific Security Improvements

| Priority | Improvement | Addresses | Fix Type | Effort |
|----------|------------|-----------|----------|--------|
| 1 | [improvement] | [which attack IDs] | Spec Fix / Design Fix / Implementation Guidance | Low / Medium / High |

---

## Implementability Assessment

### Overall Verdict

**Verdict:** READY / READY WITH MINOR TWEAKS / NOT READY
[one-sentence explanation]

### Risk Summary

- **Risk Level:** LOW / MEDIUM / HIGH / BLOCKER
- **Confidence in verdict:** XX%
- **Estimated total fix effort:** LOW / MEDIUM / HIGH

### Per-Criterion Findings

#### 1. End-to-End Wiring & Connectivity

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Source | Target | Connection | Status |
|--------|--------|------------|--------|
| [component] | [component] | [what flows between them] | Specified / Partial / Missing |

**Dangling outputs:** [list or "none"]
**Unconnected inputs:** [list or "none"]

#### 2. Workflow & Pipeline Completeness

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Gap | Location | Type | Severity |
|-----|----------|------|----------|
| [gap] | [spec section] | missing branch / missing error path / missing retry / missing timeout / missing concurrency rule | BLOCKER / HIGH / MEDIUM / LOW |

#### 3. Complete Interface & Contract Coverage

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Interface | Type | Inputs Typed? | Outputs Typed? | Errors Specified? | Auth? | Status |
|-----------|------|:---:|:---:|:---:|:---:|--------|
| [interface] | API / function / queue / DB / external | yes/partial/no | yes/partial/no | yes/partial/no | yes/no/N/A | Complete / Gaps |

#### 4. Data Models & State Management

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Entity | Schema Complete? | Relationships? | Constraints? | Migrations? | Status |
|--------|:---:|:---:|:---:|:---:|--------|
| [entity] | yes/partial/no | yes/partial/no | yes/partial/no | yes/no/N/A | Complete / Gaps |

#### 5. Non-Functional Requirements

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Requirement | Specified? | Measurable? | Value | Status |
|-------------|:---:|:---:|-------|--------|
| P99 latency | yes/no | yes/no | [value or "not specified"] | Complete / Gap |
| Throughput | yes/no | yes/no | [value or "not specified"] | Complete / Gap |

#### 6. Security, Compliance & Privacy

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Control | Specified? | Implementable? | Details |
|---------|:---:|:---:|---------|
| Encryption at rest | yes/no | yes/no | [details or "not specified"] |
| Secret management | yes/no | yes/no | [details or "not specified"] |

#### 7. Observability, Logging & Monitoring

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Capability | Specified? | Implementable? | Details |
|------------|:---:|:---:|---------|
| Structured logging | yes/no | yes/no | [details or "not specified"] |
| Metrics | yes/no | yes/no | [details or "not specified"] |
| Distributed tracing | yes/no | yes/no | [details or "not specified"] |

#### 8. Testing, Quality & Acceptance

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Test Type | Requirements Defined? | Coverage Target? | Acceptance Criteria? | Status |
|-----------|:---:|:---:|:---:|--------|
| Unit tests | yes/no | yes/no | yes/no | Complete / Gap |
| Integration tests | yes/no | yes/no | yes/no | Complete / Gap |
| E2E tests | yes/no | yes/no | yes/no | Complete / Gap |

#### 9. Deployment, Operations & Configuration

[detailed reasoning, quotes from spec, or "Not specified in the spec"]

| Aspect | Specified? | Details |
|--------|:---:|---------|
| Deployment method | yes/no | [details or "not specified"] |
| CI/CD pipeline | yes/no | [details or "not specified"] |

#### 10. Ambiguities, Assumptions & Contradictions

[detailed reasoning, quotes from spec]

| ID | Type | Description | Location | Severity |
|----|------|-------------|----------|----------|
| AA-NNN | ambiguity / assumption / contradiction / missing reference / undefined term / deferred decision | [description with spec quote] | [file/section] | BLOCKER / HIGH / MEDIUM / LOW |

#### 11. Coding-Agent Readiness

[detailed component-by-component assessment]

| Component | Implementable? | Judgment Calls Required | Missing Info |
|-----------|:---:|----------------------|--------------|
| [component] | yes / mostly / no | [what a coding agent would have to guess] | [what's missing] |

#### 12. Unspecified References

[detailed reasoning — list each phantom reference found with spec quote]

| ID | Category | Prose Mention | Missing Definition | Location | Severity |
|----|----------|---------------|--------------------|----------|----------|
| UR-NNN | config / metric / error code / role / event / threshold / flag / CLI / enum | [quote from spec] | [concrete attributes missing] | [file/section] | BLOCKER / HIGH / MEDIUM / LOW |

### Gap List

| Severity | Category | Gap Description | Suggested Fix | Effort |
|----------|----------|-----------------|---------------|--------|
| BLOCKER | [criterion #] | [description] | [specific fix] | Low / Medium / High |
| HIGH | [criterion #] | [description] | [specific fix] | Low / Medium / High |

### Prioritized Action Items

1. [what must be added/clarified before implementation — most critical first]
2. ...

### Recommended Wording

For the top 3 critical gaps, provide exact spec language to add:

#### Gap: [gap title]

**Add to:** [spec file / section]

**Suggested wording:**
> [exact text to add to the spec]

---

## Recommended Architectural Improvements

| Priority | Improvement | Addresses | Fix Type | Effort |
|----------|-------------|-----------|----------|--------|
| 1 | [improvement] | [which findings] | Spec Fix / Design Fix / Implementation Guidance / Simplification | Low / Medium / High |
| 2 | [improvement] | [which findings] | Spec Fix / Design Fix / Implementation Guidance / Simplification | Low / Medium / High |
| ... | | | | |

---

## Suggested Revised Architecture
<!-- OPTIONAL: Include only when major structural improvements are warranted. Delete for minor findings. -->

### Summary of Structural Changes

[1-2 paragraph description of what changes and why]

### Revised Component Topology

```
[text-form component diagram showing simplified architecture]
```

### Revised Pipeline Model

| Stage | Responsibility | Inputs | Outputs |
|-------|---------------|--------|---------|
| [stage] | [what it does] | [inputs] | [outputs] |

### Before / After Comparison

| Aspect | Current | Proposed | Rationale |
|--------|---------|----------|-----------|
| [aspect] | [current state] | [proposed state] | [why this is better] |

---

## What's Next

> **Next:** Run `pmp:discuss` to walk through findings interactively, or address issues and re-run `pmp:spec-review`.
