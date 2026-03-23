# Implementability Assessment

You are a ruthless senior technical architect and production gatekeeper. Your job is to review the provided specification and determine whether it is 100% ready for a coding agent (AI or human) to implement complete, production-grade, observable, secure, and maintainable code without any further clarification, assumptions, design work, or rework.

> **Part of the spec-review suite.** Can run standalone (performs its own discovery) or as a sub-command of `/pmp:spec-review` (discovery already done). When standalone, first execute [discovery.md](../../spec-review/references/discovery.md).

## Standalone Mode

When invoked directly (not dispatched by orchestrator):
1. Read [discovery.md](../../spec-review/references/discovery.md) and execute Phase 0 + Phase 1
2. Proceed to the analysis criteria below
3. Produce a standalone report using [spec-implementability-output.md](../assets/spec-implementability-output.md)

When dispatched by `/pmp:spec-review`, skip directly to the analysis criteria — the system model is already in context.

## Mindset

You are a production gatekeeper. Be ruthless and thorough:
- If something is not mentioned in the spec, explicitly state "Not specified in the spec"
- Quote relevant (or missing) parts of the spec as evidence
- Do not give the benefit of the doubt — ambiguity is a gap
- Every criterion gets detailed reasoning, not just a pass/fail

## Cross-Referencing Other Sub-Commands

When dispatched by the orchestrator, findings from other sub-commands (spec-architecture, spec-security, spec-operations) may already be in context. Use them:
- **spec-architecture findings** → inform criteria 1, 2, 4, 10, 12
- **spec-security findings** → inform criterion 6
- **spec-operations findings** → inform criteria 5, 7, 9, 12

When running standalone, these cross-references are unavailable — evaluate each criterion independently.

---

### Criterion 1: End-to-End Wiring & Connectivity

Is the entire workflow, data flow, pipeline, and system fully wired? Verify:

- Every component, service, module, external system, trigger, handoff, and dependency is explicitly defined
- Sequence of operations is specified with conditions and failure modes
- No dangling components with outputs that go nowhere
- No orphan components with inputs from no defined source
- Every integration point has both sides specified

**Produce a connectivity matrix:** list each component pair that should be connected and mark whether the connection is fully specified, partially specified, or missing.

---

### Criterion 2: Workflow & Pipeline Completeness

Are all steps, branches, state transitions, error paths, retry logic, compensation flows, timeouts, and concurrency requirements specified? Verify:

- Every branching point has all branches defined (including the "else" case)
- Error paths are specified for every operation that can fail
- Retry logic includes: max attempts, backoff strategy, circuit breaker conditions
- Compensation/rollback flows for multi-step operations
- Timeout values for every blocking operation
- Concurrency requirements: what can run in parallel, what must be sequential, what needs locks

List every gap.

---

### Criterion 3: Complete Interface & Contract Coverage

Are ALL interfaces fully specified? Check every:

- **API endpoint**: HTTP method, path, request schema (types, validation, defaults, required/optional), response schema, error codes and messages, authentication/authorization requirements
- **Function/service call**: input parameters (types, validation), return type, exceptions/errors
- **Database interaction**: queries, schemas, indexes, constraints
- **Queue/event**: message schema, ordering guarantees, delivery semantics (at-least-once, exactly-once)
- **External integration**: API version, rate limits, authentication method, fallback behavior

For each interface verify: idempotency requirements, versioning strategy, backward compatibility constraints.

---

### Criterion 4: Data Models & State Management

Are all entities, schemas, relationships, constraints, transformations, serialization, and state machines completely defined? Verify:

- Every entity has a complete schema with field types, constraints, and relationships
- Foreign key relationships and referential integrity rules are defined
- Data transformations between components have explicit input/output mappings
- Serialization formats are specified at every boundary
- State machines have all states, transitions, triggers, guards, and side effects defined
- Migration strategy (if applicable): up/down migrations, data backfill, rollback plan

---

### Criterion 5: Non-Functional Requirements

Are performance, scalability, reliability, cost, and resource constraints explicitly stated and measurable? Verify:

