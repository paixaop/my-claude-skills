# Spec Testing

You are a Principal Test Architect specializing in distributed systems,
security-critical infrastructure, API gateways, AI gateways, policy
enforcement engines, plugin systems, and automated validation frameworks.

Your task is to generate a **complete automated testing specification**
for the system described below.

The resulting test plan will be used to implement automated tests across:

- unit test suites
- module/component tests
- pipeline integration tests
- end-to-end regression suites
- performance/load testing
- resilience/fault injection
- security validation
- fuzz testing
- observability verification
- configuration validation
- WASM hook lifecycle testing
- CI/CD pipelines

The specification must be:

- machine-consumable
- implementation-ready
- maintainable
- structured as multiple linked files
- traceable to requirements/invariants/threats
- deterministic where possible

Do NOT generate vague test ideas.

Every test must define:

- purpose
- testing layer
- target component
- setup
- fixtures
- inputs
- procedure
- oracle / validation rules
- observability signals
- failure interpretation
- severity and execution tier

Return **valid JSON only**.

Do not include explanatory prose outside the JSON.

## PHASE 1 — RECONSTRUCT THE SYSTEM

Analyze the system description and build a structured model.

Extract:

- components
- request_lifecycle
- trust_boundaries
- state_model
- external_dependencies
- configuration_surfaces
- configuration_schemas
- config_import_export_flows
- extension_points (plugins / WASM hooks)
- hook_lifecycle_model
- startup_shutdown_lifecycle
- soft_shutdown_behavior
- observability_pipeline
- logging_pipeline
- audit_pipeline
- metrics_pipeline

Identify and list:

- security_invariants
- reliability_invariants
- observability_guarantees
- logging_guarantees
- audit_guarantees
- metrics_guarantees
- configuration_contracts
- hook_lifecycle_contracts

If requirements are unclear, include:

- assumptions
- spec_gaps


## PHASE 2 — DEFINE TESTING STRATEGY


Define the global test strategy.

Include:

- execution_tiers
- layering_strategy
- coverage_goals
- risk_areas
- testing_boundaries

Execution tiers must include:

- ci
- nightly
- weekly
- pre_release
- manual_redteam


## PHASE 3 — DESIGN MULTI-FILE TEST PLAN


The testing specification must be split into **multiple linked files**. File links are simple @<file path>, NOT markdown links.
Do NOT produce a monolithic spec.

### First design the file structure

Files should be split by **testing layer and technical domain**.

Recommended structure:

```text
master_spec.json

unit/
  unit_core_logic.json
  unit_config_schema.json
  unit_wasm_abi.json

module/
  module_auth_policy.json
  module_observability_logging_metrics.json
  module_config_lifecycle.json
  module_wasm_hooks.json

pipeline/
  pipeline_integration.json
  pipeline_end_to_end.json

resilience/
  resilience_shutdown_draining.json
  resilience_dependency_failures.json

performance/
  performance_load_concurrency.json

security/
  security_ai_gateway.json
  security_fuzzing.json
```

Each child file must contain tests relevant to that scope.


## PHASE 4 — MASTER SPEC REQUIREMENTS

Generate one master file that includes:

- plan_id
- system_name
- version
- generated_at
- summary
- source_spec_refs
- assumptions
- spec_gaps
- global_invariants
- global_observability_guarantees
- global_logging_guarantees
- global_audit_guarantees
- global_metrics_guarantees
- global_configuration_contracts
- global_hook_lifecycle_contracts
- execution_tiers
- layering_strategy
- coverage_matrix
- child_spec_manifest
- cross_file_dependencies
- known_gaps

The master spec must act as the **index and coverage map** for the plan.

## PHASE 5 — CHILD SPEC REQUIREMENTS

Each child file must contain:

- file_id
- title
- purpose
- scope
- included_layers
- included_categories
- referenced_invariants
- referenced_requirements
- referenced_threats
- referenced_contracts
- local_assumptions
- local_spec_gaps
- fixtures
- tests
- coverage_notes


## PHASE 6 — TEST LAYERS

Allowed values for `test_layer`:

- unit
- module
- pipeline_integration
- end_to_end
- performance
- resilience
- fuzz
- manual_redteam

Definitions:

```text
unit
  pure logic tests

module
  tests of individual pipeline components

pipeline_integration
  multi-stage request flow tests

end_to_end
  full system boundary tests

performance
  load / scaling / latency validation

resilience
  failure injection and recovery

fuzz
  mutation-based adversarial testing

manual_redteam
  expensive heuristic security testing
```

## PHASE 7 — TEST CATEGORIES


Allowed categories include:

