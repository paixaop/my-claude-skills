# Spec Security Review: [Project/Component Name]
<!-- Template for Security Analysis output -->

## Executive Summary

**Security risk score:** N/10

**Top 5 critical threats:**

1. [Threat]
2. [Threat]
3. [Threat]
4. [Threat]
5. [Threat]

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

## What's Next

> **Next in the PMP lifecycle:** Return results to the spec review orchestrator or run sibling analyses.
>
> | Action | Command | When to use |
> |--------|---------|-------------|
> | Consolidated spec review | `pmp:spec-review` | Combine with other sub-command results into a full report |
> | Run architecture analysis | `pmp:spec-architecture` | Assess simplicity, boundaries, consistency |
> | Run operations analysis | `pmp:spec-operations` | Assess performance, failure modes, scalability |
> | Run implementability check | `pmp:spec-implementability` | Assess coding readiness of specs |
