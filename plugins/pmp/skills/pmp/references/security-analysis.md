# Plan Security Analysis

Threat modeling and design-level vulnerability assessment for development plans. Operates on architecture descriptions, not code. Produces threat models, findings, and spec change recommendations.

**Mindset:** You are a penetration tester reviewing blueprints. Find every way to break this system before it's built.

## When to Use

- During Plan Review mode (mandatory for plans touching auth, data flows, new endpoints, or secrets)
- Before implementing any development plan
- During architecture design review

## Extract from the Plan

Read the plan and extract these six elements. If ANY is missing, flag the gap as a finding:

1. **Entry points** — API endpoints, webhooks, event handlers, scheduled jobs
2. **Trust boundaries** — where data crosses between components, users, external services, databases
3. **Crown jewels** — credentials, PII, payment flows, automated actions, anything an attacker targets
4. **Auth model** — authentication strategy, session management, roles/permissions, token strategy
5. **External integrations** — third-party APIs, service dependencies, data exchanged
6. **Data flows** — end-to-end traces of critical data from entry to storage/action, noting validation points

## Phase 1: STRIDE per Trust Boundary

For EACH trust boundary, analyze all six categories:

| Category | Question |
|----------|----------|
| **Spoofing** | Can an attacker impersonate a legitimate entity? (forged tokens, spoofed origins, replayed credentials) |
| **Tampering** | Can data crossing this boundary be modified? (MITM, parameter manipulation, unsigned payloads) |
| **Repudiation** | Can actions occur without audit trail? (missing logging, mutable logs, unsigned actions) |
| **Info Disclosure** | Can sensitive data leak? (verbose errors, timing side-channels, unencrypted transit) |
| **Denial of Service** | Can this boundary be overwhelmed? (missing rate limits, resource exhaustion, queue flooding) |
| **Elevation of Privilege** | Can access controls be bypassed? (privilege escalation, role confusion, token scope abuse) |

For each applicable threat:
- Describe the concrete attack scenario grounded in the planned architecture
- Reference specific components, endpoints, or data flows from the spec
- Rate as High/Medium/Low based on architectural evidence

## Phase 2: Attack Trees per Crown Jewel

For EACH crown jewel:

```
Root: "Compromise [Crown Jewel Name]"
├── Path A: [Attack vector]
│   ├── Step 1: [Technique] — Difficulty: H/M/L, Detectability: H/M/L
│   ├── Step 2: [Next step]
│   └── Prerequisites: [What attacker needs]
├── Path B: [Alternative vector]
│   └── ...
└── Path C: [Third vector]
    └── ...
```

- Every leaf references a concrete component or endpoint from the spec
- Include multi-step chains — real attacks combine weaknesses
- Prioritize by feasibility, not theoretical possibility
- Mark paths bypassing multiple controls as "high priority"

## Phase 3: Business Logic Checklist

### Race Conditions & Atomicity

For each state-changing operation in the plan:
- Check-then-act patterns: are they specified as atomic?
- Counters, quotas, rate limits: is atomicity specified?
- Concurrent requests hitting the same state: double-spend, double-submit, double-execute?
- Token refresh and nonce consumption: atomic design?

### Pipeline & Decision Manipulation

For each data processing pipeline or automated decision:
- Can crafted input manipulate extraction, matching, or decisions?
- If AI/ML involved: prompt injection? AI output bypassing validation? Schema validation on responses?
- Can processing results redirect output actions? (reply-to hijacking, recipient manipulation)
- Are pipeline stages idempotent? What happens on partial failure or retry?

### Denial of Service & Cost Amplification

For each endpoint and integration:
- Unauthenticated endpoints triggering N downstream operations: amplification factor?
- Paid resources (AI APIs, email, SMS): budget-gated BEFORE the call?
- Backpressure on task/job queues: maximum depth?
- Single request causing unbounded DB queries or writes?
- Cost ratio: attacker request vs. defender operations?

## Phase 4: Risk Matrix

| Threat ID | Description | Likelihood (1-5) | Impact (1-5) | Risk Score | Crown Jewels |
|-----------|-------------|-------------------|--------------|------------|--------------|
| T-001 | ... | ... | ... | L x I | ... |

**Likelihood** (1=Unlikely, 5=Almost Certain): attack complexity, required access level, tool availability, detectability.

**Impact** (1=Negligible, 5=Critical): data exposure scope, data/behavior modification ability, disruption duration, business consequences.

## Output Format

Use [assets/security-analysis-output.md](../assets/security-analysis-output.md) for the report structure.

Save the report to the reviews directory defined in [config.md](../config.md) File Paths, using the security analysis filename pattern (`YYYY-MM-DD-<name>-security-analysis.md`). Create the directory if it doesn't exist. The `<name>` slug should be a short, kebab-case identifier for the plan or component under analysis (e.g., `2025-03-11-auth-api-security-analysis.md`).

## Scope Boundary

This analysis covers plan-level threats only. It does NOT cover:
- Dependency/supply-chain audit
- Infrastructure/configuration review
- Code-level vulnerability scanning
- Exploit development or proof-of-concept code
