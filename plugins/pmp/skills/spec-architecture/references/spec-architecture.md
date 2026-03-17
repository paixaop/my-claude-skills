# Architecture Quality Analysis

Evaluates architectural simplicity, internal consistency, determinism, formal invariants, and state machine correctness.

> **Part of the spec-review suite.** Can run standalone (performs its own discovery) or as a sub-command of `/pmp:spec-review` (discovery already done). When standalone, first execute [discovery.md](../../spec-review/references/discovery.md).

## Standalone Mode

When invoked directly (not dispatched by orchestrator):
1. Read [discovery.md](../../spec-review/references/discovery.md) and execute Phase 0 + Phase 1
2. Proceed to the analysis phases below
3. Produce a standalone report using [spec-architecture-output.md](../assets/spec-architecture-output.md)

When dispatched by `/pmp:spec-review`, skip directly to the analysis phases — the system model is already in context.

---

### Phase 2: Architectural Simplicity & Boundaries

Evaluate the architecture for unnecessary complexity and unclear boundaries. The goal is to identify everything that can be simplified, consolidated, or removed.

#### Simplicity

Identify:
- unnecessary components that don't justify their existence
- redundant abstractions (multiple layers doing the same thing)
- over-engineered modules (solving problems that don't exist)
- features that can be merged or removed without loss
- accidental complexity introduced by the design itself
- premature extensibility (plugin systems, hook points, abstraction layers for hypothetical future needs)

Recommend:
- components to consolidate or eliminate
- abstractions to flatten
- simpler alternatives to complex mechanisms

#### Component Boundaries

Ensure the architecture clearly defines:
- module boundaries and responsibilities
- ownership of state
- allowed dependency directions

Flag:
- circular dependencies between components
- unclear or overlapping ownership of state or behavior
- cross-cutting responsibilities that blur module boundaries
- components that know too much about each other's internals

Recommend improvements to ensure:
- strong separation of concerns
- well-defined module contracts (inputs, outputs, invariants)
- unidirectional dependency graphs

#### Design Philosophy

Evaluate whether the architecture's foundational choices are sound:

- **First principles** — Does the architecture reflect first principles thinking, or does it cargo-cult patterns from other systems without justification?
- **Best practices** — Are the proposed solutions anchored in established best practices for this problem domain?
- **Misplaced optimization** — What is the design optimizing for that it shouldn't? (e.g., optimizing for flexibility when the system needs predictability)
- **Misplaced priorities** — What is the design prioritizing that it shouldn't? (e.g., prioritizing extensibility over operational simplicity)
- **Correct optimization targets** — What should the design optimize for instead? Provide concrete recommendations.
- **Correct priorities** — What should the design prioritize instead? Explain why the suggested priorities better serve the system's actual needs.

---

### Phase 3: Internal Consistency & Determinism

#### Consistency

Check that the specification is **internally coherent**. Identify:
- conflicting statements across documents or sections
- duplicate concepts described with different names
- inconsistent terminology (same thing called different names in different places)
- mismatched responsibilities (component described differently in different sections)

Verify consistency across:
- component naming
- pipeline stages and their ordering
- configuration structures and field names
- API contracts
- lifecycle descriptions and state transitions

If inconsistencies exist, propose a **unified terminology model** with canonical names.

#### Determinism

The system should behave **predictably and deterministically**. Identify areas where behavior may become:
- non-deterministic (different outcomes for same inputs)
- order-dependent (results change based on evaluation order)
- race-condition prone (concurrent operations with undefined ordering)
- dependent on hidden state (behavior changes based on invisible context)

Evaluate determinism of:
- request handling pipelines
- plugin/middleware execution order
- configuration evaluation and precedence
- policy evaluation chains
- caching layers and cache invalidation

Recommend deterministic rules:
- explicit, documented execution order for all pipeline stages
- immutable inputs within processing stages
- stateless modules where possible
- deterministic pipeline stages with well-defined input/output contracts

#### Configuration Model

Analyze the configuration design. Check for:
- ambiguous settings (unclear what a value means or does)
- overlapping configuration fields (multiple ways to control the same behavior)
- hidden defaults (behavior changes when a field is omitted, but the default isn't documented)
- configuration states that create undefined behavior (contradictory settings)
- configuration precedence ambiguity (file vs env var vs flag vs API — which wins?)

Recommend:
- explicit defaults for every configuration field
- schema validation with clear error messages
- configuration linting rules
- deterministic precedence rules (documented, tested)
- elimination of configuration combinations that produce undefined states

---

### Phase 4: Formal Invariants

Identify **critical invariants** the system must maintain. Examples:
- policies must always execute before upstream requests
- logs must record every request decision
- tokens must never be forwarded unvalidated
- upstream responses must not bypass policy filters

For each invariant determine:
- whether the spec enforces it
- how it could be violated
- consequences of violation

---

### Phase 5: State Machine Validation

Model the request lifecycle as a **state machine** with states like:

`request_received → authenticated → policy_checked → upstream_called → response_streaming → completed`

Check for:
- missing transitions
- undefined states
- race conditions
- cancellation paths
- retry loops

Identify possible **invalid states** the system could enter.
