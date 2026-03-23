## Feature N: [Name] · #<issue-number>

<!-- Issue reference added after GitHub Issues are created. Omit if no issues. -->

**Behavior:** [What it does, user-visible outcomes]

**Spec Source:** [file.md#section](file.md#section), [file2.md#section](file2.md#section)

**Spec Requirements:**
> - [Quoted requirement from spec using normative language]
> - [e.g., "The middleware MUST reject requests without a valid JWT token with HTTP 401"]
> - [e.g., "Rate limiting MUST use the `settings.security.rate_limit_rpm` setting"]

**Affected Files:**
- Create: `path/to/new_file.ext`
- Modify: `path/to/existing.ext`

**API Contract** (if applicable):
- Endpoint / CLI command / function signature
- Inputs (with types and validation rules)
- Outputs (with types and status codes)
- Error cases (specific HTTP status, error body schema, error code)

**Data Model** (if applicable):
- Schema changes, new tables/columns, migrations

**Integration Points:**
- What other components this touches
- External dependencies

**Depends On:** [Feature X, Feature Y] or None

### Tasks

<!-- One task per file. Dependencies determine execution order and parallelism. -->

| Task | File | Action | Dependencies | Parallel |
|------|------|--------|-------------|----------|
| TN.1 | `path/to/impl.go` | Create | None | Yes |
| TN.2 | `path/to/impl_test.go` | Create | TN.1 | No |
| TN.3 | `path/to/other.go` | Modify | None | Yes (with TN.1) |
| TN.4 | `path/to/other_test.go` | Create | TN.3 | No |
| TN.5 | `path/to/handler.go` | Modify | TN.1, TN.3 | No |
| TN.6 | `tests/e2e/feature_test.go` | Create | All impl tasks | After impl |

### Task Dependencies

<!-- ASCII diagram showing parallel lanes -->

```
TN.1 ──→ TN.2
   ↘
    TN.5 → TN.6
   ↗
TN.3 ──→ TN.4
```

### Testing Layers

<!-- Explicitly state which layers apply to this feature -->

| Layer | Applies | Rationale |
|-------|---------|-----------|
| Unit tests | Yes | Always — validators, parsers, state transitions |
| Module tests | [Yes/No] | [Why — e.g., "Self-contained auth module with mockable deps"] |
| Integration tests | [Yes/No] | [Why — e.g., "Touches auth + policy + handler chain"] |
| E2E tests | [Yes/No] | [Why — e.g., "New endpoint exposed at system boundary"] |

### Acceptance Criteria

#### AC-N.1: [Criterion — use normative language from spec]

**Test Harness:** `[layer]/[file].json` → `[TEST_ID]` (if harness exists)

**RED — Write failing test first:**
- File: `path/to/test_file.ext`
- [What the test does — e.g., "Send request without Authorization header"]
- Assert: [Expected outcome — e.g., "Response status 401, body `{\"error\": \"missing_auth_token\"}`"]
- **Run test → MUST FAIL** (feature not yet implemented)

**GREEN — Minimal implementation:**
- File: `path/to/impl_file.ext`
- [What to implement — only enough to make the test pass]
- **Run test → MUST PASS**

**REFACTOR — Clean up:**
- [Optional cleanup — extract helpers, improve names]
- **Run test → MUST STILL PASS**

#### AC-N.E1: [Error/Edge Case — use normative language]

**RED:** [Failing test for error path]
**GREEN:** [Implementation of error handling]
**REFACTOR:** [Cleanup]

#### AC-N.S1: [Security Criterion — use normative language]

**RED:** [Failing test for security scenario]
**GREEN:** [Implementation of security control]
**REFACTOR:** [Cleanup]
