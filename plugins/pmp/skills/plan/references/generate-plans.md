# Generate Plans

Create an implementation plan from a roadmap. Each feature includes both its implementation spec and E2E test cases together — acceptance criteria and their verification live side by side, no cross-file references.

## Input

A roadmap in any of these forms:

| Input Type | Examples |
|------------|----------|
| **File path** | `docs/roadmap.md`, `requirements.txt` |
| **Inline text** | Feature list pasted in chat |
| **URL** | Link to a Google Doc, Notion page, etc. |
| **GitHub Issues** | Epic number (`#41`), issue URL, or "plan from issues" — triggers GitHub Issues Mode (see below) |

## E2E Project Detection

Before generating a plan, auto-detect and confirm with the user:

### 1. Project Type and E2E Testing Approach

| Project Type | Detection Signals | E2E Testing Approach |
|---|---|---|
| **Web App (frontend)** | `package.json` with React/Vue/Svelte/Next, HTML templates | Playwright test scripts or Playwright MCP browser tests |
| **Web App (fullstack)** | Both frontend + backend server | Playwright for UI + HTTP client for API |
| **REST/HTTP API** | Go/Python/Node server, no UI | Language-native HTTP test client |
| **CLI Tool** | `main.go`/`main.py` with cobra/argparse/click, no server | Subprocess execution + stdout/stderr/exit-code assertions |
| **Library/Package** | No main entrypoint, exported packages | Integration tests with realistic consumer scenarios |
| **gRPC/WebSocket** | Proto files, WS handlers | Protocol-specific test clients |

### 1b. Monorepo Detection

Check for workspace configuration files that signal a monorepo:

| Signal | Tool |
|---|---|
| `pnpm-workspace.yaml` | pnpm workspaces |
| `lerna.json` | Lerna |
| `nx.json` | Nx |
| `turbo.json` | Turborepo |
| `Cargo.toml` with `[workspace]` | Rust workspaces |
| `go.work` | Go workspaces |
| Multiple `package.json` files in subdirectories | npm/yarn workspaces |

If a monorepo is detected:

1. **Ask which package(s) the plan targets** — use AskQuestion to present the list of packages/apps
2. **Scope detection per package** — project type, CI command, test directory, and integration branch may differ per package
3. **Record package scope in the plan header** — add `**Package:** <path>` (e.g., `packages/api/`)
4. **Scope E2E tests to the target package** — test runner commands and directories are relative to the package root
5. **CI command** — may be a workspace-level command (e.g., `turbo test --filter=api`) or package-level

Present detections per package for user confirmation.

### 2. Project Conventions

| Convention | Detection Method | Fallback |
|---|---|---|
| **Integration branch** | `git branch -a` for `dev`/`develop`/`main`/`master`; check CLAUDE.md / CONTRIBUTING.md | Ask the user |
| **CI command** | `Makefile` targets (`ci`, `test`), `package.json` scripts, `.github/workflows/`, `Justfile` | Ask the user |
| **Test directory** | Existing `e2e/`, `tests/e2e/`, `test/`, `__tests__/`, `*_test.go` patterns | Propose based on project type |

Present ALL detections to the user for confirmation. User can override any.

### 3. Execution Model Selection

| Model | When | How |
|---|---|---|
| **Code-file tests** | Default for API, CLI, Library | Agent writes test code files, runs with single command, parses output. Persistent, CI-runnable. |
| **Agent-driven tests** | Option for Web Apps | Agent reads test spec markdown files, executes via Playwright MCP interactively. No test code files. |
| **Hybrid** | Fullstack projects | Code-file for API tests + chosen model for UI tests. Plan notes model per AC. |

Commit to one model during project detection. Confirm with the user before plan generation.

### 4. E2E Test Opt-Out

Some projects or features legitimately don't benefit from E2E tests. When in doubt, ask the user.

**Auto-detect and flag these cases:**

| Project Signal | Reason E2E May Not Apply |
|---|---|
| Pure utility library (no I/O, no server, no CLI) | Only unit/integration tests are meaningful |
| Type definitions or schema-only packages | Nothing to execute end-to-end |
| Documentation-only changes | No behavior to test |
| Configuration or infrastructure changes | Better tested by CI/CD validation |

