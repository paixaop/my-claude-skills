# Test Layer Requirements (Phases 8–19)

Per-layer test generation requirements for the testing harness prompt. This file is a continuation of [testing-harness-prompt.md](testing-harness-prompt.md).

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
