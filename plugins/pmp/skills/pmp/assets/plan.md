---
status: draft
created_at: "YYYY-MM-DDTHH:MM:SSZ"
reviewed_at:
issues_created_at:
execution_started_at:
completed_at:
pr:
epic:
---

# [Feature Set] Implementation Plan

**Goal:** [One sentence]
**Source:** [Roadmap file path | "GitHub Epic #N" | "Inline spec"]
**Project Type:** [Detected type]
**Stack:** [Language, framework, DB, etc.]
**Integration Branch:** [Detected branch name]
**CI Command:** [Detected CI command]
**Epic:** [#<number> — added after GitHub Issues are created]

> **To execute this plan:** run `/pmp execute` and point it at this file.

---

## GitHub Issues

<!-- Added after issues are created. Maps plan hierarchy to GitHub issues.
     See config.md Plan-GitHub Mapping for the full hierarchy. -->

| Level | Name | Issue | Status |
|-------|------|-------|--------|
| Epic | [Plan Name] | #<number> | Open |
| Feature | Feature 1: [Name] | #<number> | Open |
| Task | Task A: Implement [Name] | #<number> | Open |
| Task | Task B: E2E tests for [Name] | #<number> | Open |
| Feature | Feature 2: [Name] | #<number> | Open |
| Task | Task A: Implement [Name] | #<number> | Open |
| Task | Task B: E2E tests for [Name] | #<number> | Open |

---

## E2E Test Infrastructure

**Testing Approach:** [From project detection]
**Execution Model:** [code-file | agent-driven | hybrid]
**Test Runner Command:** `[exact command]` (code-file model only)
**Test Directory:** `[path where E2E test files live]`

**Framework:** [What to install/configure]

**Test Environment:**
- How to start the system under test (server, CLI, etc.)
- Seed data / fixtures needed
- Environment variables
- Cleanup procedure

**Test Runner Configuration:**
- Config file format and location
- Parallel vs sequential execution
- Timeout settings

**Test Isolation Rules:**
- Each feature's E2E tests MUST handle their own setup and teardown
- Test groups must be runnable independently (no implicit ordering dependencies)
- If tests need data from a "prior" feature, they must create that data in their own setup
- Shared fixtures go in a common setup helper, not in test ordering

---

## Feature Dependency Graph

<!-- After issues are created, each feature shows its issue number -->

1. Feature 1: [Name] (#<number>) (no dependencies)
2. Feature 2: [Name] (#<number>) (depends on Feature 1)
3. Feature 3: [Name] (#<number>) (depends on Feature 1)
4. Feature 4: [Name] (#<number>) (depends on Feature 2, Feature 3)

---

<!-- Repeat Feature sections using assets/feature.md structure -->
<!-- After issues are created, feature headings include the issue reference -->

## Feature 1: [Name] · #<number>

...

---

## Security Considerations

- **Input validation:** [What inputs need validation, how]
- **Auth boundaries:** [What auth checks are needed]
- **Secret handling:** [How secrets are protected]
- **Injection risks:** [SQL, command injection vectors]
- **Attack surface:** [New endpoints, inputs, or trust boundaries]