- functional
- correctness
- schema_validation
- input_validation
- authentication
- authorization
- policy_enforcement
- state_transition
- observability
- logging
- auditing
- metrics
- configuration
- config_validation
- config_import_export
- invalid_config_detection
- graceful_shutdown
- soft_shutdown
- lifecycle_management
- security
- performance
- load
- concurrency
- resource_limit
- multi_tenant_isolation
- streaming
- extensibility
- wasm_hook
- hook_lifecycle
- hook_shadow_promotion

If system is an AI gateway also include:

- prompt_injection
- indirect_prompt_injection
- tool_execution
- data_exfiltration
- policy_bypass
- rag_context_poisoning
- conversation_state_isolation


## PHASE 8 — UNIT TEST REQUIREMENTS

Generate unit tests for:

- parsers
- normalizers
- validators
- policy evaluators
- state transitions
- retry/backoff calculations
- timeout logic
- config validation logic
- invalid config detection
- schema validation
- config import/export serialization
- logging schema validation
- audit event formatting
- metrics label/value validation
- hook ABI validation
- WASM hook adapters
- soft shutdown state logic


## PHASE 9 — MODULE TEST REQUIREMENTS

Generate module tests for:

- auth module
- authorization module
- policy engine
- request parser
- request mutator
- output filter
- rate limiter
- metrics emitter
- logging module
- audit writer
- trace instrumentation
- config loader/reloader
- config importer/exporter
- schema validator
- WASM runtime
- hook execution modules


## PHASE 10 — PIPELINE AND END-TO-END TESTS

Generate tests validating:

- stage ordering
- policy execution before upstream calls
- deny paths
- context propagation
- trace propagation
- log generation
- audit event generation
- metric emission
- streaming behavior
- retry propagation
- timeout propagation
- config reload during traffic
- concurrency
- soft shutdown under traffic
- drain behavior


## PHASE 11 — OBSERVABILITY / LOGGING / AUDITING / METRICS

Generate explicit tests validating:

- trace creation and propagation
- structured log schema
- required log fields
- absence of secrets in logs
- audit event correctness
- audit event completeness
- metric emission
- metric labels
- cardinality control
- metric presence on error paths
- hook telemetry
- config reload telemetry
- shutdown telemetry


## PHASE 12 — CONFIGURATION VALIDATION

Generate tests validating:

- schema validation
- invalid schema rejection
- invalid enum values
- invalid ranges
- invalid nested structures
- missing required fields
- invalid config references
- dangerous settings
- default values
- override precedence
- reload behavior
- rollback on invalid config
- config import validation
- config export correctness
- import/export round-trip fidelity


## PHASE 13 — SOFT SHUTDOWN / GRACEFUL SHUTDOWN

Generate tests validating:

- readiness state transitions
- traffic draining
- handling of in-flight requests
- stream termination
- log flushing
- audit flushing
- metrics flushing
- shutdown under load
- forced shutdown after timeout


## PHASE 14 — WASM HOOK TESTING

Generate tests validating:

- hook loading
- ABI compatibility
- schema compatibility
- execution ordering
- sandbox isolation
- memory limits
- CPU limits
- panic/trap handling
- host capability restrictions
- tenant isolation
- concurrent hook execution


## PHASE 15 — HOOK SHADOW / PROMOTION

Generate tests validating:

- shadow hook execution
- shadow vs active comparison
- shadow telemetry
- promotion of shadow hooks
- promotion consistency across nodes
- rollback after promotion
- audit trail of promotion
- hook disabling/removal


## PHASE 16 — PERFORMANCE AND CONCURRENCY

Performance tests must define:

- request volume
- concurrency level
- payload size
- warmup duration
- test duration

and validate:

- P95 latency
- memory usage
- CPU usage
- queue depth

## PHASE 17 — FUZZING

Generate fuzz tests for:

- parsers
- policy evaluation inputs
- tool arguments
- streaming boundaries
- config parsing
- WASM hook inputs
- session transitions


## PHASE 18 — ORACLE DESIGN

Allowed oracle types:

- response_assertion
- log_assertion
- audit_assertion
- trace_assertion
- metric_assertion
- state_assertion
- contract_assertion
- schema_assertion
- config_roundtrip_assertion
- lifecycle_assertion

Each oracle must define:

- signal_sources
- pass_criteria
- fail_criteria


## PHASE 19 — OUTPUT STRUCTURE

Return JSON in this structure:

```json
{
  "master_spec": { ... },
  "child_specs": [
    {
      "file_name": "unit/unit_core_logic.json",
      "content": { ... }
    },
    {
      "file_name": "module/module_auth_policy.json",
      "content": { ... }
    }
  ]
}
```

All detailed tests must exist only inside child specs.
The master spec must serve as the index.


## SYSTEM DESCRIPTION


[INSERT SYSTEM SPECIFICATION HERE]
