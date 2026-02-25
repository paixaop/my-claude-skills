# Testing Approaches

Per-project-type guide for selecting E2E frameworks, structuring test directories, managing test environments, and mitigating flakiness.

---

## Web Apps (Code-File Model)

### Framework

**Playwright** (preferred). Cross-browser, auto-wait, reliable selectors, built-in test runner. Cypress is an alternative but Playwright has better multi-tab/multi-origin support.

### Directory Convention

```
e2e/
  playwright.config.ts
  fixtures/
    setup.ts                    # shared server start/stop, seed data helpers
  suites/
    feature1.spec.ts
    feature2.spec.ts
```

### Test Lifecycle

```
beforeAll:  start dev server (or connect to running instance), seed data
beforeEach: navigate to starting page, reset state if needed
test:       actions + assertions
afterEach:  (optional) screenshot on failure
afterAll:   stop server, cleanup DB/temp files
```

### Test Runner Command

```
npx playwright test               # run all
npx playwright test suites/       # run specific directory
npx playwright test --project=chromium  # single browser
```

### Isolation

- Each spec file manages its own test data via API calls or UI actions in `beforeAll`/`beforeEach`
- Use unique identifiers (timestamps, UUIDs) in test data to avoid collisions during parallel runs
- Clean up created data in `afterAll`

### Flakiness Mitigation

- Use `data-testid` attributes for selectors instead of CSS classes or text content
- Prefer `await page.waitForSelector()` over arbitrary timeouts
- Use `networkidle` or specific response waits after navigation
- Set `actionTimeout` and `navigationTimeout` in config (not per-test)
- Run with `--retries=1` during development to catch flaky tests early

---

## Web Apps (Agent-Driven Model)

### Framework

**Playwright MCP** browser tools. No code files written -- the agent reads test spec markdown files and executes actions interactively.

### Directory Convention

```
e2e/
  feature1-tests.md             # test spec (markdown, not code)
  feature2-tests.md
```

### Test Lifecycle

The agent manages the lifecycle directly:

1. Start the system under test (via shell command)
2. Navigate to the application
3. Execute test specs sequentially
4. Report results
5. Stop the system, cleanup

### Test Spec Format

Each spec file follows this pattern:

```markdown
# [Feature Name] Tests

## Prerequisites
- [Server running, logged in, specific page loaded, etc.]

## Tests

### Test 1: [Name]
1. **Action** -- [click, fill, navigate, etc.]
2. **Snapshot** -- verify [expected state]
3. **Action** -- [next step]
4. **Snapshot** -- verify [expected result]

**Pass criteria:** [One sentence]
```

### Isolation

- Each spec file starts by navigating to its starting page
- Specs create their own test data during execution
- Specs clean up created data before finishing (or the suite teardown handles it)

### Flakiness Mitigation

- Always call `browser_snapshot` after every action before asserting
- Use `browser_wait_for` with expected text/element before interacting
- Short incremental waits (1-3s) with snapshot checks, not single long waits
- Never reuse element refs from a previous snapshot -- always re-snapshot

---

## HTTP APIs

### Framework

Language-native HTTP test clients. No external testing framework needed -- use the language's built-in test runner:

| Language | HTTP Client | Test Runner | Test Files |
|---|---|---|---|
| **Go** | `net/http` (stdlib) | `go test` | `e2e/*_test.go` |
| **Python** | `httpx` or `requests` | `pytest` | `e2e/test_*.py` |
| **Node/TS** | `supertest` or native `fetch` | `vitest` or `jest` | `e2e/*.test.ts` |
| **Rust** | `reqwest` | `cargo test` | `tests/e2e/*.rs` |

### Directory Convention

```
e2e/
  helpers/
    client.{ext}                # HTTP client wrapper, base URL config
    fixtures.{ext}              # seed data helpers
  feature1_test.{ext}
  feature2_test.{ext}
```

### Test Lifecycle

```
suite setup:    start server process, wait for health check, seed base data
test setup:     create test-specific data via API calls (not direct DB)
test:           HTTP request(s) + response assertions
test teardown:  delete test-specific data via API calls
suite teardown: stop server, remove temp DB/files
```

