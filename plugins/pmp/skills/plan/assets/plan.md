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
**Spec Corpus:** [N files | "Inline spec" | "GitHub Issues"]
**Master Plan:** [No | `path/to/master-plan.md`]

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

## Feature Dependencies

<!-- Execution parallelism matrix. Controller uses this to dispatch parallel feature orchestrators. -->

| Feature | Depends On | Can Parallel With |
|---------|-----------|-------------------|
| Feature 1: [Name] | None | Feature 2, Feature 3 |
| Feature 2: [Name] | None | Feature 1, Feature 3 |
| Feature 3: [Name] | Feature 1 | Feature 2 |

---

## Spec Traceability

<!-- Every spec section maps to a feature + AC. Uncovered sections are flagged. -->

| Spec File | Section | Feature | AC | Status |
|-----------|---------|---------|----|----|
| [file.md](file.md) | #section | Feature 1 | AC-1.1 | Covered |
| [file2.md](file2.md) | #section | — | — | Not covered — intentional (deferred to v3) |

---

## Phases

<!-- Include this section for plans with 5+ features. Group features into phases
     based on dependency ordering. Features within a phase have no cross-dependencies.
     Execute loop uses phase boundaries as batch boundaries when present. -->

### Phase 1: [Name]
**Features:** Feature 1, Feature 2, Feature 3
**Entry:** Branch created, E2E infrastructure ready
**Exit:** All Phase 1 E2E tests pass, CI green

### Phase 2: [Name]
**Features:** Feature 4
**Depends on:** Phase 1
**Entry:** Phase 1 exit criteria met
**Exit:** All Phase 2 E2E tests pass, CI green

---

<!-- Repeat Feature sections using assets/feature.md structure -->
<!-- After issues are created, feature headings include the issue reference -->

## Feature 1: [Name] · #<number>

...

---

## Performance Test Plan

<!-- Include when spec defines performance targets. Omit if N/A. -->

**Targets:** [From spec — P95 latency, throughput, concurrency]
**Workload:** [Request volume, payload size, concurrency level]
**Duration:** [Warmup + test duration]
**Validation:** [P95 < Xms, memory < Y MB, CPU < Z%]

## Resilience Test Plan

<!-- Include when spec defines failure handling. Omit if N/A. -->

**Scenarios:**
- [Dependency X outage → expected system behavior]
- [Config reload under traffic → expected behavior]
**Validation:** [What to assert per scenario]

## Fuzz Test Plan

<!-- Include when feature includes parsers/decoders. Omit if N/A. -->

**Targets:** [Which parsers/inputs to fuzz]
**Corpus:** [Seed inputs]
**Duration:** [Minimum fuzz duration]
**Validation:** [No panics, no memory corruption, graceful error handling]

---

## Security Considerations

- **Input validation:** [What inputs need validation, how]
- **Auth boundaries:** [What auth checks are needed]
- **Secret handling:** [How secrets are protected]
- **Injection risks:** [SQL, command injection vectors]
- **Attack surface:** [New endpoints, inputs, or trust boundaries]

---

## What's Next

> **Next in the PMP lifecycle:** Get the plan reviewed before execution.
>
> | Action | Command | When to use |
> |--------|---------|-------------|
> | Review this plan | `pmp:plan-review` | Validate completeness, security, and feasibility |
> | Decompose into phases | `pmp:decompose` | Break a large plan (5+ features) into ordered phases |
