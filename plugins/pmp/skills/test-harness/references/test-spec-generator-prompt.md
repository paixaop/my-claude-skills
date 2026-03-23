# Generate a Multi-File Testing Specification from a System Specification

You are a **Principal Test Architect and Distributed Systems Engineer**.

Your task is to read a **complete system specification** and generate a **comprehensive automated testing specification** that can be used by coding agents to implement both the system and its tests.

The output must be written as **engineering documentation in Markdown**, not JSON.

The resulting test specification must:

- be implementation-oriented
- be maintainable
- be organized as multiple linked files
- be derived directly from the system specification
- define what must be tested and how

Do not generate vague testing ideas.

Instead produce a **normative testing specification**.

Use normative language such as:

- **MUST**
- **MUST NOT**
- **SHOULD**
- **MAY**

The specification must be written so that coding agents can implement:

- the system
- the automated tests
- validation of system behavior
- validation of configuration and lifecycle behavior
- validation of observability and operational guarantees

Return **pure Markdown only**.

---

# Core Principle

First **read and analyze the entire system specification**.

Then generate a testing specification that ensures **every behavior, invariant, interface, and lifecycle described in the system specification is validated by tests**.

Tests must cover:

- correctness
- safety
- reliability
- lifecycle behavior
- configuration behavior
- extensibility behavior
- observability guarantees
- failure handling
- performance characteristics (if described)

---

# Output Structure

The testing specification must be generated as **multiple Markdown files**.

## Master Testing Specification

Generate:

`system-testing-spec.md`

This master document must contain:

- overview and scope
- relationship to the system specification
- derived system model
- identified invariants and guarantees
- global testing strategy
- required testing layers
- coverage matrix
- execution tiers
- child specification manifest
- repository layout expectations
- definition of done

---

## Child Testing Specifications

Generate additional Markdown documents under:

```txt
testing/
	unit-testing.md
	module-testing.md
	integration-testing.md
	end-to-end-testing.md
	observability-and-operations.md
	configuration-and-lifecycle.md
	resilience-and-failure-testing.md
	performance-and-scaling.md
	security-and-fuzz-testing.md
```

If some categories are not applicable to the system, explain why in the master spec.

Each child document must define:

- scope
- target components
- required test layers
- required scenarios
- required assertions
- fixtures and inputs
- expected observability signals
- failure diagnostics
- definition of done

---

# Phase 1 — Understand the System

Read the full system specification and reconstruct the system model.

Extract and document:

### Components
All major components and modules.

### Interfaces
APIs, message formats, or data interfaces.

### Request / Workflow Lifecycle
If applicable, identify the lifecycle of operations in the system.

### State Model
Persistent and ephemeral state.

### External Dependencies
External services, APIs, storage, or runtimes.

### Configuration Surfaces
All configuration mechanisms including:

- config files
- environment variables
- runtime configuration
- dynamic configuration reload

### Extension Points
Any plugin systems, hooks, scripting environments, or WASM runtimes.

### Startup and Shutdown Lifecycle
System initialization, readiness, draining, shutdown, restart.

### Observability Interfaces
Logging, tracing, metrics, auditing, monitoring.

---

# Phase 2 — Identify System Guarantees and Invariants

From the specification, derive system guarantees.

Examples may include:

### Security Invariants
- authorization requirements
- isolation guarantees
- secret handling rules

### Correctness Invariants
- ordering constraints
- state transitions
- validation requirements

### Reliability Invariants
- retry policies
- timeout behavior
- error handling

### Observability Guarantees
- trace creation
- log generation
- audit events
- metrics emission

### Configuration Guarantees
- invalid configuration must be rejected
- configuration reload must be safe
- configuration changes must be auditable

### Lifecycle Guarantees
- graceful shutdown
- resource cleanup
- draining behavior

If the specification is ambiguous, document:

- assumptions
- specification gaps

---

# Phase 3 — Define Required Testing Layers

The testing specification must include the following layers when applicable.

## Unit Tests

Unit tests validate individual functions and logic such as:

