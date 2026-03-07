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

## Recommended Architectural Improvements

| Priority | Improvement | Addresses | Effort |
|----------|-------------|-----------|--------|
| 1 | [improvement] | [which findings] | Low / Medium / High |
| 2 | [improvement] | [which findings] | Low / Medium / High |
| ... | | | |