### Test Runner Command

```
go test ./e2e/ -v -count=1          # Go
pytest e2e/ -v                       # Python
npx vitest run e2e/                  # Node/TS
```

### Isolation

- Seed data via the API itself (POST endpoints), not by manipulating the DB directly
- Tests should use the same interface as real clients
- Each suite creates its own data with unique identifiers and cleans up after
- Use a fresh DB/temp directory per test run if possible

### Flakiness Mitigation

- Wait for server health check before running tests (retry loop with backoff)
- Set explicit HTTP timeouts in the test client
- Avoid testing timing-dependent behavior (e.g., "response within 100ms")
- Use deterministic test data (not random) for reproducibility

---

## CLI Tools

### Framework

Subprocess execution using the language's standard process API. No external test framework needed beyond the built-in test runner.

| Language | Process API | Example |
|---|---|---|
| **Go** | `os/exec` | `exec.Command("./mybinary", "arg1", "arg2")` |
| **Python** | `subprocess` | `subprocess.run(["./mybinary", "arg1"], capture_output=True)` |
| **Node** | `node:child_process` (use `execFile`, not `exec`) | `execFileSync("./mybinary", ["arg1"])` |

### Directory Convention

```
e2e/
  helpers/
    runner.{ext}                # wrapper that builds + runs the CLI binary
  feature1_test.{ext}
  feature2_test.{ext}
```

### Test Lifecycle

```
suite setup:    build the binary, create temp directory for test artifacts
test setup:     create temp config files / input files as needed
test:           run binary with args, assert on stdout, stderr, exit code, output files
test teardown:  remove temp files created by the test
suite teardown: remove temp directory, remove built binary (optional)
```

### Test Runner Command

```
go test ./e2e/ -v -count=1          # Go
pytest e2e/ -v                       # Python
npx vitest run e2e/                  # Node
```

### Isolation

- Use a temp directory per suite (`t.TempDir()` in Go, `tmp_path` in pytest)
- Each test creates its own input files and config
- Tests assert on output (stdout/stderr/files), not internal state
- Binary should be built once per suite, not per test

### Flakiness Mitigation

- Set explicit process timeouts (kill after N seconds)
- Avoid timing-dependent assertions on stderr output order
- Use unique temp directories to prevent cross-test interference
- Flush stdout/stderr before asserting (some CLIs buffer output)

---

## Libraries / Packages

### Framework

Integration tests using the language's test runner. Tests exercise the public API in realistic scenarios, not internal implementation details.

### Directory Convention

```
e2e/                            # or tests/integration/
  scenario1_test.{ext}          # realistic consumer scenario
  scenario2_test.{ext}
  testdata/                     # sample files, fixtures
```

### Test Lifecycle

```
suite setup:    initialize shared resources (DB connections, file systems, etc.)
test setup:     create scenario-specific state
test:           call public API functions in a realistic sequence, assert results
test teardown:  clean up scenario state
suite teardown: close connections, remove temp resources
```

### Test Runner Command

Same as the project's unit test runner, but targeting the e2e directory:

```
go test ./e2e/ -v -count=1
pytest e2e/ -v
npx vitest run e2e/
```

### Isolation

- Each test scenario is self-contained
- Use realistic data volumes where performance matters
- Test only the public interface (exported functions/types)
- Don't mock dependencies that are part of the "unit under integration"

### Flakiness Mitigation

- Avoid shared mutable state between tests
- Use deterministic test data
- If tests involve concurrency, use synchronization primitives, not sleeps
- Set generous but bounded timeouts for long-running operations

---

## Choosing Between Models

Quick decision guide when project detection is ambiguous:

| Question | If Yes | If No |
|---|---|---|
| Do you want tests in CI? | Code-file | Either |
| Is the UI highly dynamic (SPAs, complex state)? | Agent-driven | Code-file |
| Is this a pure API with no UI? | Code-file | Consider both |
| Do you need tests to run without the agent? | Code-file | Either |
| Is token cost a concern? | Code-file (cheaper) | Either |

When in doubt, default to **code-file tests**. They're cheaper, CI-compatible, and the test artifacts persist.