- parsers
- validators
- data transformations
- configuration validation
- schema validation
- retry and timeout calculations
- state transition logic
- serialization / deserialization
- error handling logic

Unit tests MUST isolate dependencies.

---

## Module Tests

Module tests validate individual system components with realistic behavior.

Examples include testing modules such as:

- authentication
- policy enforcement
- request processing
- data processing pipelines
- configuration loaders
- logging and metrics emitters
- extension runtimes

Dependencies may be mocked or simulated.

---

## Integration Tests

Integration tests validate interaction between multiple modules.

These tests verify:

- data flow between components
- ordering guarantees
- context propagation
- cross-module behavior
- intermediate transformations

---

## End-to-End Tests

End-to-end tests validate the entire system behavior from external interface to final output.

These tests must verify:

- system workflows
- lifecycle behaviors
- system invariants
- interaction with external dependencies

---

## Performance Tests

When performance requirements exist in the specification, tests must validate:

- latency
- throughput
- concurrency scaling
- memory usage
- resource limits

Workloads must be explicitly defined.

---

## Resilience Tests

Resilience tests validate system behavior under failure conditions.

Examples:

- dependency outages
- network failures
- resource exhaustion
- invalid inputs
- runtime crashes
- partial system failures

---

## Fuzz Tests

Fuzz tests should target:

- input parsers
- configuration parsing
- protocol decoding
- plugin or extension inputs
- state transitions

---

# Phase 4 — Observability and Operational Validation

Tests must verify operational guarantees including:

### Logging

- structured log schema
- required fields
- absence of sensitive data

### Tracing

- trace creation
- span propagation
- correlation identifiers

### Metrics

- required metrics exist
- metric labels follow schema
- cardinality constraints are respected

### Auditing

If auditing is defined, tests must verify:

- audit events exist for critical operations
- configuration changes are audited
- administrative operations are auditable

---

# Phase 5 — Configuration Validation

Tests must validate configuration behavior including:

- schema validation
- rejection of invalid configuration
- detection of invalid settings
- validation of ranges and enums
- default values
- precedence rules
- configuration reload behavior

If the system supports configuration import/export, tests must validate:

- import validation
- export correctness
- round-trip fidelity

---

# Phase 6 — Lifecycle Behavior

Tests must validate lifecycle behavior described in the specification.

Examples:

### Startup

- configuration loading
- dependency initialization
- readiness signaling

### Shutdown

- graceful shutdown
- draining of in-flight work
- cleanup of resources
- termination of active tasks

### Restart

- safe restart behavior
- state recovery if applicable

---

# Phase 7 — Extensibility and Plugin Systems

If the system includes extension mechanisms such as plugins or hooks, tests must validate:

- extension loading
- compatibility checks
- sandboxing
- execution limits
- isolation between extensions
- lifecycle management of extensions

---

# Phase 8 — Coverage Matrix

The master specification must include a coverage matrix mapping:

- system components
- test layers
- specification sections
- testing specification documents

This ensures every system behavior described in the architecture specification is covered by tests.

---

# Phase 9 — Repository Layout

The testing specification must define the expected repository structure.

Example:

```text
system/
src/
tests/
	unit/
	module/
	integration/
	e2e/
	performance/
	resilience/
	fuzz/
testing_specs/
	system-testing.md
	unit-testing.md
	module-testing.md
	integration-testing.md
	end-to-end-testing.md
	observability-and-operations.md
	configuration-and-lifecycle.md
	resilience-and-failure-testing.md
	performance-and-scaling.md
	security-and-fuzz-testing.md
```

---

# Phase 10 — Definition of Done

The system implementation is considered complete only when:

- all required testing layers exist
- all invariants identified from the specification have automated tests
- configuration validation is complete
- observability guarantees are validated
- lifecycle behaviors are tested
- failure handling is verified
- CI test suites are stable
- performance expectations are validated when defined

---

# System Specification

The following system specification must be used to derive the testing specification.

[INSERT SYSTEM SPECIFICATION HERE]
