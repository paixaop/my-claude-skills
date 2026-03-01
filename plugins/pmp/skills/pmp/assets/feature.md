## Feature N: [Name] · #<issue-number>

<!-- Issue reference added after GitHub Issues are created. Omit if no issues. -->

**Behavior:** [What it does, user-visible outcomes]

**Affected Files:**
- Create: `path/to/new_file.ext`
- Modify: `path/to/existing.ext`

**API Contract** (if applicable):
- Endpoint / CLI command / function signature
- Inputs (with types and validation rules)
- Outputs (with types and status codes)
- Error cases

**Data Model** (if applicable):
- Schema changes, new tables/columns, migrations

**Integration Points:**
- What other components this touches
- External dependencies

### Acceptance Criteria

#### AC-N.1: [Criterion]

**E2E Test: [Scenario Name]**
- **Test File:** `path/to/featureN_e2e_test.ext` (code-file) or `<test-dir>/featureN-tests.md` (agent-driven)
- **Setup:** [Data/state to create before this test]
- **Steps:**
  1. [Action -- e.g., "Send POST /api/users with body {name, email}"]
  2. [Action -- e.g., "Send GET /api/users/{id}"]
- **Assertions:**
  - [Expected outcome -- e.g., "Response status 201, body contains id"]
  - [Expected outcome -- e.g., "GET returns the created user"]
- **Teardown:** [Cleanup if needed]
- **Pass Criteria:** [One sentence summary]

#### AC-N.2: [Criterion]

**E2E Test: [Scenario Name]**
- **Steps:** ...
- **Assertions:** ...
- **Pass Criteria:** ...

#### AC-N.E1: [Error/Edge Case Criterion]

**E2E Test: [Error Scenario]**
- **Steps:**
  1. [Action triggering error path]
- **Assertions:**
  - [Expected error response/behavior]

#### AC-N.S1: [Security Criterion]

**E2E Test: [Security Scenario]**
- **Steps:**
  1. [Action attempting exploit -- e.g., "Send POST with SQL injection in name field"]
- **Assertions:**
  - [Expected rejection -- e.g., "Response status 400, validation error, no DB corruption"]