When any of these are detected, use AskQuestion:

> "This looks like a [type]. E2E tests may not add value here. Options:"
> 1. **Include E2E tests anyway** — full E2E infrastructure and per-feature tests
> 2. **Skip E2E, use integration tests** — plan includes integration tests but no E2E infrastructure
> 3. **Skip E2E, unit tests only** — plan includes unit tests only

If the user opts out of E2E:
- Plan generation skips the E2E Test Infrastructure section
- Feature specs include test cases but marked as unit or integration tests
- Execute loop runs the project's test runner instead of a dedicated E2E suite
- The code-test-fix loop still applies (tests must pass before commit)

**Default:** If unsure whether E2E tests apply, ask the user. Never silently skip them.

---

## Process

### 0. Fetch Issues (GitHub Issues Mode Only)

**When the input is a GitHub issue number, URL, or the user says "plan from issues" / "plan from epic":**

This step fetches the issues and normalizes them into a roadmap before continuing to step 1.

**a. Identify the source:**

```bash
# Get repo context
OWNER=$(gh repo view --json owner --jq '.owner.login')
REPO=$(gh repo view --json name --jq '.name')
REPO_ID=$(gh repo view --json id --jq '.id')
```

**b. Fetch the epic and its sub-issues:**

```bash
gh api graphql -f query='
  query($owner: String!, $repo: String!, $number: Int!) {
    repository(owner: $owner, name: $repo) {
      issue(number: $number) {
        number
        title
        body
        labels(first: 10) { nodes { name } }
        milestone { title }
        subIssues(first: 50) {
          nodes {
            number
            title
            body
            labels(first: 10) { nodes { name } }
          }
        }
      }
    }
  }
' -f owner="$OWNER" -f repo="$REPO" -F number=<epic-number>
```

If the input is a single issue (not an epic), fetch just that issue:
```bash
gh issue view <number> --json number,title,body,labels,milestone
```

**c. Parse each issue into a feature spec:**

For each sub-issue (or the single issue):
- **Title** → Feature name
- **Body → Description** section → Behavior description
- **Body → Verification** table → Acceptance criteria seed
- **Body → Definition of Done** → Additional ACs
- **Body → Source Context** → Original spec excerpt (if present)
- **Labels** → Priority, type classification

**d. Build the `## GitHub Issues` table immediately:**

Since the issues already exist, pre-fill the mapping table that will be written into the plan:

```markdown
## GitHub Issues
| Feature | Issue | Status |
|---------|-------|--------|
| Feature 1: [title from #42] | #42 | Open |
| Feature 2: [title from #43] | #43 | Open |
| Epic | #41 | Open |
```

**e. Present to user for confirmation:**

Show the parsed features and ask:
- "I found N sub-issues under Epic #X. Here's how I'll map them to plan features: [list]. Does this look right?"
- Confirm ordering and dependencies
- Flag any issues with incomplete bodies that need clarification

**f. Proceed to step 1** with the normalized roadmap.

### 1. Read the Roadmap

Read it completely. Understand every feature, requirement, and acceptance criterion before proceeding.

For GitHub Issues Mode: the "roadmap" is the normalized feature list from step 0.

### 1b. Locate SSoT Index and Rules

**If the project has spec files** (under `specs/`, `docs/architecture/`, or similar):

1. **Look for `spec-index.md`** in the spec directory root. If found, read it — this is the ownership registry that maps concepts to their canonical files.
2. **Look for SSoT rules** in CLAUDE.md, the spec directory README.md, or `docs/single-source-of-truth.md`. If found, read them — all plan features that touch spec-owned concepts must reference the canonical source.

**If an SSoT index exists**, use it throughout plan generation:
- When a feature references a concept, link to the canonical file from the index — not to duplicates or summaries
- When a feature modifies a spec-owned behavior, the plan must include updating the canonical spec file as an explicit step
- When a feature adds a new concept, the plan must include adding it to the SSoT index as an explicit step

