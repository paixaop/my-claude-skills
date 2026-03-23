---
name: test-harness
description: Use when the user says 'generate test harness', 'test spec', 'testing specification', 'test plan', or needs structured JSON test specifications generated from system specs before plan generation
---

# PMP: Test Harness Generator

**Announce at start:** "Using pmp:test-harness to generate structured test specifications."

Generates structured JSON test specifications from system specs. Produces a master spec + child specs organized by testing layer and domain. Plan generation consumes these specs to ensure complete, structured test coverage.

**Always confirm output format and location before writing.** Use AskQuestion. Never auto-write.

## When NOT to Use

- No system specs exist yet → write specs first
- Want to review existing tests → use pmp:spec-review instead
- Want to generate a plan → use pmp:plan (which will prompt for test harness)

## Workflow

1. **REQUIRED:** Read [config.md](../pmp/config.md) for current constants — especially Test Harness Directory
2. Ask user for spec directory if not provided (default: `specs/` or project spec root)
3. Ask user for output format:
   - **JSON** (default) — structured, machine-consumable. Uses [testing-harness-prompt.md](references/testing-harness-prompt.md)
   - **Markdown** — human-readable multi-file spec. Uses [test-spec-generator-prompt.md](references/test-spec-generator-prompt.md)
4. Read the system specification completely
5. Apply the selected prompt to generate test specifications
6. Save output to test harness directory per [config.md](../pmp/config.md)
7. Present summary to user: N test files generated, M total tests, coverage by layer

## Output Structure (JSON mode)

```
specs/test-specs/
  master_spec.json
  unit/
    unit_core_logic.json
    unit_config_schema.json
  module/
    module_auth_policy.json
    module_observability.json
  pipeline/
    pipeline_integration.json
    pipeline_end_to_end.json
  resilience/
    resilience_shutdown.json
    resilience_dependency_failures.json
  performance/
    performance_load.json
  security/
    security_fuzzing.json
```

## Output Structure (Markdown mode)

```
specs/test-specs/
  system-testing-spec.md
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

## Key References

- JSON generator prompt: [testing-harness-prompt.md](references/testing-harness-prompt.md)
- Markdown generator prompt: [test-spec-generator-prompt.md](references/test-spec-generator-prompt.md)

## Shared Resources

- **REQUIRED:** [config.md](../pmp/config.md) — test harness directory and master spec filename
- **BACKGROUND:** [overview.md](../pmp/references/overview.md) for lifecycle context