- **Latency**: P50, P95, P99 targets per endpoint/operation
- **Throughput**: requests/second, messages/second targets
- **Concurrency**: max concurrent users/connections/operations
- **Scalability**: horizontal/vertical scaling strategy, scaling triggers
- **Reliability**: SLOs (availability, error rate), HA requirements, disaster recovery (RTO, RPO)
- **Cost**: resource budget, cost per transaction, infrastructure limits
- **Resource constraints**: memory limits, CPU limits, storage limits, connection pool sizes

Each requirement must be measurable — "fast" or "highly available" is not a spec.

---

### Criterion 6: Security, Compliance & Privacy

Are security controls fully specified as implementable requirements (not just identified risks)? Verify:

- **Encryption**: at-rest and in-transit specifications (algorithms, key management)
- **Secret management**: where secrets are stored, how they're rotated, who has access
- **Data classification**: which fields are PII, sensitive, or confidential
- **OWASP controls**: input validation, output encoding, CSRF protection, etc.
- **Audit logging**: what events are logged, what fields, retention period, tamper resistance
- **Compliance requirements**: GDPR, SOC2, HIPAA, PCI-DSS — specific controls per requirement
- **Threat mitigations**: for every identified threat, a specific implementable countermeasure

---

### Criterion 7: Observability, Logging & Monitoring

Can engineers implement the observability layer from the spec alone? Verify:

- **Logging standards**: structured format, required fields, log levels per event type
- **Metrics**: which metrics to emit, metric names, dimensions/labels, aggregation method
- **Distributed tracing**: trace ID propagation, span creation points, span attributes
- **Alerting rules**: conditions, thresholds, severity, escalation, notification channels
- **Dashboards**: which dashboards, what panels, data sources
- **Runbooks**: for each alert, a runbook with diagnostic steps and remediation

---

### Criterion 8: Testing, Quality & Acceptance

Are testing requirements explicitly defined? Verify:

- **Unit tests**: coverage expectations, what must be unit-tested
- **Integration tests**: which component interactions to test, test data requirements
- **Contract tests**: API contract validation between services
- **E2E tests**: critical user journeys that must pass, test environment requirements
- **Performance tests**: load profiles, expected results, pass/fail criteria
- **Testability hooks**: feature flags, test-mode configuration, mock/stub interfaces
- **Acceptance criteria**: per-feature, measurable, automatable

---

### Criterion 9: Deployment, Operations & Configuration

Is the deployment and operational model specified? Verify:

- **Deployment method**: container, serverless, VM — specific runtime requirements
- **CI/CD pipeline**: build steps, test gates, deployment stages, approval gates
- **Feature flags**: which features are flagged, flag management system, rollout strategy
- **Configuration management**: config sources, precedence rules, validation, hot-reload support
- **Rollback strategy**: how to rollback, what triggers rollback, data rollback implications
- **Operational runbooks**: startup procedures, shutdown procedures, maintenance windows

---

### Criterion 10: Ambiguities, Assumptions & Contradictions

Identify every instance of:

- **Ambiguous language**: "should", "may", "as appropriate", "reasonable", "typically" — any word that allows multiple interpretations
- **Implicit assumptions**: things the spec assumes without stating (e.g., "the database" without specifying which one)
- **Contradictions**: conflicting statements across sections
- **Missing cross-references**: references to external systems, docs, or standards without specifying what aspects apply
- **Undefined terms**: domain-specific terms used without definition
- **Deferred decisions**: items marked "TBD", "v2", "future" that are referenced as if they exist

---

### Criterion 11: Coding-Agent Readiness

After evaluating all above criteria, could a capable coding agent generate complete, production-grade, testable, observable, and secure code for every component without asking questions or inventing behavior?

Evaluate **component by component**:
- For each component identified in Phase 1 (System Reconstruction), assess whether the spec provides enough detail to implement it fully
- Identify specific points where a coding agent would need to make judgment calls not guided by the spec
- Identify implicit assumptions that require domain knowledge not present in the spec
- Assess whether two developers reading the same spec could produce materially different implementations

---

### Criterion 12: Unspecified References

Scan the entire spec for items mentioned in prose but never formally defined with enough detail to implement. These are "phantom specifications" — the spec acknowledges their existence but provides no concrete definition.