**If no SSoT index exists** but spec files exist, note this and suggest running `/pmp:spec-index` after plan generation.

### 2. Analyze the Codebase

Use parallel agents if available, otherwise sequential:

- Detect project type, language, framework, build system
- Find existing test infrastructure (test directories, frameworks, CI config)
- Detect integration branch and CI command (see E2E Project Detection above)
- Map current architecture (entry points, data layer, API surface)
- Identify relevant files per roadmap feature

### 3. Ask Clarifying Questions

Use the AskQuestion tool. Do NOT proceed with ambiguity:

- Confirm detected project type, E2E approach, and execution model (code-file vs agent-driven)
- Confirm detected integration branch and CI command
- Clarify any unclear requirements
- Confirm feature priority/ordering if not specified in the roadmap

### 4. Generate the Plan

Follow the Plan Structure below. For each feature, write both the implementation spec and its E2E test cases together.

### 4b. Auto-Phase (5+ Features)

If the plan has **5 or more features**, group them into phases after generating the dependency graph:

1. **Build dependency layers** — features with no dependencies form Phase 1. Features depending only on Phase 1 features form Phase 2, and so on.
2. **Apply cohesion** — if two features in the same layer are closely related (same domain area, same files), keep them together. If a layer has more than 5 features, split into sub-phases by cohesion.
3. **Name each phase** — use a descriptive name reflecting the group's purpose (e.g., "Foundation", "Core Logic", "User-Facing Polish").
4. **Define entry/exit criteria** — entry is the prior phase's exit criteria being met. Exit is all phase features' E2E tests passing + CI green.
5. **Minimize phase count** — don't create phases for the sake of it. 2-3 phases for 5-7 features, 3-4 for 8-12, 4+ for 12+.
6. **Insert the `## Phases` section** — between `## Feature Dependency Graph` and the first feature (see [plan.md](../assets/plan.md) template).

For plans with fewer than 5 features, omit the `## Phases` section entirely.

### 5. Verify Coverage

Walk through every acceptance criterion and confirm it has an E2E test case directly beneath it. If any AC lacks a test, add one before saving.

### 6. Save the Plan

