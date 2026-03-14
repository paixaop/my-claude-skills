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

### 5. Verify Coverage

Walk through every acceptance criterion and confirm it has an E2E test case directly beneath it. If any AC lacks a test, add one before saving.

### 6. Save the Plan

- **Bootstrap the directory:** `mkdir -p docs/plans` (create if it doesn't exist)
- Save using filename pattern from [config.md](../../pmp/config.md) File Paths
- **Frontmatter (MANDATORY):** Every plan MUST include YAML frontmatter as the very first content in the file (see [config.md](../../pmp/config.md) Plan Frontmatter). Set `status: draft` and `created_at` to the current UTC timestamp. All other fields are left blank.
- **GitHub Issues Mode:** Include the pre-filled GitHub Issues table (see [assets/github-issues-table.md](../../pmp/assets/github-issues-table.md)) in the plan file. Add `**Source:** GitHub Epic #<number>` to the plan header.

After saving, use AskQuestion: "Plan saved to docs/plans/. Ready to move to the review stage?"

Once confirmed, read [review.md](../../review/references/review.md) and follow it. Do NOT skip plan review.

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

- NO code snippets -- only behavioral specs
- Each AC has its E2E test case directly beneath it -- traceability is structural
- Features ordered by dependency (implement in this order)
- Each feature produces two commits: implementation (with unit/integration tests) and E2E tests. The implementation commit is the atomic boundary for the feature's behavior. The E2E commit verifies it.
- Integration branch and CI command recorded in the header (not hardcoded)
- Security ACs are E2E-testable assertions, not vague guidelines
- E2E test cases describe WHAT to verify, not HOW (no test code)
- Each feature's tests are self-contained with their own setup/teardown
- Error/edge cases and security scenarios are explicit ACs with tests, not afterthoughts
- Test runner command must be a single command that runs ALL E2E tests (code-file model)
- For hybrid model: note execution model (code-file or agent-driven) per AC test
- **Group changes by file** -- units of work should group changes by file if at all possible. When a task touches a file, all related changes to that file belong in the same task. This prevents parallel agents from conflicting on the same file and makes diffs reviewable.