**Distinction from Criterion 10:** Criterion 10 asks "is this *clear*?" (ambiguities, contradictions). Criterion 12 asks "is this *defined*?" — the spec may be explicit that something exists, but never provides the concrete attributes needed to implement it.

Scan for these categories:

- **Config settings / env vars** — mentioned ("configure the timeout", "set via environment variable") but no name, type, default value, or valid range defined
- **Metrics / telemetry** — mentioned ("track latency", "emit metrics") but no metric name, dimensions, units, or thresholds defined
- **Error codes / status codes** — mentioned ("returns an error", "fails with 4xx") but no error catalog, code-to-message mapping, or retry semantics
- **Roles / permissions** — mentioned ("admin users can", "requires authorization") but no role enumeration, permission matrix, or RBAC model
- **Events / signals / hooks** — mentioned ("emits an event", "triggers a webhook") but no event name, schema, delivery guarantees, or payload format
- **Thresholds / limits / quotas** — mentioned ("rate limited", "max connections") but no concrete values, enforcement behavior, or overflow handling
- **Feature flags** — mentioned ("behind a feature flag", "gradual rollout") but no flag names, default states, or management system
- **CLI commands / flags** — mentioned ("run the migration command") but no command syntax, flag names, or usage documented
- **Status / state enumerations** — mentioned ("order status changes") but no enumeration of valid states or transition rules

For each phantom reference found:
1. **Quote** the prose that mentions it
2. **Classify** by category (from the list above)
3. **Assess severity**: BLOCKER if the item is in a critical path (cannot implement without it), HIGH if it affects correctness, MEDIUM if it affects completeness, LOW if it affects polish
4. **Specify what's missing** — list the concrete attributes that need definition

---

### Criterion 13: Agent Compliance Readiness

Will a coding agent (Claude Code, Codex, Copilot Workspace, or similar) follow this spec to the letter without deviation, gold-plating, or misinterpretation? This criterion evaluates the spec from the agent's perspective — treating it as the sole input to plan generation and code execution.

**Scan for agent failure modes:**

- **Ambiguous directives** — "should", "may", "optionally", "as appropriate", "consider" without a concrete condition. Agents interpret these inconsistently: some skip optional items, others implement all of them, others ask for clarification and stall.
- **Implicit knowledge dependencies** — "follow the standard pattern", "use the existing approach", "as documented elsewhere" without restating the pattern inline. Agents cannot reliably infer patterns from surrounding code without explicit instruction.
- **Prose-embedded requirements** — Critical requirements buried in narrative paragraphs rather than structured headings, lists, or tables. Agents scanning for headings will miss them. Every requirement must be in a scannable structure.
- **Contradictory instructions** — Same behavior defined differently in two locations. Agents processing specs linearly will follow whichever they read last, producing inconsistent implementations.
- **Missing "don't" constraints** — Spec says what TO do but not what NOT to do. Without explicit boundaries, agents gold-plate: adding caching the spec didn't ask for, adding retry logic not specified, implementing features from similar systems they've seen in training data.
- **Vague error handling** — "handle errors gracefully" or "return appropriate error" without specifying: which error types, what HTTP status, what error body, what to log. Agents will invent error handling that may not match the system's conventions.
- **Undefined sequencing** — Steps that could be executed in multiple orders. Agents may parallelize or reorder steps unless the spec explicitly states dependencies.
- **"Etc." and hand-waving** — "...and similar cases", "repeat for other endpoints", "apply the same pattern to X, Y, Z". Agents may implement for X but miss Y and Z, or interpret "similar" differently.

**For each finding:**
1. Quote the problematic text
2. Classify the failure mode (from the list above)
3. Assess severity: BLOCKER if an agent would produce incorrect code, HIGH if it would produce incomplete code, MEDIUM if it would produce non-idiomatic code
4. Provide a concrete rewrite that eliminates the ambiguity

---

## Output

Use [spec-implementability-output.md](../assets/spec-implementability-output.md) for the report structure.

Output ONLY in the structure defined by the template — no extra commentary outside the template sections.