- **Bootstrap the directory:** `mkdir -p docs/plans` (create if it doesn't exist)
- Save using filename pattern from [config.md](../../pmp/config.md) File Paths
- **Frontmatter (MANDATORY):** Every plan MUST include YAML frontmatter as the very first content in the file (see [config.md](../../pmp/config.md) Plan Frontmatter). Set `status: draft` and `created_at` to the current UTC timestamp. All other fields are left blank.
- **GitHub Issues Mode:** Include the pre-filled GitHub Issues table (see [assets/github-issues-table.md](../../pmp/assets/github-issues-table.md)) in the plan file. Add `**Source:** GitHub Epic #<number>` to the plan header.

After saving, use AskQuestion: "Plan saved to docs/plans/. Ready to move to the review stage?"

Once confirmed, read [review.md](../../plan-review/references/review.md) and follow it. Do NOT skip plan review.

---

## Extend Mode

When a plan already exists and the user provides an updated roadmap:

### 1. Read Existing Plan

Read the plan completely.

### 2. Classify Roadmap Items

For each item in the new/updated roadmap:

- **New feature**: doesn't exist in current plan
- **Modified feature**: changes behavior of an existing feature
- **Removed feature**: no longer needed (user must confirm removal)

### 3. Apply Changes

**New features:**
- Append with next sequential feature number (implementation spec + E2E tests together)
- Add dependency links to existing features where applicable

**Modified features:**
- Update the existing feature spec in-place
- Mark with `**Updated: YYYY-MM-DD**`
- Update the E2E test cases under that feature to match the new behavior

**Removed features:**
- Mark as `**Removed: YYYY-MM-DD**` (don't delete -- audit trail)
- Mark the E2E test cases as skipped

### 4. Re-verify Coverage

Walk through every active (non-removed) feature and confirm all ACs have test cases.

### 5. Present Change Summary

Before saving, present a structured diff summary to the user for confirmation:

```
## Plan Changes: [Plan Name]

### Added
| Feature | Dependencies | ACs | E2E Tests |
|---------|-------------|-----|-----------|
| Feature N: [Name] | [deps] | [count] | [count] |

### Modified
| Feature | What Changed |
|---------|-------------|
| Feature K: [Name] | [sections changed — e.g., "AC-K.2 updated, new affected file"] |

### Removed
| Feature | Reason |
|---------|--------|
| Feature J: [Name] | [reason from user or roadmap] |

### Unchanged
- Feature 1, Feature 2, ...
```

Use AskQuestion: "Here are the planned changes. Apply them?"

### 6. Save Updated Plan

Same filename. Preserve existing frontmatter — do NOT reset `status` or timestamps from prior stages. Add a changelog section below the frontmatter/header:

```markdown
## Changelog

- **YYYY-MM-DD**: Added Feature N, Feature M. Modified Feature K. Removed Feature J.
```

---

## Plan Structure

Use [assets/plan.md](../assets/plan.md) for the overall plan structure. For each feature section within the plan, use [assets/feature.md](../assets/feature.md).

If the plan was generated from GitHub Issues, include a [assets/github-issues-table.md](../../pmp/assets/github-issues-table.md) section.

### Plan Rules

- NO code snippets — only behavioral specs with normative language from the spec
- Each AC structured as Red-Green-Refactor (not Setup → Steps → Assertions)
- Features ordered by dependency; feature dependency matrix enables parallel execution
- **One task per file** — each task creates or modifies exactly one file. Dependencies between tasks are explicit.
- Testing layers per feature: unit (always), module (self-contained components), integration (cross-module), E2E (system boundary). Each feature explicitly lists which apply with rationale.
- Integration branch and CI command recorded in the header (not hardcoded)
- Security ACs are testable assertions with specific HTTP status, error codes, and behaviors — not vague guidelines
- Test cases describe WHAT to verify using Red-Green-Refactor format
- Each feature's tests are self-contained with their own setup/teardown
- Error/edge cases and security scenarios are explicit ACs with tests, not afterthoughts
- Test runner command must be a single command that runs ALL tests (code-file model)
- For hybrid model: note execution model (code-file or agent-driven) per AC test
- **Spec traceability** — every spec section maps to a feature + AC; uncovered sections are flagged
- **Test harness** — when available, ACs reference harness test IDs instead of inventing tests

### Spec Fidelity Rules

Plans MUST follow specs to the letter. Generic descriptions cause coding agents to drift from the spec.

**Spec Anchoring:**
- Every feature MUST have a `**Spec Source:** [file.md#section](file.md#section)` field linking to the exact spec section(s) it implements
- Every AC MUST quote or paraphrase the spec requirement it validates — not rewrite it in the agent's own words
- Use normative language from the spec: if the spec says "MUST", the AC says "MUST". If the spec says "SHOULD", the AC says "SHOULD". Never downgrade "MUST" to "should" or "handles."

**Traceability Matrix:**
- After generating all features, produce a `## Spec Traceability` section that maps every spec section to the feature + AC that covers it
- Any spec section without coverage MUST be flagged as "Not covered — intentional (reason)" or added as a feature
- Any plan feature without a spec source MUST be flagged as "Gold-plating — not in spec" and removed unless the user explicitly approves it

**Precision Rules — no vague language:**
- Error responses: specify HTTP status, error body schema, error code, and log level — not "returns an error"
- Thresholds: specify the exact value and the setting name from the settings catalog — not "rejects large requests"
- State transitions: specify from-state, to-state, trigger, and side effects — not "updates the status"
- Config keys: use the exact setting name from the settings catalog — not "the timeout setting"
- Timeouts: specify the duration and the setting key — not "with appropriate timeout"

### Test Harness Integration

Before generating features, check for a structured test harness:

1. Look for test harness files in the project:
   - `specs/test-specs/master_spec.json` (JSON format — per [config.md](../../pmp/config.md))
   - `specs/test-specs/system-testing-spec.md` (Markdown format)
   - Or any directory the user points to
2. **If found:** ask the user with AskQuestion:

> Found existing test harness at `<directory>` ([N] test files, [M] total tests).
> Do you want to use it for plan generation?
> 1. **Yes** — I'll reference these test specs in the plan's ACs
> 2. **No, regenerate** — I'll run `/pmp:test-harness` to create a fresh one
> 3. **No, skip** — I'll generate test cases inline without a harness

   If option 1: read the master spec, resume plan generation with harness references
   If option 2: run `/pmp:test-harness`, then resume
   If option 3: continue with inline test generation

3. **If NOT found:** ask the user with AskQuestion:

> No test harness found. A test harness is a structured testing specification (JSON or Markdown) that defines every test — purpose, inputs, assertions, oracle — so the plan doesn't invent tests from scratch.
>
> **Options:**
> 1. **Point me to an existing test harness** — if you already have test specs somewhere else
> 2. **Generate a test harness now** — I'll run `/pmp:test-harness` to create one from your system specs (you may need to customize the generator prompt for your project)
> 3. **Continue without a test harness** — I'll generate test cases inline in each AC (less structured, higher risk of gaps)

4. If option 1: user provides path, read it, resume plan generation
5. If option 2: run `/pmp:test-harness`, then resume plan generation
6. If option 3: continue with inline test generation (current behavior)

**Important:** The test harness generator prompts (`testing-harness-prompt.md` and `test-spec-generator-prompt.md`) are templates. They may need project-specific customization — invariant IDs, component names, language-specific test conventions — before they produce useful output. The `/pmp:test-harness` sub-command handles this by reading the project's specs and adapting the prompt.

**When a test harness exists (any format):**
- Each AC links to the specific test from the harness: `**Test Harness:** module/module_auth_policy.json → AUTH_001` (JSON) or `**Test Harness:** module-testing.md § Auth Module Tests` (Markdown)
- Test tasks implement the tests defined in the harness — the harness is the contract, not a suggestion
- After generating all features, cross-reference spec traceability against harness coverage matrix — any gap = missing test coverage
- Performance/Resilience/Fuzz plan sections are populated from harness categories, not invented
- If the harness defines invariant IDs (e.g., INV-PIPE-001), ACs reference them for traceability

### Single-File Task Granularity

Each task in the plan MUST target exactly ONE file. When a feature creates or modifies multiple files, decompose into one task per file.

**Task table format:**

```markdown
| Task | File | Action | Dependencies | Parallel |
|------|------|--------|-------------|----------|
| T1.1 | `internal/auth/middleware.go` | Create | None | Yes |
| T1.2 | `internal/auth/middleware_test.go` | Create | T1.1 | No |
| T1.3 | `internal/policy/evaluator.go` | Modify | None | Yes (with T1.1) |
| T1.4 | `internal/policy/evaluator_test.go` | Create | T1.3 | No |
| T1.5 | `internal/server/handler.go` | Modify | T1.1, T1.3 | No |
| T1.6 | `tests/e2e/auth_test.go` | Create | T1.5 | After all impl |
```

**Rules:**
- Each task targets exactly ONE file (create or modify)
- Test file tasks depend on their implementation file task
- Tasks with no mutual dependencies CAN run as parallel agents
- The Dependencies column is the execution contract — the orchestrator uses it to decide parallelism
- Every feature MUST include a task dependency graph showing parallel lanes

**Task dependency graph per feature:**

```
T1.1 ──→ T1.2
   ↘
    T1.5 → T1.6
   ↗
T1.3 ──→ T1.4
```

### Feature Dependency Matrix

Each phase MUST include a feature dependency matrix so the controller can parallelize feature orchestrators:

```markdown
## Feature Dependencies

| Feature | Depends On | Can Parallel With |
|---------|-----------|-------------------|
| Feature 1: Auth Middleware | None | Feature 2, Feature 3 |
| Feature 2: Policy Engine | None | Feature 1, Feature 3 |
| Feature 3: Config Loader | None | Feature 1, Feature 2 |
| Feature 4: Request Pipeline | Feature 1, Feature 2 | None (blocked) |
```

### Red-Green-Refactor Testing

All test ACs MUST be structured as Red-Green-Refactor cycles, not Setup → Steps → Assertions.

**AC format:**

```markdown
#### AC-1.1: Auth middleware MUST reject requests without valid JWT

**RED — Write failing test first:**
- File: `internal/auth/middleware_test.go`
- Test sends request without Authorization header
- Assert: response is 401 with body `{"error": "missing_auth_token"}`
- **Run test → MUST FAIL** (middleware not yet implemented)

**GREEN — Minimal implementation:**
- File: `internal/auth/middleware.go`
- Implement JWT validation — only enough to make this test pass
- **Run test → MUST PASS**

**REFACTOR — Clean up:**
- Extract JWT validation into reusable helper if pattern repeats
- **Run test → MUST STILL PASS**
```

If the test passes immediately in RED, the test is wrong — it tests existing behavior, not the new requirement. The agent must fix the test before proceeding.

### Large-Spec Plan Decomposition

When the spec corpus has 50+ files (per [config.md](../../pmp/config.md) Large Spec Threshold):

1. **Announce:** "This is a large specification (N files). I'll generate a master plan with sub-plans per phase."
2. **Group specs** by architectural concern (building blocks, runtime, crosscutting, deployment, etc.)
3. **Build dependency graph** between groups
4. **Propose phases** to user — each phase = one group or set of tightly coupled groups. Use AskQuestion: "I've identified P phases: [list]. Does this grouping look right?"
5. **Generate master plan** using [master-plan.md](../assets/master-plan.md) template — contains spec coverage summary, phase dependency graph, cross-phase traceability
6. **Generate one sub-plan per phase** — each is a normal plan file scoped to that phase's spec files. Filename: per [config.md](../../pmp/config.md) Sub-Plan Filename Pattern
7. **Announce progress:** "Phase N sub-plan generated (M features). Moving to Phase N+1."

Sub-plans include:
- `**Master Plan:** [link]` in header
- Feature dependency matrix (for parallel execution within the phase)
- Spec traceability (subset of master)
- Test harness references (subset of harness for this phase's components)

### Orchestrator Agent Execution Model

Plans are structured for a three-tier execution model:

```
Controller (session, max 3 features per batch)
  └── Feature Orchestrator Agent (one per feature)
      ├── Dispatches single-file task agents (parallel where dependencies allow)
      ├── Waits for dependency resolution, dispatches next wave
      ├── After all impl tasks: runs spec review inline
      ├── Dispatches test task agents (parallel where independent)
      ├── Verifies Red-Green-Refactor: each test MUST fail before passing
      ├── After all tests: runs code quality review inline
      └── Reports structured summary to controller
```

The orchestrator does NOT implement code — it only dispatches, reviews, and coordinates. Implementation agents are fresh subagents per task.

Features with no mutual dependencies (per feature dependency matrix) execute as parallel orchestrators.

### SSoT Compliance Rules

When the project has spec files with SSoT conventions, every plan MUST follow these rules so the executing agent maintains spec integrity:

- **Reference canonical sources** — Every feature that implements a spec-defined behavior must link to the canonical spec file (from the SSoT index). Never reference duplicates, summaries, or inline copies.
- **One concept = one file** — If a feature adds a new architectural concept, the plan must include a task to create its canonical spec file. No new concepts defined only in code comments or inline docs.
- **Update specs alongside code** — If a feature changes spec-owned behavior, the plan must include an explicit task to update the canonical spec file. Code changes without spec updates are incomplete.
- **No duplication** — Plan tasks must never instruct the agent to duplicate spec content into other files. Cross-reference with section-level links (`file.md#section`) instead.
- **Update the SSoT index** — If the plan creates or removes spec files, include a task to update `spec-index.md`.
- **Stable anchors** — If a plan task renames a heading in a spec file, it must include updating all inbound cross-reference links. Use grep to find them.
- **Section-level links** — All cross-references in plan tasks must use `file.md#section` format, not bare file links, so the executing agent links to the exact location.
- **No magic numbers** — Numeric literals in plan tasks (thresholds, limits, timeouts) must reference a named setting from the settings catalog. If the setting doesn't exist yet, include a task to add it.
